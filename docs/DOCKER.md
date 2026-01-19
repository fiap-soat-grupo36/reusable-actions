# 🐳 Docker Build & Push

**Arquivo:** `_reusable-dockerhub.yml`

## 📋 Características

- ✅ Build paralelo de múltiplos módulos
- ✅ 4 estratégias de tagging flexíveis
- ✅ Dockerfile path pattern customizável
- ✅ Exclusão de módulos específicos
- ✅ Cache GHA otimizado
- ✅ Build summary automático

## 📝 Inputs

| Input | Obrigatório | Default | Descrição |
|-------|-------------|---------|-----------|
| `modules` | ✅ | - | JSON array de módulos (e.g., `["api", "worker"]`) |
| `registry_namespace` | ✅ | - | Namespace/username do registry |
| `dockerfile_pattern` | ❌ | `./{module}/Dockerfile` | Pattern do Dockerfile (`{module}` = placeholder) |
| `tag_strategy` | ❌ | `both` | Estratégia: `branch`, `sha`, `both`, `custom` |
| `custom_tags` | ❌ | `[]` | Tags customizadas (JSON array) |
| `push_images` | ❌ | `false` | Fazer push ao registry |
| `max_parallel` | ❌ | `4` | Máximo de builds paralelos |
| `exclude_modules` | ❌ | `["shared-library"]` | Módulos a excluir |

## 🔐 Secrets

| Secret | Obrigatório |
|--------|-------------|
| `DOCKERHUB_USERNAME` | ✅ |
| `DOCKERHUB_TOKEN` | ✅ |

---

## 🎯 Estratégias de Tagging

| Estratégia | Tags Geradas | Exemplo | Uso |
|------------|--------------|---------|-----|
| `branch` | `namespace/module:branch` | `mycompany/api:main` | Simples, por branch |
| `sha` | `namespace/module:sha` | `mycompany/api:abc1234` | Immutable |
| `both` | `namespace/module:branch`<br>`namespace/module:branch-sha` | `mycompany/api:main`<br>`mycompany/api:main-abc1234` | **Recomendado** |
| `custom` | Tags definidas | `mycompany/api:latest`<br>`mycompany/api:v1.0` | Releases |

---

## 📚 Exemplos

### 1. Build Simples

```yaml
name: Build Docker Images

on:
  push:
    branches: [main]

jobs:
  docker:
    uses: your-org/reusable-actions/.github/workflows/_reusable-dockerhub.yml@v1
    with:
      modules: '["api", "worker", "scheduler"]'
      registry_namespace: 'mycompany'
      push_images: true
    secrets:
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}
```

**Tags geradas (strategy=both):**
- `mycompany/api:main`
- `mycompany/api:main-abc1234`
- `mycompany/worker:main`
- `mycompany/worker:main-abc1234`
- `mycompany/scheduler:main`
- `mycompany/scheduler:main-abc1234`

### 2. Tags Customizadas (Release)

```yaml
name: Release Build

on:
  push:
    tags: ['v*']

jobs:
  docker:
    uses: your-org/reusable-actions/.github/workflows/_reusable-dockerhub.yml@v1
    with:
      modules: '["frontend", "backend"]'
      registry_namespace: 'mycompany'
      tag_strategy: 'custom'
      custom_tags: '["latest", "${{ github.ref_name }}", "stable"]'
      push_images: true
    secrets:
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}
```

**Tags geradas:**
- `mycompany/frontend:latest`
- `mycompany/frontend:v1.2.0`
- `mycompany/frontend:stable`

### 3. Estrutura de Dockerfile Customizada

```yaml
# Estrutura do projeto:
# services/
#   api/
#     docker/Dockerfile
#   worker/
#     docker/Dockerfile

jobs:
  docker:
    uses: your-org/reusable-actions/.github/workflows/_reusable-dockerhub.yml@v1
    with:
      modules: '["api", "worker"]'
      dockerfile_pattern: './services/{module}/docker/Dockerfile'
      registry_namespace: 'mycompany'
      tag_strategy: 'both'
      push_images: true
    secrets:
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}
```

### 4. Build sem Push (Validação em PR)

```yaml
name: PR Docker Build

on:
  pull_request:

jobs:
  docker-validation:
    uses: your-org/reusable-actions/.github/workflows/_reusable-dockerhub.yml@v1
    with:
      modules: '["api", "worker"]'
      registry_namespace: 'mycompany'
      push_images: false  # Apenas valida que build funciona
      exclude_modules: '["shared-library", "common"]'
    secrets:
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}
```

### 5. Multi-ambiente com Strategies

```yaml
name: Multi-Environment Build

on:
  push:
    branches: [develop, staging, main]

jobs:
  docker:
    uses: your-org/reusable-actions/.github/workflows/_reusable-dockerhub.yml@v1
    with:
      modules: '["api", "worker"]'
      registry_namespace: 'mycompany'
      tag_strategy: ${{ github.ref == 'refs/heads/main' && 'custom' || 'both' }}
      custom_tags: ${{ github.ref == 'refs/heads/main' && '["latest", "stable"]' || '[]' }}
      push_images: true
      max_parallel: 6  # Mais builds simultâneos
    secrets:
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}
```

