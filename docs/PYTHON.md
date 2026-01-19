# 🐍 Python Workflows

## 1. Build & Test

**Arquivo:** `_reusable-build-python.yml`

### 📋 Características

- ✅ Testes e lint opcionais
- ✅ Coverage report como artifact
- ✅ Controle de severidade de lint
- ✅ Python version configurável
- ✅ Outputs reutilizáveis

### 📝 Inputs

| Input | Obrigatório | Default | Descrição |
|-------|-------------|---------|-----------|
| `python_version` | ❌ | `3.12` | Versão do Python |
| `project_dir` | ❌ | `./app/` | Diretório do projeto |
| `requirements_file` | ❌ | `requirements.txt` | Arquivo de dependências |
| `run_tests` | ❌ | `true` | Executar testes |
| `run_lint` | ❌ | `true` | Executar lint (flake8) |
| `fail_on_lint_error` | ❌ | `false` | Falhar no erro de lint |

### 📤 Outputs

| Output | Descrição |
|--------|-----------|
| `coverage_artifact` | Nome do artifact de coverage (`coverage-report`) |

---

### 📚 Exemplos

#### Build Completo

```yaml
name: Python CI

on:
  pull_request:
    branches: [main, develop]

jobs:
  build:
    uses: fiap-soat-grupo36/reusable-actions/.github/workflows/_reusable-build-python.yml@v1
    with:
      python_version: '3.11'
      project_dir: './app'
      requirements_file: 'requirements.txt'
      run_tests: true
      run_lint: true
      fail_on_lint_error: true
```

#### Apenas Testes (sem lint)

```yaml
jobs:
  test:
    uses: fiap-soat-grupo36/reusable-actions/.github/workflows/_reusable-build-python.yml@v1
    with:
      python_version: '3.12'
      project_dir: './src'
      run_tests: true
      run_lint: false
```

#### Apenas Lint (CI rápido)

```yaml
jobs:
  lint:
    uses: fiap-soat-grupo36/reusable-actions/.github/workflows/_reusable-build-python.yml@v1
    with:
      project_dir: './'
      run_tests: false
      run_lint: true
      fail_on_lint_error: false  # Warning only
```

#### Usando o Output (Coverage)

```yaml
jobs:
  build:
    uses: fiap-soat-grupo36/reusable-actions/.github/workflows/_reusable-build-python.yml@v1
    with:
      run_tests: true

  sonar:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Download coverage
        uses: actions/download-artifact@v4
        with:
          name: ${{ needs.build.outputs.coverage_artifact }}
      
      - name: Upload to SonarCloud
        run: |
          # Use coverage.xml
```

---

## 2. SonarCloud Analysis {#sonarcloud}

**Arquivo:** `_reusable-sonar-python.yml`

### 📋 Características

- ✅ Usa action oficial do SonarCloud
- ✅ Coverage paths configurável
- ✅ Testes opcionais (pode usar coverage externo)
- ✅ Propriedades Sonar customizáveis

### 📝 Inputs

| Input | Obrigatório | Default | Descrição |
|-------|-------------|---------|-----------|
| `python_version` | ❌ | `3.11` | Versão do Python |
| `sonar_org` | ✅ | - | Organização SonarCloud |
| `sonar_project_key` | ✅ | - | Project key |
| `project_dir` | ❌ | `.` | Diretório do projeto |
| `requirements_file` | ❌ | `requirements.txt` | Dependências |
| `coverage_paths` | ❌ | `coverage.xml` | Path do coverage |
| `sonar_extra_properties` | ❌ | `''` | Propriedades adicionais |
| `run_tests` | ❌ | `true` | Executar testes |

### 🔐 Secrets

| Secret | Obrigatório |
|--------|-------------|
| `SONAR_TOKEN` | ✅ |

---

### 📚 Exemplos

#### Análise Completa

```yaml
name: SonarCloud

on:
  push:
    branches: [main, develop]

jobs:
  sonar:
    uses: fiap-soat-grupo36/reusable-actions/.github/workflows/_reusable-sonar-python.yml@v1
    with:
      python_version: '3.11'
      project_dir: './src'
      sonar_org: 'my-organization'
      sonar_project_key: 'my-project'
      run_tests: true
      sonar_extra_properties: |
        -Dsonar.exclusions=**/*test*.py,**/migrations/**
        -Dsonar.test.inclusions=**/tests/**
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

#### Usando Coverage Externo

```yaml
jobs:
  build:
    uses: fiap-soat-grupo36/reusable-actions/.github/workflows/_reusable-build-python.yml@v1
    with:
      run_tests: true

  sonar:
    needs: build
    uses: fiap-soat-grupo36/reusable-actions/.github/workflows/_reusable-sonar-python.yml@v1
    with:
      sonar_org: 'my-org'
      sonar_project_key: 'my-project'
      run_tests: false  # Usa coverage do job anterior
      coverage_paths: 'app/coverage.xml'
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

#### Com Exclusões e Customizações

```yaml
jobs:
  sonar:
    uses: fiap-soat-grupo36/reusable-actions/.github/workflows/_reusable-sonar-python.yml@v1
    with:
      sonar_org: 'my-org'
      sonar_project_key: 'lambda-function'
      coverage_paths: 'lambda/coverage.xml'
      sonar_extra_properties: |
        -Dsonar.exclusions=**/vendor/**,**/node_modules/**
        -Dsonar.python.version=3.11
        -Dsonar.sourceEncoding=UTF-8
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

---

## 🔄 Pipeline Completo Python

```yaml
name: Python CI/CD

on:
  push:
    branches: [main]
  pull_request:

jobs:
  build:
    uses: fiap-soat-grupo36/reusable-actions/.github/workflows/_reusable-build-python.yml@v1
    with:
      python_version: '3.11'
      run_tests: true
      run_lint: true
      fail_on_lint_error: true

  sonar:
    needs: build
    if: github.event_name == 'push'
    uses: fiap-soat-grupo36/reusable-actions/.github/workflows/_reusable-sonar-python.yml@v1
    with:
      sonar_org: 'my-org'
      sonar_project_key: 'my-app'
      run_tests: false  # Já rodou no build
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

---

## 🔍 Troubleshooting

### Coverage não encontrado
```yaml
# ✅ Path relativo ao project_dir
coverage_paths: 'coverage.xml'

# ❌ Path absoluto
coverage_paths: '/home/runner/work/coverage.xml'
```

### Lint falhando
```yaml
# Se lint não é crítico
fail_on_lint_error: false

# Para desabilitar temporariamente
run_lint: false
```

### Tests path incorreto
```yaml
# Estrutura esperada: project_dir/tests/
# Se diferente, ajuste:
project_dir: './src'  # Onde está o código e /tests
```

---

## 📖 Ver Também

- [Pipelines Completos](./PIPELINES.md)
- [Upload Lambda Package](./DEPLOYMENT.md)
- [Troubleshooting](./TROUBLESHOOTING.md)
