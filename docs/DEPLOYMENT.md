# 📦 Upload Lambda Package

**Arquivo:** `_reusable-upload-package.yml`

## 📋 Características

- ✅ Build de pacote Lambda
- ✅ Upload para S3 (opcional)
- ✅ Validação de artifact
- ✅ AWS credentials seguras
- ✅ Outputs para integração
- ✅ Python version configurável

## 📝 Inputs

| Input | Obrigatório | Default | Descrição |
|-------|-------------|---------|-----------|
| `project_root` | ❌ | `./` | Raiz do projeto |
| `package_script` | ❌ | `./scripts/package_lambda.sh` | Script de build |
| `artifact_path` | ❌ | `app/lambda.zip` | Path do zip gerado |
| `artifact_name` | ❌ | `lambda-package` | Nome do artifact |
| `s3_bucket` | ❌ | `''` | Bucket S3 (vazio = sem upload) |
| `s3_key` | ❌ | `''` | Key S3 (vazio = nome do arquivo) |
| `aws_region` | ❌ | `us-east-1` | Região AWS |
| `python_version` | ❌ | `3.12` | Versão Python |
| `validate_artifact` | ❌ | `true` | Validar que zip existe |

## 🔐 Secrets

| Secret | Obrigatório | Quando |
|--------|-------------|--------|
| `AWS_ACCESS_KEY_ID` | ❌ | Se `s3_bucket` fornecido |
| `AWS_SECRET_ACCESS_KEY` | ❌ | Se `s3_bucket` fornecido |

## 📤 Outputs

| Output | Descrição |
|--------|-----------|
| `s3_bucket` | Bucket usado no upload |
| `s3_key` | Key do arquivo no S3 |

---

## 📚 Exemplos

### 1. Upload para S3

```yaml
name: Build and Upload Lambda

on:
  push:
    branches: [main]

jobs:
  upload:
    uses: your-org/reusable-actions/.github/workflows/_reusable-upload-package.yml@v1
    with:
      project_root: './lambda'
      package_script: './scripts/package.sh'
      artifact_path: 'dist/function.zip'
      artifact_name: 'lambda-function'
      s3_bucket: 'my-lambda-artifacts'
      s3_key: 'functions/processor-${{ github.sha }}.zip'
      python_version: '3.11'
      validate_artifact: true
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

### 2. Apenas Artifact (sem S3)

```yaml
jobs:
  package:
    uses: your-org/reusable-actions/.github/workflows/_reusable-upload-package.yml@v1
    with:
      project_root: './app'
      package_script: './build.sh'
      artifact_path: 'build/app.zip'
      artifact_name: 'application-package'
      validate_artifact: true
      # Sem s3_bucket = não faz upload S3
```

### 3. Usando Outputs para Deploy

```yaml
jobs:
  upload:
    uses: your-org/reusable-actions/.github/workflows/_reusable-upload-package.yml@v1
    with:
      s3_bucket: 'my-lambdas'
      s3_key: 'app-${{ github.sha }}.zip'
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

  deploy:
    needs: upload
    runs-on: ubuntu-latest
    steps:
      - name: Deploy Lambda
        run: |
          echo "Deploying from s3://${{ needs.upload.outputs.s3_bucket }}/${{ needs.upload.outputs.s3_key }}"
          aws lambda update-function-code \
            --function-name my-function \
            --s3-bucket ${{ needs.upload.outputs.s3_bucket }} \
            --s3-key ${{ needs.upload.outputs.s3_key }}
```

### 4. Com Terraform

```yaml
jobs:
  package:
    uses: your-org/reusable-actions/.github/workflows/_reusable-upload-package.yml@v1
    with:
      s3_bucket: 'lambdas-prod'
      s3_key: 'processor-${{ github.sha }}.zip'
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

  deploy:
    needs: package
    uses: your-org/reusable-actions/.github/workflows/_reusable-terraform.yml@v1
    with:
      workspace: production
      terraform_vars: |
        {
          "lambda_s3_bucket": "${{ needs.package.outputs.s3_bucket }}",
          "lambda_s3_key": "${{ needs.package.outputs.s3_key }}"
        }
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

### 5. Multi-Lambda

```yaml
jobs:
  package-processor:
    uses: your-org/reusable-actions/.github/workflows/_reusable-upload-package.yml@v1
    with:
      project_root: './lambdas/processor'
      artifact_name: 'processor-lambda'
      s3_bucket: 'my-lambdas'
      s3_key: 'processor.zip'
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

  package-validator:
    uses: your-org/reusable-actions/.github/workflows/_reusable-upload-package.yml@v1
    with:
      project_root: './lambdas/validator'
      artifact_name: 'validator-lambda'
      s3_bucket: 'my-lambdas'
      s3_key: 'validator.zip'
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

### 6. Com Versionamento Semântico

```yaml
on:
  push:
    tags: ['v*']