### 6. Monorepo com Seleção Dinâmica

```yaml
name: Smart Build

on:
  push:
    paths:
      - 'services/**'
      - '.github/workflows/**'

jobs:
  detect-changes:
    runs-on: ubuntu-latest
    outputs:
      modules: ${{ steps.filter.outputs.modules }}
    steps:
      - uses: actions/checkout@v4
      
      - name: Detect changed modules
        id: filter
        run: |
          # Lógica para detectar quais módulos mudaram
          CHANGED_MODULES='["api", "worker"]'
          echo "modules=${CHANGED_MODULES}" >> $GITHUB_OUTPUT

  docker:
    needs: detect-changes
    uses: your-org/reusable-actions/.github/workflows/_reusable-dockerhub.yml@v1
    with:
      modules: ${{ needs.detect-changes.outputs.modules }}
      registry_namespace: 'mycompany'
      push_images: true
    secrets:
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}
```

### 7. Com Diferentes Namespaces por Ambiente

```yaml
jobs:
  docker:
    uses: your-org/reusable-actions/.github/workflows/_reusable-dockerhub.yml@v1
    with:
      modules: '["api"]'
      registry_namespace: ${{ github.ref == 'refs/heads/main' && 'mycompany' || 'mycompany-dev' }}
      tag_strategy: 'branch'
      push_images: true
    secrets:
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}
```

---

## 🎯 Casos de Uso

### Estruturas de Projeto Suportadas

#### Padrão (Dockerfile na raiz do módulo)
```
project/
├── api/
│   └── Dockerfile
├── worker/
│   └── Dockerfile
```
```yaml
dockerfile_pattern: './{module}/Dockerfile'  # Default
```

#### Docker em subdiretório
```
project/
├── api/
│   └── docker/
│       └── Dockerfile
```
```yaml
dockerfile_pattern: './{module}/docker/Dockerfile'
```

#### Services em pasta separada
```
project/
├── services/
│   ├── api/
│   │   └── Dockerfile
│   └── worker/
│       └── Dockerfile
```
```yaml
dockerfile_pattern: './services/{module}/Dockerfile'
```

### Exclusão de Módulos

```yaml
# Excluir bibliotecas compartilhadas
exclude_modules: '["shared-library", "common", "utils"]'

# Módulos excluídos não serão buildados mesmo se listados em 'modules'
```

### Performance: max_parallel

```yaml
# Conservative (economiza runners)
max_parallel: 2

# Balanced (recomendado)
max_parallel: 4

# Aggressive (mais rápido, mais runners)
max_parallel: 8
```

---

## 🔄 Pipeline Completo

```yaml
name: Build and Deploy

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run tests
        run: mvn test

  docker:
    needs: test
    uses: your-org/reusable-actions/.github/workflows/_reusable-dockerhub.yml@v1
    with:
      modules: '["api-gateway", "user-service", "order-service"]'
      registry_namespace: 'mycompany'
      tag_strategy: 'both'
      push_images: true
      exclude_modules: '["shared-lib"]'
    secrets:
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}

  deploy:
    needs: docker
    uses: your-org/reusable-actions/.github/workflows/_reusable-terraform.yml@v1
    with:
      workspace: production
      terraform_vars: |
        {
          "image_tag": "${{ github.ref_name }}-${{ github.sha }}"
        }
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

---

## 🔍 Troubleshooting

### Módulos não encontrados

```yaml
# ❌ Erro: String, não JSON
modules: "api,worker"

# ✅ Correto: JSON array
modules: '["api", "worker"]'
```

### Dockerfile não encontrado

```bash
# Verifique o pattern e estrutura
# Debug: liste arquivos no workflow
- run: find . -name "Dockerfile"

# Ajuste o pattern conforme necessário
dockerfile_pattern: './apps/{module}/Dockerfile'
```

### Tags não customizadas sendo aplicadas

```yaml
# Se usar custom, DEVE passar custom_tags
tag_strategy: 'custom'
custom_tags: '["v1.0"]'  # Obrigatório!
```

### Push falha com credenciais

```yaml
# Verifique que secrets existem e são válidos
# Test login manual:
- name: Test login
  run: |
    echo "${{ secrets.DOCKERHUB_TOKEN }}" | docker login -u "${{ secrets.DOCKERHUB_USERNAME }}" --password-stdin
```

### Build muito lento

```yaml
# Aumente paralelismo
max_parallel: 8

# Ou reduza módulos sendo buildados
# Use detecção de mudanças (exemplo 6)
```

---

## 📖 Ver Também

- [Terraform Deployment](./TERRAFORM.md)
- [Pipelines Completos](./PIPELINES.md)
- [Troubleshooting](./TROUBLESHOOTING.md)
