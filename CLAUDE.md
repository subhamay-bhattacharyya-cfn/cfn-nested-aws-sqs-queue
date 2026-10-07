# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a **CloudFormation template repository** that provides reusable nested stack templates for deploying SQS queues with flexible configuration for Standard and FIFO queue types. Templates follow the nested stack pattern and are designed to be referenced by parent CloudFormation stacks.

**Key characteristics:**

- Nested CloudFormation templates (referenced via `TemplateURL`)
- Parameterized queue naming with environment and region
- Support for both Standard and FIFO queue types
- Configurable message retention and visibility timeout
- Cross-account access policies for multi-account scenarios
- Automated semantic versioning and releases
- AWS OIDC authentication for CI/CD deployments

## Project Structure

```text
cloudformation/
└── template.yaml                  # Nested template: SQS queue creation (Standard/FIFO)

parameters/
├── dev.json                       # Parameters for development environment
├── staging.json                   # Parameters for staging environment
└── prod.json                      # Parameters for production environment

.github/workflows/
├── ci.yaml                        # Validates, deploys, and cleans up templates
├── release.yaml                   # Semantic release on push to main
└── create-branch.yaml             # Auto-create feature branches from issues

scripts/plugins/
├── release.config.js              # Semantic-release configuration
└── (other release plugins)        # Custom commit analysis, notes generation

.claude/
├── settings.json                  # Claude Code workspace settings
└── settings.local.json            # Local overrides

.devcontainer/
└── devcontainer.json              # Dev container setup (Node.js 20)

package.json                       # Dependencies: semantic-release, commitizen
README.md                          # Template documentation and usage examples
CLAUDE.md                          # This file: Claude Code guidance
```

## Development Commands

### Install dependencies

```bash
npm ci
```

### Trigger semantic release (usually automatic on main)

```bash
npm run release
```

### Commit with conventional commit format

```bash
npx cz commit
```

Select `feat`, `fix`, or `chore` type. Only `feat` and `fix` trigger releases.

## Key Architecture Concepts

### Nested Stack Pattern

This repo provides **nested stack templates** — templates that are referenced from parent/root CloudFormation stacks via `TemplateURL`. The templates are self-contained and export outputs for cross-stack references.

- **Parent stack** calls: `AWS::CloudFormation::Stack` with `TemplateURL` pointing to S3
- **Nested templates** output values via `Outputs` section with `Export`
- Parent retrieves outputs via `!GetAtt NestedStack.Outputs.OutputKey`

### Queue Naming Convention

Queue names follow a deterministic pattern driven by parameters:

**Standard Queue:**
```bash
{ProjectName}-{QueueBaseName}-{Environment}-{Region}[-{CiSuffix}]
```

**FIFO Queue:**
```bash
{ProjectName}-{QueueBaseName}-{Environment}-{Region}[-{CiSuffix}].fifo
```

Examples:
- `myapp-task-queue-devl-us-east-1` (Standard)
- `myapp-order-queue-prod-us-east-1.fifo` (FIFO)
- `myapp-test-queue-test-us-east-1-ci123` (Standard with CI suffix)

This ensures:

- Uniqueness across environments and regions
- Environment isolation
- Clear distinction between Standard and FIFO queues
- Consistent naming for infrastructure automation

### Parameter-Driven Configuration

The template accepts parameters to support:

- **Queue Type Selection**: Standard or FIFO queues with automatic naming
- **Message Configuration**: Visibility timeout, retention period, and long polling
- **Cross-Account Access**: Optional resource policy for multi-account scenarios
- **CI/CD Integration**: Optional suffix for ephemeral test deployments

## Key Files to Understand

### `cloudformation/template.yaml`

**Purpose:** Creates an SQS queue with flexible configuration for Standard or FIFO queue types

**Key inputs:**

- `ProjectName` (required): Project prefix (lowercase, alphanumeric, hyphens only)
- `QueueBaseName` (default: `sqs-queue`): Base name component
- `Environment` (default: `devl`): Environment label (devl, stag, prod)
- `QueueType` (default: `Standard`): Queue type (Standard or FIFO)
- `CiSuffix` (optional): Suffix for CI/CD unique deployments
- `CrossAccountArns` (optional): Comma-delimited list of cross-account principal ARNs

**Key outputs:**

- `QueueName`: Queue name (exported for parent stack)
- `QueueUrl`: Queue URL (exported for parent stack)
- `QueueArn`: Queue ARN (exported for parent stack)
- `QueueType`: Queue type indicator

**Features:**

- Support for both Standard and FIFO queue types
- Automatic `.fifo` suffix for FIFO queues
- Configurable message retention (60s to 14 days, default: 4 days)
- Configurable visibility timeout (0-43200s, default: 30s)
- Optional long polling support (0-20s wait time)
- Content-based deduplication for FIFO queues
- Same-account and cross-account IAM policies
- Conditional naming: different queue name with/without CI suffix

