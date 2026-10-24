### Lesson 1 — IAM Roles and AWS STS

Demonstrates role assumption, trust policies, permission policies,
temporary credentials, authorization testing, and cleanup.

[Open Lesson 1](01-iam-roles-sts/README.md)

### Lesson 2 — OIDC Federation for CI/CD

Will demonstrate how a CI/CD workload can obtain temporary AWS
credentials through OIDC federation without storing long-lived AWS
access keys in the pipeline.

### Lesson 3 — EKS Authentication

Will explore kubeconfig, EKS access, and Kubernetes RBAC.

### Lesson 4 — Workload Identity

Will compare IAM Roles for Service Accounts (IRSA) and EKS Pod Identity.

### Lesson 5 — Enterprise Identity

Will explore identity federation, SSO, least privilege, and auditing.

## Security Practices

- Never commit AWS access keys, secret keys, session tokens, or passwords.
- Use least-privilege IAM permissions.
- Restrict OIDC trust to the intended repository and workflow context.
- Verify the active AWS identity before making changes.
- Remove temporary lab resources after testing.
- Review and sanitize logs and screenshots before publishing.

## Status

Lesson 1 hands-on testing and AWS cleanup have been completed.
Documentation is being added to the repository.

## Disclaimer

This is a learning project. Review all policies and commands before
using them in a production environment.

