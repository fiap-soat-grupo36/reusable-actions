# 🤖 Automation - Create Pull Request

**Arquivo:** `_reusable-create-pr.yml`

## 📋 Características

- ✅ Criação automática de PRs
- ✅ Customização completa (title, body, draft)
- ✅ Auto-adiciona reviewers e labels
- ✅ Previne PRs duplicados
- ✅ Usa API do GitHub

## 📝 Inputs

| Input | Obrigatório | Default | Descrição |
|-------|-------------|---------|-----------|
| `base` | ✅ | - | Branch destino (e.g., `main`) |
| `head` | ❌ | branch atual | Branch origem |
| `title` | ❌ | Auto-gerado | Título do PR |
| `body` | ❌ | Auto-gerado | Descrição do PR |
| `draft` | ❌ | `false` | Criar como draft |
| `reviewers` | ❌ | `[]` | JSON array de usernames |
| `labels` | ❌ | `[]` | JSON array de labels |

## 🔐 Secrets

Nenhum necessário (usa `GITHUB_TOKEN` automático)

---

## 📚 Exemplos

### 1. PR Automático Simples

```yaml
name: Auto PR

on:
  push:
    branches: [feature/*]

jobs:
  create-pr:
    uses: fiap-soat-grupo36/reusable-actions/.github/workflows/_reusable-create-pr.yml@v1
    with:
      base: develop
      # head usa branch atual automaticamente
```

**Resultado:**
- Título: `Merge feature/new-api into develop`
- Body: `Automated PR created after successful CI on branch feature/new-api.`

### 2. PR com Reviewers e Labels

```yaml
name: Create PR with Review

on:
  push:
    branches: [feature/*]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run tests
        run: npm test

  create-pr:
    needs: test
    uses: fiap-soat-grupo36/reusable-actions/.github/workflows/_reusable-create-pr.yml@v1
    with:
      base: main
      title: "🚀 Feature: New API Integration"
      body: |
        ## Changes
        - Added new API integration
        - Updated documentation
        
        ## Testing
        All tests passed ✅
      reviewers: '["tech-lead", "senior-dev", "devops-team"]'
      labels: '["feature", "needs-review", "api"]'
```

### 3. Draft PR

```yaml
jobs:
  create-draft-pr:
    uses: fiap-soat-grupo36/reusable-actions/.github/workflows/_reusable-create-pr.yml@v1
    with:
      base: main
      title: "WIP: Refactoring Authentication"
      body: |
        ## Work in Progress
        - [ ] Refactor auth module
        - [ ] Add tests
        - [ ] Update docs
      draft: true
      labels: '["work-in-progress", "refactoring"]'
```

### 4. Release PR Automático

```yaml
name: Create Release PR

on:
  push:
    branches: [develop]

jobs:
  create-pr:
    uses: fiap-soat-grupo36/reusable-actions/.github/workflows/_reusable-create-pr.yml@v1
    with:
      base: main
      head: develop
      title: "🚀 Release: v${{ github.ref_name }}"
      body: |
        ## Release Notes
        
        ### Features
        - New feature A
        - Enhancement B
        
        ### Bug Fixes
        - Fixed issue #123
        - Fixed issue #456
        
        ### Breaking Changes
        None
        
        ---
        **Ready for:** Production deployment
      reviewers: '["cto", "lead-engineer"]'
      labels: '["release", "production", "high-priority"]'
      draft: false
```

### 5. Hotfix PR

```yaml
name: Hotfix PR

on:
  push:
    branches: [hotfix/*]

jobs:
  create-pr:
    uses: fiap-soat-grupo36/reusable-actions/.github/workflows/_reusable-create-pr.yml@v1
    with:
      base: main
      title: "🔥 HOTFIX: ${{ github.ref_name }}"
      body: |
        ## Hotfix Required
        
        **Priority:** URGENT
        **Branch:** ${{ github.ref_name }}
        
        ### Issue
        Critical production bug
        
        ### Fix
        [Description of fix]
        
        ### Testing
        Tested locally ✅
      reviewers: '["on-call-engineer", "team-lead"]'
      labels: '["hotfix", "critical", "production"]'
```

### 6. Multi-Environment Staging

```yaml
name: Progressive Deployment PR

on:
  workflow_dispatch:

jobs:
  # Develop → Staging
  pr-to-staging:
    uses: fiap-soat-grupo36/reusable-actions/.github/workflows/_reusable-create-pr.yml@v1
    with:
      base: staging
      head: develop
      title: "📦 Deploy to Staging"
      labels: '["deployment", "staging"]'
      reviewers: '["qa-lead"]'

  # Staging → Production (draft)
  pr-to-prod:
    uses: fiap-soat-grupo36/reusable-actions/.github/workflows/_reusable-create-pr.yml@v1
    with:
      base: main
      head: staging
      title: "🚀 Deploy to Production"
      draft: true
      labels: '["deployment", "production", "needs-approval"]'
      reviewers: '["cto", "devops-lead"]'
```

### 7. Após CI Success

