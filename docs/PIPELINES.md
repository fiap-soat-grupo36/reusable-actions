# 🔄 Pipelines Completos - End-to-End Examples

## 1. Java Microservices CI/CD {#java-microservices}

### Pipeline Completo

```yaml
name: Java Microservices CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  # 1. Build e Test com Maven
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'
          cache: 'maven'
      
      - name: Build and Test
        run: mvn clean verify
      
      - name: Upload JaCoCo Reports
        uses: actions/upload-artifact@v4
        with:
          name: jacoco-reports
          path: '**/target/site/jacoco/jacoco.xml'

  # 2. SonarCloud Analysis
  sonar:
    needs: build
    if: github.event_name == 'push'
    uses: your-org/reusable-actions/.github/workflows/_reusable-sonar-java.yml@v1
    with:
      java_version: '21'
      sonar_org: 'my-org'
      sonar_project_key: 'microservices'
      maven_skip_tests: true
      download_jacoco_artifacts: true
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

  # 3. Docker Build
  docker:
    needs: [build, sonar]
    if: github.event_name == 'push'
    uses: your-org/reusable-actions/.github/workflows/_reusable-dockerhub.yml@v1
    with:
      modules: '["api-gateway", "user-service", "order-service", "notification-service"]'
      registry_namespace: 'mycompany'
      tag_strategy: ${{ github.ref == 'refs/heads/main' && 'custom' || 'both' }}
      custom_tags: ${{ github.ref == 'refs/heads/main' && '["latest", "stable"]' || '[]' }}
      push_images: true
      exclude_modules: '["shared-library"]'
    secrets:
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}

  # 4. Deploy to Staging
  deploy-staging:
    needs: docker
    if: github.ref == 'refs/heads/develop'
    uses: your-org/reusable-actions/.github/workflows/_reusable-terraform.yml@v1
    with:
      workspace: staging
      environment: staging
      auto_apply: true
      terraform_vars: |
        {
          "image_tag": "${{ github.ref_name }}-${{ github.sha }}",
          "replicas": 2,
          "instance_type": "t3.small"
        }
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.STAGING_AWS_ACCESS_KEY }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.STAGING_AWS_SECRET_KEY }}

  # 5. Deploy to Production
  deploy-production:
    needs: docker
    if: github.ref == 'refs/heads/main'
    uses: your-org/reusable-actions/.github/workflows/_reusable-terraform.yml@v1
    with:
      workspace: production
      environment: prod
      auto_apply: false  # Requer aprovação manual
      terraform_vars: |
        {
          "image_tag": "stable",
          "replicas": 5,
          "instance_type": "t3.medium"
        }
      terraform_secrets: |
        {
          "datadog_api_key": "DATADOG_API_KEY"
        }
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.PROD_AWS_ACCESS_KEY }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.PROD_AWS_SECRET_KEY }}
      DATADOG_API_KEY: ${{ secrets.DATADOG_API_KEY }}
```

**Fluxo:**
1. ✅ Build e testes (gera coverage)
2. 📊 Análise SonarCloud
3. 🐳 Build Docker images
4. 🚀 Deploy automático staging (develop)
5. 🎯 Deploy manual production (main)

---

## 2. Python Lambda CI/CD {#python-lambda}

### Pipeline Completo

```yaml
name: Lambda CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  # 1. Lint e Test
  build:
    uses: your-org/reusable-actions/.github/workflows/_reusable-build-python.yml@v1
    with:
      python_version: '3.11'
      project_dir: './lambda'
      requirements_file: 'requirements.txt'
      run_tests: true
      run_lint: true
      fail_on_lint_error: true

  # 2. SonarCloud
  sonar:
    needs: build
    if: github.event_name == 'push'
    uses: your-org/reusable-actions/.github/workflows/_reusable-sonar-python.yml@v1
    with:
      python_version: '3.11'
      sonar_org: 'my-org'
      sonar_project_key: 'lambda-processor'
      project_dir: './lambda'
      run_tests: false  # Já rodou no build
      coverage_paths: 'lambda/coverage.xml'
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

  # 3. Package e Upload
  package:
    needs: [build, sonar]
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    uses: your-org/reusable-actions/.github/workflows/_reusable-upload-package.yml@v1
    with:
      project_root: './lambda'
      package_script: './scripts/package_lambda.sh'
      artifact_path: 'lambda/dist/lambda.zip'
      artifact_name: 'lambda-processor'
      s3_bucket: 'lambdas-prod'
      s3_key: 'processor-${{ github.sha }}.zip'
      python_version: '3.11'
      validate_artifact: true
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

  # 4. Deploy via Terraform
  deploy:
    needs: package
    uses: your-org/reusable-actions/.github/workflows/_reusable-terraform.yml@v1
    with:
      workspace: production
      environment: prod
      working_directory: './infra'
      auto_apply: true
      terraform_vars: |
        {
          "lambda_s3_bucket": "${{ needs.package.outputs.s3_bucket }}",
          "lambda_s3_key": "${{ needs.package.outputs.s3_key }}",
          "lambda_runtime": "python3.11",
          "lambda_memory": 512,
          "lambda_timeout": 300
        }
      terraform_secrets: |
        {
          "api_key": "EXTERNAL_API_KEY",
          "db_password": "DB_PASSWORD"
        }
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      EXTERNAL_API_KEY: ${{ secrets.EXTERNAL_API_KEY }}
      DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
```

