---
type: personal-project
status: in-progress
stack: [Terraform, AWS KMS, mise, Docker, PostgreSQL]
code_path: "~/fun-projects/prism"
---

# Prism

Infrastructure as code project with Terraform and encrypted secrets management.

## Purpose
Manage cloud infrastructure declaratively with secrets encrypted via AWS KMS — likely used for hosting personal/freelance projects or homelab-adjacent infra.

## Stack
- **IaC**: Terraform
- **Secrets**: AWS KMS encryption
- **Tool manager**: mise (manages Terraform and other tool versions)
- **Database**: PostgreSQL (local dev via Docker)

## Setup
```bash
# Install mise first
curl https://mise.run | sh
echo 'eval "$(~/.local/bin/mise activate zsh)"' >> ~/.zshrc

# Install all tools
mise install

# Local DB
cp .env.example .env
# Edit .env → set POSTGRES_PASSWORD
docker compose up
```

## Secrets Workflow
```bash
# Encrypt a secret for dev
echo "value" > secret.txt
mise run encrypt dev secret.txt
mv secret.txt infrastructure/terraform/stacks/secrets/_encrypted_files/dev/
```

## Structure
```
prism/
├── infrastructure/
│   └── terraform/
│       └── stacks/
│           └── secrets/
│               └── _encrypted_files/
│                   └── dev/
├── mise.toml
└── .env.example
```
