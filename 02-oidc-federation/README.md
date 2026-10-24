# GitHub Actions to AWS OIDC Federation

## Overview

This lab demonstrates how to securely authenticate GitHub Actions workflows to AWS using OpenID Connect (OIDC) federation, without storing long-lived AWS access keys in GitHub Secrets.

GitHub Actions obtains an OIDC token, AWS validates the token against an IAM trust policy, and AWS Security Token Service (STS) issues temporary credentials for an IAM role.

## Architecture

```text
Developer pushes code to GitHub
             |
             v
      GitHub Actions
             |
             v
 Requests GitHub OIDC token
             |
             v
 AWS IAM OIDC Identity Provider
             |
             v
 IAM Trust Policy Validation
             |
             v
 GitHubActionsOIDCRole
             |
             v
 AWS STS Temporary Credentials
             |
             v
 Verify AWS Identity
             |
             v
 List AWS Regions
```

## Objectives

- Configure GitHub as an AWS OIDC identity provider.
- Create an IAM role trusted by a specific GitHub repository and branch.
- Configure least-privilege permissions for the role.
- Authenticate GitHub Actions to AWS without long-lived access keys.
- Verify the assumed role and test its AWS permissions.
- Demonstrate secure identity federation in a CI/CD pipeline.

## Environment

- GitHub repository: `paycenonoli/aws-kubernetes-identity-lab`
- Development environment: WSL Ubuntu
- CI/CD platform: GitHub Actions
- Cloud provider: Amazon Web Services
- Identity federation: OpenID Connect (OIDC)
- Temporary credentials: AWS Security Token Service (STS)
- AWS region used by the workflow: `us-east-1`

## Implementation

### 1. Create the AWS OIDC identity provider

Registered GitHub's OIDC issuer in AWS IAM:

```text
https://token.actions.githubusercontent.com
```

Configured the expected audience:

```text
sts.amazonaws.com
```

Provider ARN:

```text
arn:aws:iam::240250535062:oidc-provider/token.actions.githubusercontent.com
```

### 2. Create the IAM role

Created the role:

```text
GitHubActionsOIDCRole
```

Role ARN:

```text
arn:aws:iam::240250535062:role/GitHubActionsOIDCRole
```

### 3. Configure the trust policy

Created `trust-policy.json` to allow web identity federation through GitHub's OIDC provider.

The policy restricts role assumption by checking:

- The OIDC audience is `sts.amazonaws.com`.
- The token subject identifies the intended repository and `main` branch.
- The subject uses GitHub's immutable owner and repository IDs.

This limits trust to the intended repository context instead of allowing arbitrary GitHub repositories to assume the role.

### 4. Configure least-privilege permissions

Created `permissions-policy.json` granting the following action:

```text
ec2:DescribeRegions
```

Attached the policy to `GitHubActionsOIDCRole` using the inline policy name:

```text
GitHubActionsReadRegions
```

This permission allows the workflow to list AWS regions without granting permission to create or delete infrastructure.

### 5. Configure the GitHub Actions workflow

Created:

```text
.github/workflows/aws-oidc.yml
```

The workflow:

1. Runs when code is pushed to `main` or is manually triggered.
2. Requests OIDC token access through the `id-token: write` permission.
3. Uses the AWS credentials action to assume `GitHubActionsOIDCRole`.
4. Verifies the AWS caller identity.
5. Lists AWS regions using the role's limited permissions.

No long-lived AWS access keys are configured in GitHub Secrets for this workflow.

## Validation and Results

The workflow completed successfully.

| Validation | Result |
|---|---|
| GitHub OIDC provider configured | Passed |
| IAM role trust policy configured | Passed |
| OIDC authentication from GitHub Actions | Passed |
| Temporary AWS credentials obtained | Passed |
| AWS caller identity verified | Passed |
| AWS Regions listed | Passed |

The `aws sts get-caller-identity` command returned an assumed-role ARN similar to:

```text
arn:aws:sts::240250535062:assumed-role/GitHubActionsOIDCRole/GitHubActions
```

This confirms that the workflow operated under the IAM role rather than the administrator IAM user.

## Security Considerations

- Avoid storing long-lived AWS access keys in GitHub repository secrets.
- Restrict OIDC trust policies to the required repositories, branches, or environments.
- Grant only the AWS permissions needed by the workflow.
- Do not commit credentials, tokens, or other secrets to Git.
- Pin third-party GitHub Actions to reviewed full commit SHAs in production pipelines.
- Review IAM policies periodically and remove unused resources.
- Sanitize screenshots and logs before publishing them.

## Repository Files

```text
02-oidc-federation/
├── README.md
├── trust-policy.json
└── permissions-policy.json

.github/
└── workflows/
    └── aws-oidc.yml
```

## Key Takeaways

This lab demonstrates how OIDC federation enables secure, keyless authentication between GitHub Actions and AWS.

Instead of relying on a permanent IAM user's access keys, the workflow presents a GitHub-issued identity token. AWS validates that identity against the configured trust policy, and STS provides temporary credentials for the permitted role.

This approach reduces long-lived credential exposure and supports secure CI/CD automation using identity federation and least privilege.

## Disclaimer

This is a hands-on learning project intended to demonstrate AWS IAM, OIDC federation, STS, and GitHub Actions security concepts. Production implementations should follow organizational security standards and use reviewed, immutable action references.