**Fluxo:**
1. ✅ Build, lint e tests
2. 📊 SonarCloud analysis
3. 📦 Package e upload S3
4. 🚀 Deploy Lambda via Terraform

---

## 3. Feature Branch Workflow {#feature-branch}

### Auto-PR após CI Success

```yaml
name: Feature Branch Workflow

on:
  push:
    branches: [feature/*]

jobs:
  # 1. CI completo
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Lint
        run: npm run lint
      
      - name: Test
        run: npm test
      
      - name: Build
        run: npm run build

  # 2. Criar PR automaticamente
  create-pr:
    needs: ci
    uses: your-org/reusable-actions/.github/workflows/_reusable-create-pr.yml@v1
    with:
      base: develop
      title: "✨ ${{ github.event.head_commit.message }}"
      body: |
        ## Automated PR
        
        **Branch:** `${{ github.ref_name }}`
        **Commit:** `${{ github.sha }}`
        **Author:** @${{ github.actor }}
        
        ### CI Status
        ✅ Lint passed
        ✅ Tests passed
        ✅ Build successful
        
        ### Changes
        ${{ github.event.head_commit.message }}
        
        ---
        Ready for review! 🚀
      labels: '["feature", "auto-pr", "ci-passed"]'
      reviewers: '["tech-lead", "senior-dev"]'
```

---

## 4. Multi-Environment Deployment

### Progressive Rollout

```yaml
name: Progressive Deployment

on:
  workflow_dispatch:
    inputs:
      version:
        required: true
        type: string

jobs:
  # 1. Build
  build:
    uses: your-org/reusable-actions/.github/workflows/_reusable-build-python.yml@v1
    with:
      run_tests: true

  # 2. Docker
  docker:
    needs: build
    uses: your-org/reusable-actions/.github/workflows/_reusable-dockerhub.yml@v1
    with:
      modules: '["api"]'
      registry_namespace: 'mycompany'
      tag_strategy: 'custom'
      custom_tags: '["${{ inputs.version }}", "latest"]'
      push_images: true
    secrets:
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}

  # 3. Deploy Dev
  deploy-dev:
    needs: docker
    uses: your-org/reusable-actions/.github/workflows/_reusable-terraform.yml@v1
    with:
      workspace: dev
      environment: dev
      auto_apply: true
      terraform_vars: '{"image_tag": "${{ inputs.version }}", "replicas": 1}'
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.DEV_AWS_ACCESS_KEY }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.DEV_AWS_SECRET_KEY }}

  # 4. Approval Gate
  approval-staging:
    needs: deploy-dev
    runs-on: ubuntu-latest
    environment: staging-approval
    steps:
      - name: Wait for approval
        run: echo "Approved for staging"

  # 5. Deploy Staging
  deploy-staging:
    needs: approval-staging
    uses: your-org/reusable-actions/.github/workflows/_reusable-terraform.yml@v1
    with:
      workspace: staging
      environment: staging
      auto_apply: true
      terraform_vars: '{"image_tag": "${{ inputs.version }}", "replicas": 2}'
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.STAGING_AWS_ACCESS_KEY }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.STAGING_AWS_SECRET_KEY }}

  # 6. Approval Gate Production
  approval-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment: production-approval
    steps:
      - name: Wait for approval
        run: echo "Approved for production"

  # 7. Deploy Production
  deploy-production:
    needs: approval-production
    uses: your-org/reusable-actions/.github/workflows/_reusable-terraform.yml@v1
    with:
      workspace: production
      environment: prod
      auto_apply: true
      terraform_vars: '{"image_tag": "${{ inputs.version }}", "replicas": 5}'
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.PROD_AWS_ACCESS_KEY }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.PROD_AWS_SECRET_KEY }}
```

