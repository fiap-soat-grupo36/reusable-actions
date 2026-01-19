# 🚀 Terraform Deployment

**Arquivo:** `_reusable-terraform.yml`

## 📋 Características

- ✅ Variáveis dinâmicas via JSON
- ✅ Secrets customizáveis por projeto
- ✅ Multi-workspace support
- ✅ Auto-apply opcional
- ✅ Plan output em PRs
- ✅ Suporte a tfvars files

## 📝 Inputs

| Input | Obrigatório | Default | Descrição |
|-------|-------------|---------|-----------|
| `workspace` | ✅ | - | Nome do workspace Terraform |
| `environment` | ✅ | - | Ambiente (dev/staging/prod) |
| `working_directory` | ❌ | `./infra/` | Diretório do código Terraform |
| `aws_region` | ❌ | `us-east-1` | Região AWS |
| `tfvars_file` | ❌ | `''` | Arquivo .tfvars (opcional) |
| `auto_apply` | ❌ | `false` | Auto-aplicar mudanças |
| `terraform_vars` | ❌ | `{}` | JSON com variáveis TF |
| `terraform_secrets` | ❌ | `{}` | JSON mapeando secrets → TF_VAR |

## 🔐 Secrets

| Secret | Obrigatório | Descrição |
|--------|-------------|-----------|
| `AWS_ACCESS_KEY_ID` | ✅ | Access Key AWS |
| `AWS_SECRET_ACCESS_KEY` | ✅ | Secret Key AWS |
| Outros | ❌ | Definidos via `terraform_secrets` |

---

## 📚 Exemplos

### 1. Serviço Java com Docker

```yaml
name: Deploy Java Service

on:
  push:
    branches: [main]

jobs:
  deploy:
    uses: your-org/reusable-actions/.github/workflows/_reusable-terraform.yml@v1
    with:
      workspace: production
      environment: prod
      working_directory: ./infra
      auto_apply: true
      terraform_vars: |
        {
          "image_tag": "${{ github.sha }}",
          "service_name": "my-java-app",
          "instance_type": "t3.medium",
          "heap_size": "2048m",
          "replicas": 3
        }
      terraform_secrets: |
        {
          "datadog_api_key": "DATADOG_API_KEY",
          "datadog_app_key": "DATADOG_APP_KEY",
          "db_password": "RDS_PASSWORD"
        }
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      DATADOG_API_KEY: ${{ secrets.DATADOG_API_KEY }}
      DATADOG_APP_KEY: ${{ secrets.DATADOG_APP_KEY }}
      RDS_PASSWORD: ${{ secrets.RDS_PASSWORD }}
```

### 2. Lambda Python

```yaml
name: Deploy Lambda

on:
  push:
    branches: [main]

jobs:
  deploy:
    uses: your-org/reusable-actions/.github/workflows/_reusable-terraform.yml@v1
    with:
      workspace: staging
      environment: dev
      working_directory: ./terraform
      auto_apply: true
      terraform_vars: |
        {
          "lambda_s3_bucket": "my-lambdas",
          "lambda_s3_key": "processor-${{ github.sha }}.zip",
          "runtime": "python3.11",
          "memory_size": 512,
          "timeout": 300
        }
      terraform_secrets: |
        {
          "datadog_api_key": "DD_API_KEY"
        }
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      DD_API_KEY: ${{ secrets.DD_API_KEY }}
```

### 3. Infraestrutura Pura (sem secrets adicionais)

```yaml
name: Deploy Infrastructure

on:
  pull_request:
    branches: [main]

jobs:
  plan:
    uses: your-org/reusable-actions/.github/workflows/_reusable-terraform.yml@v1
    with:
      workspace: infra-base
      environment: prod
      working_directory: ./infrastructure
      auto_apply: false  # Apenas plan no PR
      terraform_vars: |
        {
          "vpc_cidr": "10.0.0.0/16",
          "enable_nat_gateway": true,
          "az_count": 3,
          "environment": "production"
        }
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

### 4. Com arquivo tfvars

```yaml
jobs:
  deploy:
    uses: your-org/reusable-actions/.github/workflows/_reusable-terraform.yml@v1
    with:
      workspace: production
      environment: prod
      tfvars_file: 'environments/prod.tfvars'
      terraform_vars: |
        {
          "image_tag": "${{ github.sha }}"
        }
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

### 5. Multi-ambiente com Matrix

```yaml
name: Multi-Environment Deploy

on:
  workflow_dispatch:
    inputs:
      environment:
        type: choice
        options: [dev, staging, prod]

jobs:
  deploy:
    strategy:
      matrix:
        include:
          - environment: dev
            workspace: dev
            auto_apply: true
            instance_type: t3.micro
          - environment: staging
            workspace: staging
            auto_apply: true
            instance_type: t3.small
          - environment: prod
            workspace: production
            auto_apply: false
            instance_type: t3.medium
    
    uses: your-org/reusable-actions/.github/workflows/_reusable-terraform.yml@v1
    with:
      workspace: ${{ matrix.workspace }}
      environment: ${{ matrix.environment }}
      auto_apply: ${{ matrix.auto_apply }}
      terraform_vars: |
        {
          "environment": "${{ matrix.environment }}",
          "instance_type": "${{ matrix.instance_type }}"
        }
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

---

## 🎯 Casos de Uso

### Variáveis Simples
```yaml
terraform_vars: '{"image_tag": "v1.2.3"}'
```

### Sem Secrets Adicionais
```yaml
# Não passe terraform_secrets se não precisa
terraform_vars: '{"instance_type": "t3.micro"}'
```

### Secrets Dinâmicos
O `terraform_secrets` mapeia o **nome da secret do GitHub** para o **nome da variável TF**:

```yaml
terraform_secrets: |
  {
    "datadog_api_key": "DATADOG_API_KEY",
    "db_password": "DB_PASSWORD"
  }
```

Isso cria:
- `TF_VAR_datadog_api_key` ← valor de `${{ secrets.DATADOG_API_KEY }}`
- `TF_VAR_db_password` ← valor de `${{ secrets.DB_PASSWORD }}`

---

## ⚙️ Comportamento

### Em Pull Requests
- Executa `terraform plan`
- Comenta o PR com resultado
- **Nunca** aplica mudanças

### Em Push (com auto_apply=true)
- Executa `terraform plan`
- Executa `terraform apply -auto-approve`

### Em Push (com auto_apply=false)
- Apenas `terraform plan`
- Requer aprovação manual separada

---

## 🔍 Troubleshooting

### Variáveis não reconhecidas
```yaml
# ❌ JSON inválido
terraform_vars: "{key: value}"

# ✅ JSON válido
terraform_vars: '{"key": "value"}'
```

### Secret não existe
```yaml
# Se mapear secret inexistente, variável ficará vazia
# Sempre valide que secrets existem no repositório

terraform_secrets: '{"api_key": "MISSING_SECRET"}'  # ⚠️
```

### Workspace não existe
O workflow cria automaticamente se não existir.

---

## 📖 Ver Também

- [Pipelines Completos](./PIPELINES.md)
- [Docker Build](./DOCKER.md) - Para build de imagens antes do deploy
- [Upload Package](./DEPLOYMENT.md) - Para Lambdas
- [Troubleshooting](./TROUBLESHOOTING.md)
