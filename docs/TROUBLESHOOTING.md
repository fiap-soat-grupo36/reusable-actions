# 🆘 Troubleshooting & Best Practices

## 🔍 Troubleshooting por Workflow

### Terraform

#### ❌ Variáveis não reconhecidas
```yaml
# ❌ JSON inválido
terraform_vars: "{key: value}"

# ✅ JSON válido
terraform_vars: '{"key": "value"}'
```

#### ❌ Secret vazio
```yaml
# Problema: Secret mapeado não existe
terraform_secrets: '{"api_key": "NONEXISTENT_SECRET"}'

# Solução: Verifique que secret existe
# GitHub → Settings → Secrets → Actions
```

#### ❌ Workspace não muda
```yaml
# Workspace é criado automaticamente se não existe
# Verifique logs para confirmar workspace correto
```

---

### Docker Build

#### ❌ Módulos não encontrados
```yaml
# ❌ String, não JSON!
modules: "api,worker"

# ✅ JSON array válido
modules: '["api", "worker"]'
```

#### ❌ Dockerfile não encontrado
```bash
# Debug: liste arquivos
- run: find . -name "Dockerfile"

# Ajuste o pattern
dockerfile_pattern: './apps/{module}/Dockerfile'
```

#### ❌ Push falha
```yaml
# Verifique credenciais
# Test login:
- run: echo "${{ secrets.DOCKERHUB_TOKEN }}" | docker login -u "${{ secrets.DOCKERHUB_USERNAME }}" --password-stdin

# Verifique namespace existe
# Crie organization no DockerHub se necessário
```

#### ❌ Cache não funciona
```yaml
# GitHub Actions cache tem limite de 10GB
# Limpe caches antigos:
# GitHub → Actions → Caches
```

---

### SonarCloud (Python)

#### ❌ Coverage não encontrado
```yaml
# ✅ Path relativo ao project_dir
coverage_paths: 'coverage.xml'

# ❌ Path absoluto
coverage_paths: '/home/runner/work/coverage.xml'
```

#### ❌ Sonar timeout
```yaml
# Projetos grandes podem demorar
# Aumente o timeout (default: 5min)
# Ou divida em módulos menores
```

#### ❌ Token inválido
```yaml
# Verifique que SONAR_TOKEN é válido
# Gere novo em: sonarcloud.io → My Account → Security
```

---

### SonarCloud (Java)

#### ❌ Coverage zerada
```yaml
# 1. Certifique-se de upload correto
- uses: actions/upload-artifact@v4
  with:
    name: jacoco-reports  # Nome exato!
    path: '**/target/site/jacoco/jacoco.xml'

# 2. Ative download
download_jacoco_artifacts: true
```

#### ❌ JaCoCo não gerado
```xml
<!-- Adicione no pom.xml: -->
<plugin>
  <groupId>org.jacoco</groupId>
  <artifactId>jacoco-maven-plugin</artifactId>
  <version>0.8.11</version>
  <executions>
    <execution>
      <goals>
        <goal>prepare-agent</goal>
        <goal>report</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

#### ❌ Maven timeout
```yaml
# Aumente memória
maven_args: '-Xmx2048m -DskipITs'
```

---

### Python Build

#### ❌ Paths incorretos no upload
```yaml
# ✅ Inclua project_dir no path
path: |
  ${{ inputs.project_dir }}/coverage.xml
  ${{ inputs.project_dir }}/htmlcov

# ❌ Path relativo sem project_dir
path: coverage.xml  # Pode não existir na raiz
```

#### ❌ Lint falhando o build
```yaml
# Se lint não é crítico
fail_on_lint_error: false

# Ou desabilite temporariamente
run_lint: false
```

#### ❌ Tests path incorreto
```yaml
# Estrutura esperada: project_dir/tests/
project_dir: './src'  # Onde está o código

