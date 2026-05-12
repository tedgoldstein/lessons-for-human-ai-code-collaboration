---
name: local-aws-environment-using-localstack
description: When developing services that talk to AWS (S3, SQS, DynamoDB, etc.), run those AWS services LOCALLY via LocalStack rather than (a) letting the app fail when AWS calls happen, (b) faking the data, or (c) sharing a dev AWS account. Recipe is a small docker-compose service, an init script that creates the resources at startup, and a client wrapper that targets the local endpoint or the real AWS based on config.
version: 0.1.0
---

# Local AWS Environment Using LocalStack

Services that depend on AWS have a familiar dev-loop problem: how do
you run them on your laptop? Four options, ranked by leverage:

1. **Let the app fail when AWS is called.** The dev loop is broken
   for every code path that touches AWS. Bad.
2. **Fake the data / stub the client.** Works in unit tests; falls
   over when behavior actually matters (queue semantics, S3 ACL
   quirks, request signing). Mocks lie eventually.
3. **Share a dev AWS account.** Real semantics; collisions between
   teammates; AWS bill; data hygiene; credentials sprawl.
4. **Run AWS locally via LocalStack.** A docker container that
   emulates S3 / SQS / DynamoDB / Lambda / etc. on `localhost`.
   Real client SDKs work against it. No shared state, no bill.

This skill is the **recipe for option 4**.

## When to invoke

- A service depends on S3 / SQS / Lambda / DynamoDB / Kinesis /
  Secrets Manager / IAM / etc.
- A new contributor's onboarding doc says "ask someone for AWS
  credentials" or "wait, this needs the staging account."
- Integration tests are skipped on laptops because they need real
  AWS.
- You're considering writing a mock for an AWS service in your
  test suite — pause and consider LocalStack instead.
- A long-running queue or stream is needed in dev and you don't
  want to point at production resources.

## What LocalStack is — and isn't

**Is:** a single Docker container that listens on a single port
(default `4566`) and emulates AWS service APIs. Most services
ship in the free/community image; some are paid Pro features.
The official AWS SDK clients work against it unchanged — just
point at `http://localhost:4566` and use any credentials.

**Isn't:** byte-identical to AWS. Behaviors near the edges
(IAM authorization, eventual consistency, throttling, certain
Lambda runtimes) can drift. Use LocalStack for the dev loop +
integration tests; rely on a real (sandbox) AWS account before
prod for the parts where parity matters.

## The recipe

### 1. `docker-compose.yml` service

A small service block in the project's `docker-compose.yml`:

```yaml
services:
  localstack:
    image: localstack/localstack:latest
    ports:
      - "4566:4566"
    environment:
      SERVICES: s3,sqs                  # opt in to only what you need
      DEFAULT_REGION: eu-west-2
      HOSTNAME_EXTERNAL: localstack     # so SQS URLs are reachable from app containers
      AWS_ACCESS_KEY_ID: test           # dummy — LocalStack doesn't check
      AWS_SECRET_ACCESS_KEY: test
    volumes:
      - ./localstack/init:/etc/localstack/init/ready.d
```

Key flags:

- **`SERVICES`** — start only what the project uses. Smaller
  image footprint, faster boot.
- **`HOSTNAME_EXTERNAL`** — LocalStack hands out URLs (e.g. SQS
  queue URLs) containing this hostname. If the app runs in
  another container, set this to LocalStack's docker-network
  name (typically `localstack`) so the URLs are reachable.
- **`init/ready.d` volume** — scripts in this directory run
  once LocalStack is ready. This is where you create buckets,
  queues, tables.

### 2. Init script — create the resources

A shell script (e.g. `localstack/init/create_resources.sh`) run
on LocalStack startup, using the `aws` CLI with the LocalStack
endpoint:

```bash
#!/usr/bin/env bash
set -euo pipefail

awslocal() {
  aws --endpoint-url=http://localhost:4566 "$@"
}

# Buckets
awslocal s3 mb s3://my-app-uploads
awslocal s3 mb s3://my-app-thumbnails

# Queues
awslocal sqs create-queue --queue-name jobs
awslocal sqs create-queue --queue-name dead-letters
```

(`awslocal` is also available as a packaged wrapper; the inline
function shown is equivalent.)

The script lives in the repo so the resources are deterministic
— `docker-compose up` rebuilds them.

### 3. Two config profiles

The app reads its AWS endpoint + credentials from config. Two
profiles, structurally identical:

**Local (`conf/application.local.conf`)**
```hocon
aws {
  endpoint = "http://localstack:4566"   # or localhost from outside docker
  region   = "eu-west-2"
  uploads-bucket = "my-app-uploads"
  jobs-queue     = "jobs"
}
```