### `.github/workflows/ci.yaml`

**Triggered on:**

- Manual workflow_dispatch (anytime)
- Pull requests (any branch)
- Pushes to `feature/**` and `bug/**` branches

**Path filter:** Only runs if changes to `cloudformation/`, `parameters/`, or `.github/workflows/ci.yaml`

**Phases:**

1. **Validation:** `aws cloudformation validate-template` on the queue template
2. **Deployment:** Creates CloudFormation stack in CI environment with test parameters
3. **Cleanup:** Destroys the test stack for ephemeral testing

**Environment setup:**

- Reads config from GitHub environment variables: `AWS_REGION`, `AWS_ACCOUNT_ID`, `OIDC_ROLE_NAME`, `CFN_TEMPLATES_S3_BUCKET`
- Uses AWS OIDC for keyless authentication via `aws-actions/configure-aws-credentials`
- Requires GitHub environment `ci` with OIDC trust configured

### `.github/workflows/release.yaml`

**Triggered:** On push to main

**Process:**

1. Analyze commits (conventional format: `feat:`, `fix:`, `BREAKING CHANGE:`)
2. Generate release notes
3. Update CHANGELOG.md
4. Create GitHub release and tag
5. Commit version bump

**Release rules:**

- `feat:` → MINOR bump (0.1.0 → 0.2.0)
- `fix:` → PATCH bump (0.1.0 → 0.1.1)
- `BREAKING CHANGE:` → MAJOR bump (0.1.0 → 1.0.0)
- Other commits → no release

## Testing & Validation

**Manual template validation:**

```bash
aws cloudformation validate-template --template-body file://cloudformation/template.yaml
```

**Manual stack deployment:**

```bash
# Deploy queue to dev environment
aws cloudformation deploy \
  --template-file cloudformation/template.yaml \
  --stack-name my-sqs-queue-dev \
  --parameter-overrides file://parameters/dev.json \
  --region us-east-1

# Wait for stack creation
aws cloudformation wait stack-create-complete --stack-name my-sqs-queue-dev --region us-east-1

# Get queue details
aws cloudformation describe-stacks \
  --stack-name my-sqs-queue-dev \
  --query 'Stacks[0].Outputs' \
  --output table
```

The CI workflow (ci.yaml) runs this full cycle automatically on PR, then cleans up.

## AWS Credentials & Environment Variables

**GitHub environment variables required in `ci` environment:**

- `AWS_REGION`: CloudFormation deployment region
- `AWS_ACCOUNT_ID`: AWS account to deploy into
- `OIDC_ROLE_NAME`: IAM role name for OIDC trust (uses `arn:aws:iam::{ACCOUNT_ID}:role/{ROLE_NAME}`)
- `CFN_TEMPLATES_S3_BUCKET`: S3 bucket where templates are stored

**OIDC setup:** The CI workflow uses AWS OIDC for keyless auth. The GitHub OIDC provider must trust the specified role.

## Conventional Commits & Release Flow

This repo enforces conventional commits to drive semantic versioning:

```bash
npx cz commit
```

Commit types:

- `feat: add support for X` → triggers MINOR release
- `fix: correct behavior of Y` → triggers PATCH release
- `chore: update deps` → no release
- `docs: clarify README` → no release

Only commits to `main` trigger releases. Feature branches use this format but releases happen on merge to main.

## When Modifying Templates

1. **Edit the template YAML** in `cloudformation/`
2. **Update parameter files** in `parameters/` if new parameters added
3. **Test locally** with `aws cloudformation validate-template --template-body file://cloudformation/template.yaml`
4. **Verify queue naming** matches the expected pattern: `{ProjectName}-{QueueBaseName}-{Environment}-{Region}[-{CiSuffix}][.fifo]`
5. **Create a PR** with conventional commit message (e.g., `feat: add cross-account access support`)
6. **CI validates and deploys** to dev environment automatically
7. **Merge to main** → release workflow creates version tag and GitHub release

## Dev Container

Pre-configured with:

- Node.js 20
- GitHub Copilot extension

Use via VS Code: `code --remote-container-url <repo-url>`

## Current Branch & Development Workflow

- **Main branch** (`main`) is the release branch — auto-triggered for semantic versioning on merge
- **Feature branches** derive from main and merge back via PR
- **Branch naming convention:** `{type}/CFN-{issue-number}-{slug}`
  - Examples: `feature/CFN-0001-implement-sqs-queue`, `bug/CFN-0002-fix-naming`
- **CI runs automatically** on PR and feature/* branches to validate templates
- **Commit format:** Use `npx cz commit` to ensure conventional commit format for semantic releases