# Se testes em local diferente, ajuste estrutura
```

---

### Upload Package

#### ❌ Artifact não encontrado
```yaml
# ✅ Ative validação
validate_artifact: true

# Verifique o script de build
# Debug:
- run: ls -la ${{ inputs.artifact_path }}
```

#### ❌ Upload S3 falha
```bash
# Teste credenciais:
aws s3 ls s3://my-bucket/

# Verifique permissões IAM:
# Necessário: s3:PutObject, s3:ListBucket
```

#### ❌ Zip muito grande
```bash
# Lambda tem limite de 50MB (direct upload)
# Ou 250MB (via S3)

# Remova arquivos desnecessários:
zip -r lambda.zip . \
  -x "*.pyc" \
  -x "*__pycache__*" \
  -x "*.git*" \
  -x "tests/*"

# Ou use Lambda Layers para dependencies
```

---

### Create PR

#### ❌ PR não criado
```yaml
# Verifique:
# 1. Branch head existe
# 2. Não há PR aberto já
# 3. GITHUB_TOKEN tem permissões
```

#### ❌ Reviewers não adicionados
```yaml
# ✅ JSON válido
reviewers: '["user1", "user2"]'

# ❌ String
reviewers: "user1,user2"

# Verifique que usernames/teams existem
```

#### ❌ Labels não aplicados
```yaml
# Labels devem existir no repo
# Crie antes:
- run: gh label create "auto-pr" --color "0366d6" || true
  env:
    GH_TOKEN: ${{ github.token }}
```

---

## 📝 Best Practices

### 1. Versionamento

```yaml
# ❌ Evite: instável
uses: fiap-soat-grupo36/reusable-actions/.github/workflows/_reusable-terraform.yml@main

# ✅ Use tags em produção
uses: fiap-soat-grupo36/reusable-actions/.github/workflows/_reusable-terraform.yml@v1.0.0

# ✅ Ou use SHA específico
uses: fiap-soat-grupo36/reusable-actions/.github/workflows/_reusable-terraform.yml@abc1234
```

### 2. Secrets Management

```yaml
# ✅ Separe por ambiente
secrets:
  AWS_ACCESS_KEY_ID: ${{ secrets.PROD_AWS_ACCESS_KEY }}
  
# ✅ Use GitHub Environments para proteção
environment: production

# ❌ Nunca hardcode
terraform_vars: '{"api_key": "sk-12345"}'  # NUNCA!
```

### 3. Multi-ambiente

```yaml
strategy:
  matrix:
    environment: [dev, staging, prod]
    include:
      - environment: dev
        auto_apply: true
        replicas: 1
      - environment: staging
        auto_apply: true
        replicas: 2
      - environment: prod
        auto_apply: false  # Manual approval
        replicas: 5
```

### 4. Conditional Jobs

```yaml
# Deploy apenas em push para main
deploy:
  if: github.event_name == 'push' && github.ref == 'refs/heads/main'
  uses: ...

# Skip em draft PRs
build:
  if: github.event.pull_request.draft == false
  uses: ...
```

### 5. Docker: Namespace Organization

```yaml
# ✅ Use organization/team namespace
registry_namespace: 'mycompany'

# ❌ Não use pessoal em produção
registry_namespace: 'johndoe'
```

### 6. Docker: Tag Strategy por Ambiente

```yaml
# Development: branch tags (mutáveis)
tag_strategy: 'branch'

# Staging: branch + SHA (rastreável)
tag_strategy: 'both'

# Production: tags fixas e versionadas
tag_strategy: 'custom'
custom_tags: '["v1.2.0", "latest"]'
```

### 7. Coverage Multi-módulo

```yaml
# ✅ Use glob patterns
coverage_paths: '**/target/site/jacoco/jacoco.xml'

# ✅ Para monorepo com estrutura diferente
coverage_paths: 'modules/*/coverage/jacoco.xml'
```

### 8. Build Matrix: Performance vs Custo

```yaml
# ✅ Conservative (economiza runners)
max_parallel: 2