**Production (no endpoint — SDK targets real AWS)**
```hocon
aws {
  # endpoint omitted — SDK uses real AWS endpoints
  region   = ${?AWS_REGION}
  uploads-bucket = ${?UPLOADS_BUCKET}    # from Terraform output
  jobs-queue     = ${?JOBS_QUEUE}
}
```

The bucket / queue names differ; everything else is the same
shape. The presence-or-absence of `endpoint` is the switch.

### 4. Client construction — one if-statement

The client builder reads the config and branches on
endpoint-set / endpoint-absent. (Pattern shown in Scala; same
shape works in any AWS SDK language.)

```scala
def buildSqsClient(cfg: Config): AmazonSQS = {
  val region = cfg.getString("aws.region")
  val endpointOpt = if (cfg.hasPath("aws.endpoint")) Some(cfg.getString("aws.endpoint")) else None

  val builder = AmazonSQSClientBuilder.standard()
  endpointOpt match {
    case Some(endpoint) =>
      // LocalStack — dummy creds, explicit endpoint
      builder
        .withCredentials(new AWSStaticCredentialsProvider(new BasicAWSCredentials("test", "test")))
        .withEndpointConfiguration(new EndpointConfiguration(endpoint, region))
    case None =>
      // Real AWS — let the chain find the instance role / env / shared file
      builder
        .withRegion(region)
        .withCredentials(DefaultAWSCredentialsProviderChain.getInstance())
  }
  builder.build()
}
```

Three things to notice:

- One `if` does all the switching. No `if (env == "dev")`
  scattered across the codebase.
- The *real-AWS* path uses
  `DefaultAWSCredentialsProviderChain` so it pulls from
  whatever the production environment provides (IAM instance
  role, OIDC, shared file). Production code never sees a
  hard-coded credential.
- The *local* path uses literal `"test"` credentials because
  LocalStack ignores them.

## Why this beats the alternatives

| | Real-AWS calls fail | Faked data | Shared dev AWS | LocalStack |
|---|---|---|---|---|
| Onboarding | "ask for creds" | works | "ask for creds" | `docker-compose up` |
| Realism | low | low | high | medium-high |
| Cost | none | none | AWS bill | none |
| Isolation | n/a | yes | NO (collisions) | yes |
| Tests can run on CI | no | yes | with creds | yes (just spin up LocalStack) |
| Production code path | different (mocks) | different | same | same |

The last row is the big one. The production code is the same code
that runs locally; only the endpoint differs. That's the property
mocks can never give you.

## Common failure modes

- **Forgetting `HOSTNAME_EXTERNAL`.** SQS queue URLs come back
  containing `localhost`, which the app container can't reach.
  Set `HOSTNAME_EXTERNAL` to the docker-network name of the
  LocalStack service.
- **Drifting init script.** A new bucket is added in prod's
  Terraform but the init script isn't updated — local dev
  silently misses it. Treat the init script as a first-class
  source-of-truth, reviewed alongside the Terraform.
- **Assuming feature parity.** Some AWS services / APIs only
  exist in LocalStack Pro; some have subtle behavior gaps.
  Smoke-test against a real sandbox account before relying on
  edge-case behavior.
- **Leaving real credentials in dev.** With LocalStack you no
  longer *need* real AWS creds locally. Remove them from
  `.aws/credentials` or scope them out — fewer credentials in
  developer machines is a security win.
- **One container per dev.** Most teams run LocalStack as
  part of the project's `docker-compose.yml` so it's
  per-checkout, not per-machine. Per-machine instances drift
  in resource shape between team members.

## When the principle DOES NOT apply

- **Services LocalStack doesn't cover.** Some AWS services
  (Bedrock, real Lambda runtimes for some languages, certain
  newer offerings) aren't emulated faithfully or at all. For
  those, a sandbox AWS account is the right answer.
- **Behavior parity matters and the bug surface is at the AWS
  edges.** When testing IAM policy correctness, eventual
  consistency edge cases, throttling response, or signed-URL
  semantics, LocalStack is helpful but not authoritative.
  Final verification belongs on real AWS.
- **The app doesn't actually depend on AWS-specific
  semantics.** If the only AWS thing is S3-as-blob-store, a
  generic local blob store (MinIO) may be a closer fit.
  Pick the smallest substrate that actually mirrors prod.

## Tagline

> Don't fake AWS. Run it.

The endpoint is the only thing that changes between dev and
prod. The code is the same code.

## Sources

Recipe and motivation from Oksana Horlock's "Setting up local
AWS environment using Localstack" (67 Bricks engineering blog,
January 2023; blog.67bricks.com). This SKILL.md is a
restatement in our own voice; the docker-compose + init script
+ client-init pattern is from the original.