**Fluxo:**
1. Build + Docker
2. 🟢 Auto-deploy dev
3. ⏸️ Aprovação manual
4. 🟡 Deploy staging
5. ⏸️ Aprovação manual
6. 🔴 Deploy production

---

## 5. Monorepo Multi-Service

### Detecção de Mudanças

```yaml
name: Monorepo CI/CD

on:
  push:
    branches: [main]
    paths:
      - 'services/**'

jobs:
  # 1. Detectar serviços alterados
  detect-changes:
    runs-on: ubuntu-latest
    outputs:
      changed-services: ${{ steps.filter.outputs.services }}
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 2
      
      - name: Detect changed services
        id: filter
        run: |
          CHANGED=$(git diff --name-only HEAD^ HEAD | grep '^services/' | cut -d'/' -f2 | sort -u | jq -R -s -c 'split("\n")[:-1]')
          echo "services=${CHANGED}" >> $GITHUB_OUTPUT
          echo "Changed services: ${CHANGED}"

  # 2. Build apenas serviços alterados
  build:
    needs: detect-changes
    if: needs.detect-changes.outputs.changed-services != '[]'
    strategy:
      matrix:
        service: ${{ fromJSON(needs.detect-changes.outputs.changed-services) }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build ${{ matrix.service }}
        run: |
          cd services/${{ matrix.service }}
          mvn clean verify

  # 3. Docker apenas serviços alterados
  docker:
    needs: [detect-changes, build]
    if: needs.detect-changes.outputs.changed-services != '[]'
    uses: your-org/reusable-actions/.github/workflows/_reusable-dockerhub.yml@v1
    with:
      modules: ${{ needs.detect-changes.outputs.changed-services }}
      dockerfile_pattern: './services/{module}/Dockerfile'
      registry_namespace: 'mycompany'
      tag_strategy: 'both'
      push_images: true
    secrets:
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}

  # 4. Deploy apenas serviços alterados
  deploy:
    needs: docker
    uses: your-org/reusable-actions/.github/workflows/_reusable-terraform.yml@v1
    with:
      workspace: production
      terraform_vars: |
        {
          "updated_services": ${{ needs.detect-changes.outputs.changed-services }},
          "image_tag": "${{ github.sha }}"
        }
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

---

## 6. Hotfix Workflow

### Fast-track Production Fix

```yaml
name: Hotfix Deployment

on:
  push:
    branches: [hotfix/*]

jobs:
  # 1. Fast CI
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Quick tests
        run: npm test -- --coverage=false

  # 2. Build e Deploy Imediato
  docker:
    needs: ci
    uses: your-org/reusable-actions/.github/workflows/_reusable-dockerhub.yml@v1
    with:
      modules: '["api"]'
      registry_namespace: 'mycompany'
      tag_strategy: 'custom'
      custom_tags: '["hotfix", "${{ github.ref_name }}"]'
      push_images: true
    secrets:
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}

  deploy:
    needs: docker
    uses: your-org/reusable-actions/.github/workflows/_reusable-terraform.yml@v1
    with:
      workspace: production
      environment: prod
      auto_apply: true  # Auto-apply para hotfix!
      terraform_vars: '{"image_tag": "hotfix"}'
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

  # 3. Criar PR para backport
  create-pr:
    needs: deploy
    uses: your-org/reusable-actions/.github/workflows/_reusable-create-pr.yml@v1
    with:
      base: main
      title: "🔥 HOTFIX: ${{ github.ref_name }}"
      body: |
        ## Hotfix Deployed to Production
        
        **Branch:** ${{ github.ref_name }}
        **Deployed:** ${{ github.sha }}
        
        ### Issue
        Critical production bug
        
        ### Fix Applied
        [Description]
        
        ---
        ⚠️ This hotfix has been deployed directly to production
      labels: '["hotfix", "production", "deployed"]'
      reviewers: '["tech-lead"]'
```

---

## 📊 Diagrama de Fluxos

### Java Microservices
```
PR → Build → Sonar → [Merge] → Docker → Deploy Staging → Deploy Prod
```

### Python Lambda
```
PR → Build → Sonar → [Merge] → Package → Upload S3 → Deploy
```

### Feature Branch
```
Push → CI → Create PR → [Review] → Merge
```

### Hotfix
```
Push → Quick CI → Docker → Deploy → Backport PR
```

---

## 📖 Ver Também

- [Terraform](./TERRAFORM.md)
- [Docker](./DOCKER.md)
- [Python Workflows](./PYTHON.md)
- [Java Workflows](./JAVA.md)
- [Troubleshooting](./TROUBLESHOOTING.md)