```yaml
name: Feature Branch Workflow

on:
  push:
    branches: [feature/*]

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Lint
        run: npm run lint
      - name: Test
        run: npm test
      - name: Build
        run: npm run build

  create-pr:
    needs: ci
    uses: fiap-soat-grupo36/reusable-actions/.github/workflows/_reusable-create-pr.yml@v1
    with:
      base: develop
      title: "✨ ${{ github.event.head_commit.message }}"
      body: |
        ## Automated PR
        
        **Commit:** ${{ github.sha }}
        **Author:** @${{ github.actor }}
        
        ### CI Status
        ✅ Lint passed
        ✅ Tests passed
        ✅ Build successful
        
        Ready for review!
      labels: '["auto-pr", "ci-passed"]'
      reviewers: '["team-lead"]'
```

---

## 🎯 Casos de Uso

### Título Dinâmico

```yaml
title: "Feature: ${{ github.event.head_commit.message }}"
title: "Release ${{ github.ref_name }}"
title: "[AUTO] Merge ${{ github.ref_name }}"
```

### Body com Template

```yaml
body: |
  ## Description
  ${{ github.event.head_commit.message }}
  
  ## Changes
  - Item 1
  - Item 2
  
  ## Checklist
  - [ ] Tests added
  - [ ] Docs updated
  - [ ] Breaking changes documented
  
  ## Related Issues
  Closes #${{ github.event.issue.number }}
```

### Reviewers por Time

```yaml
# Indivíduos
reviewers: '["john", "jane"]'

# Times (use team slug)
reviewers: '["frontend-team", "backend-team"]'
```

### Labels Condicionais

```yaml
labels: ${{ github.ref == 'refs/heads/hotfix/*' && '["hotfix", "urgent"]' || '["feature"]' }}
```

---

## 🔄 Workflows Integrados

### Feature Branch → Auto PR

```yaml
name: Feature Workflow

on:
  push:
    branches: [feature/*]

jobs:
  build:
    uses: fiap-soat-grupo36/reusable-actions/.github/workflows/_reusable-build-python.yml@v1
    with:
      run_tests: true
      run_lint: true

  create-pr:
    needs: build
    uses: fiap-soat-grupo36/reusable-actions/.github/workflows/_reusable-create-pr.yml@v1
    with:
      base: develop
      labels: '["feature"]'
      reviewers: '["tech-lead"]'
```

### Dependabot PRs Auto-Label

```yaml
name: Dependabot PR Enhancement

on:
  pull_request:
    types: [opened]
    branches: [main]

jobs:
  enhance-pr:
    if: github.actor == 'dependabot[bot]'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/github-script@v6
        with:
          script: |
            await github.rest.issues.addLabels({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
              labels: ['dependencies', 'automated']
            });
```

---

## ⚙️ Comportamento

### PR Já Existe
Se um PR aberto já existe da mesma `head` para a mesma `base`:
- ✅ Detecta o PR existente
- ✅ Log: `PR already exists: <url>`
- ❌ Não cria duplicado

### Branch Não Existe
Se o branch `head` não existe:
- ❌ Workflow falha
- 📝 Erro: `Reference does not exist`

### Reviewers Inválidos
Se reviewer não existe:
- ⚠️ API retorna erro
- 📝 Log mostra falha
- ✅ PR ainda é criado (sem reviewers)

---

## 🔍 Troubleshooting

### PR não criado

```yaml
# Verifique que head branch existe
# Verifique permissões do GITHUB_TOKEN
# Verifique que não há PR aberto já
```

### Reviewers não adicionados

```yaml
# ✅ JSON válido
reviewers: '["user1", "user2"]'

# ❌ String simples
reviewers: "user1,user2"

# Verifique que usernames existem
# Verifique permissões (teams precisam de acesso)
```

### Labels não aplicados

```yaml
# Verifique que labels existem no repo
# Crie labels antes:
- name: Create label
  run: |
    gh label create "auto-pr" --color "0366d6" || true
  env:
    GH_TOKEN: ${{ github.token }}
```

### Draft não funciona

```yaml
# Verifique tipo de input
draft: true  # Boolean, não string!

# ❌ Errado
draft: "true"
```

---

## 🎨 Boas Práticas

### 1. Use Emojis no Título
```yaml
title: "✨ New Feature"
title: "🐛 Bug Fix"
title: "🚀 Release"
title: "🔥 Hotfix"
title: "📝 Docs"
```

### 2. Template de Body Completo
```yaml
body: |
  ## 📋 Description
  [What changed]
  
  ## 🎯 Motivation
  [Why this change]
  
  ## 🧪 Testing
  - [ ] Unit tests
  - [ ] Integration tests
  - [ ] Manual testing
  
  ## 📸 Screenshots
  [If applicable]
  
  ## 🔗 Related
  Closes #123
```

### 3. Auto-assign por Path
```yaml
# Se mudar frontend/
reviewers: '["frontend-team"]'

# Se mudar backend/
reviewers: '["backend-team"]'
```

---

## 📖 Ver Também

- [Pipelines Completos](./PIPELINES.md)
- [GitHub Actions: Creating PRs](https://docs.github.com/en/rest/pulls/pulls#create-a-pull-request)
- [Troubleshooting](./TROUBLESHOOTING.md)