jobs:
  package:
    uses: your-org/reusable-actions/.github/workflows/_reusable-upload-package.yml@v1
    with:
      s3_bucket: 'prod-lambdas'
      s3_key: 'releases/app-${{ github.ref_name }}.zip'
      artifact_name: 'lambda-${{ github.ref_name }}'
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

---

## 🛠️ Script de Package

### Exemplo: package_lambda.sh

```bash
#!/bin/bash
set -euo pipefail

echo "📦 Building Lambda package..."

# Limpar builds anteriores
rm -rf dist/ lambda.zip

# Criar diretório dist
mkdir -p dist

# Instalar dependências
pip install -r requirements.txt -t dist/ --upgrade

# Copiar código fonte
cp -r src/* dist/

# Criar zip
cd dist
zip -r ../lambda.zip . -q
cd ..

echo "✅ Package created: lambda.zip"
ls -lh lambda.zip
```

### Exemplo: Com layers

```bash
#!/bin/bash
set -euo pipefail

# Separar código de dependências
mkdir -p dist/python dist/code

# Dependencies (para layer)
pip install -r requirements.txt -t dist/python/lib/python3.11/site-packages/

# Código
cp -r src/* dist/code/

# Criar zips
cd dist/python && zip -r ../../layer.zip . -q && cd ../..
cd dist/code && zip -r ../../function.zip . -q && cd ../..

echo "✅ Created: layer.zip and function.zip"
```

---

## 🎯 Casos de Uso

### S3 Key Patterns

```yaml
# Por commit
s3_key: 'app-${{ github.sha }}.zip'

# Por branch
s3_key: '${{ github.ref_name }}/app-latest.zip'

# Por tag/release
s3_key: 'releases/app-${{ github.ref_name }}.zip'

# Timestamped
s3_key: 'app-${{ github.run_number }}-${{ github.sha }}.zip'
```

### Validação Desabilitada

```yaml
# Para builds experimentais
validate_artifact: false
```

### Múltiplas Regiões

```yaml
jobs:
  upload-us-east:
    uses: ...
    with:
      s3_bucket: 'lambdas-us-east-1'
      aws_region: 'us-east-1'

  upload-eu-west:
    uses: ...
    with:
      s3_bucket: 'lambdas-eu-west-1'
      aws_region: 'eu-west-1'
```

---

## 🔄 Pipeline Completo Lambda

```yaml
name: Lambda CI/CD

on:
  push:
    branches: [main]

jobs:
  test:
    uses: your-org/reusable-actions/.github/workflows/_reusable-build-python.yml@v1
    with:
      project_dir: './lambda'
      run_tests: true

  sonar:
    needs: test
    uses: your-org/reusable-actions/.github/workflows/_reusable-sonar-python.yml@v1
    with:
      sonar_org: 'my-org'
      sonar_project_key: 'lambda'
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

  package:
    needs: [test, sonar]
    uses: your-org/reusable-actions/.github/workflows/_reusable-upload-package.yml@v1
    with:
      project_root: './lambda'
      s3_bucket: 'prod-lambdas'
      s3_key: 'app-${{ github.sha }}.zip'
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

  deploy:
    needs: package
    uses: your-org/reusable-actions/.github/workflows/_reusable-terraform.yml@v1
    with:
      workspace: production
      terraform_vars: |
        {
          "lambda_s3_bucket": "${{ needs.package.outputs.s3_bucket }}",
          "lambda_s3_key": "${{ needs.package.outputs.s3_key }}"
        }
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

---

## 🔍 Troubleshooting

### Artifact não encontrado

```yaml
# ✅ Ative validação
validate_artifact: true

# Verifique o path no script
artifact_path: 'dist/lambda.zip'  # Path correto?
```

### Script de package falha

```bash
# Adicione debug no script
set -x  # Print commands

# Verifique permissões
chmod +x ./scripts/package_lambda.sh
```

### Upload S3 falha

```yaml
# Verifique credenciais
# Teste manual:
aws s3 ls s3://my-bucket/

# Verifique permissões IAM
# Necessário: s3:PutObject
```

### Zip muito grande

```bash
# Remova arquivos desnecessários
zip -r lambda.zip . -x "*.pyc" -x "*__pycache__*" -x "*.git*"

# Ou use Layer para dependencies
```

---

## 📖 Ver Também

- [Python Workflows](./PYTHON.md)
- [Terraform Deployment](./TERRAFORM.md)
- [Pipelines Completos](./PIPELINES.md)
- [Troubleshooting](./TROUBLESHOOTING.md)