# ✅ Balanced (recomendado)
max_parallel: 4

# ⚠️ Aggressive (mais rápido, mais caro)
max_parallel: 8
```

### 9. Artifact Retention

```yaml
# Defaults: 90 dias
# Reduza para economizar storage:
- uses: actions/upload-artifact@v4
  with:
    name: build-output
    path: dist/
    retention-days: 7  # 7 dias é suficiente
```

### 10. Fail-fast vs Complete

```yaml
strategy:
  fail-fast: false  # ✅ Continue outros jobs mesmo se um falhar
  matrix:
    service: [api, worker, scheduler]

# Útil para identificar múltiplos problemas de uma vez
```

---

## 🎯 Checklist de Configuração

### Antes de Usar os Workflows

- [ ] **Secrets configurados**
  - [ ] AWS credentials
  - [ ] DockerHub credentials
  - [ ] SonarCloud token
  - [ ] Outros secrets necessários

- [ ] **Repositório configurado**
  - [ ] Labels criados (se usar create-pr)
  - [ ] Branch protection rules
  - [ ] Environments configurados (staging, prod)

- [ ] **Estrutura do projeto**
  - [ ] Dockerfiles nos lugares corretos
  - [ ] Scripts de build existem e funcionam
  - [ ] Terraform modules organizados

- [ ] **CI local testado**
  - [ ] Testes passam localmente
  - [ ] Build funciona localmente
  - [ ] Docker build funciona

---

## 🚨 Erros Comuns

### JSON inválido
```yaml
# ❌ Sem aspas em keys
terraform_vars: "{key: value}"

# ❌ Trailing comma
modules: '["api", "worker",]'

# ✅ JSON válido
terraform_vars: '{"key": "value"}'
modules: '["api", "worker"]'
```

### Paths com Windows
```yaml
# ❌ Backslash
path: "C:\project\dist"

# ✅ Forward slash (funciona em todos OS)
path: "C:/project/dist"

# ✅ Relativo
path: "./dist"
```

### Secrets vazios
```yaml
# Se secret não existe, value será ""
# Sempre valide se critical:
- name: Validate secrets
  run: |
    if [ -z "${{ secrets.CRITICAL_SECRET }}" ]; then
      echo "Secret missing!"
      exit 1
    fi
```

---

## 📊 Debugging

### Enable Debug Logging

```yaml
# No repo settings → Secrets → Variables:
# ACTIONS_RUNNER_DEBUG = true
# ACTIONS_STEP_DEBUG = true

# Ou no workflow:
env:
  ACTIONS_RUNNER_DEBUG: true
  ACTIONS_STEP_DEBUG: true
```

### Inspecionar Outputs

```yaml
- name: Debug outputs
  run: |
    echo "S3 Bucket: ${{ needs.upload.outputs.s3_bucket }}"
    echo "S3 Key: ${{ needs.upload.outputs.s3_key }}"
```

### Listar Environment

```yaml
- name: Debug environment
  run: |
    echo "=== Environment ==="
    env | sort
    echo "=== Workspace ==="
    ls -la
    echo "=== Git ==="
    git branch -a
    git log -1
```

---

## 📖 Recursos Úteis

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Reusable Workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows)
- [Docker Build Push Action](https://github.com/docker/build-push-action)
- [SonarCloud GitHub Action](https://github.com/SonarSource/sonarcloud-github-action)
- [AWS Configure Credentials](https://github.com/aws-actions/configure-aws-credentials)
- [Terraform GitHub Actions](https://www.terraform.io/docs/github-actions)

---

## 💬 Precisa de Ajuda?

1. Verifique esta documentação
2. Procure em [Issues](https://github.com/fiap-soat-grupo36/reusable-actions/issues)
3. Abra nova issue com:
   - Workflow usado
   - Erro completo
   - Logs relevantes
   - Steps para reproduzir
