# ☕ Java - SonarCloud Analysis

**Arquivo:** `_reusable-sonar-java.yml`

## 📋 Características

- ✅ Usa plugin Maven oficial do Sonar
- ✅ Sempre atualizado (via `sonar:sonar`)
- ✅ Coverage paths configurável
- ✅ JaCoCo artifacts opcionais
- ✅ Propriedades Sonar customizáveis
- ✅ Skip tests opcional
- ✅ Working directory configurável

## 📝 Inputs

| Input | Obrigatório | Default | Descrição |
|-------|-------------|---------|-----------|
| `java_version` | ❌ | `21` | Versão do Java |
| `sonar_org` | ✅ | - | Organização SonarCloud |
| `sonar_project_key` | ✅ | - | Project key |
| `project_dir` | ❌ | `.` | Diretório do projeto |
| `maven_args` | ❌ | `''` | Args adicionais Maven |
| `coverage_paths` | ❌ | `**/target/site/jacoco/jacoco.xml` | Glob pattern para JaCoCo |
| `sonar_extra_properties` | ❌ | `''` | Propriedades Sonar adicionais |
| `maven_skip_tests` | ❌ | `true` | Pular testes (já rodaram antes) |
| `download_jacoco_artifacts` | ❌ | `true` | Baixar artifacts JaCoCo |

## 🔐 Secrets

| Secret | Obrigatório |
|--------|-------------|
| `SONAR_TOKEN` | ✅ |

---

## 📚 Exemplos

### 1. Análise Completa

```yaml
name: Java Quality Check

on:
  push:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'
          cache: 'maven'
      
      - name: Test with Coverage
        run: mvn clean verify
      
      - name: Upload JaCoCo Reports
        uses: actions/upload-artifact@v4
        with:
          name: jacoco-reports
          path: '**/target/site/jacoco/jacoco.xml'

  sonar:
    needs: test
    uses: fiap-soat-grupo36/reusable-actions/.github/workflows/_reusable-sonar-java.yml@v1
    with:
      java_version: '21'
      sonar_org: 'my-organization'
      sonar_project_key: 'my-java-project'
      maven_skip_tests: true
      download_jacoco_artifacts: true
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

### 2. Com Propriedades Customizadas

```yaml
jobs:
  sonar:
    uses: fiap-soat-grupo36/reusable-actions/.github/workflows/_reusable-sonar-java.yml@v1
    with:
      java_version: '17'
      sonar_org: 'my-org'
      sonar_project_key: 'backend-api'
      sonar_extra_properties: |
        -Dsonar.exclusions=**/*test*.java,**/generated/**
        -Dsonar.test.inclusions=**/src/test/**
        -Dsonar.java.binaries=target/classes
        -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
      maven_args: '-P sonar-profile -DskipITs'
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

### 3. Multi-módulo Maven

```yaml
jobs:
  sonar:
    uses: fiap-soat-grupo36/reusable-actions/.github/workflows/_reusable-sonar-java.yml@v1
    with:
      java_version: '21'
      sonar_org: 'my-org'
      sonar_project_key: 'multi-module-project'
      project_dir: '.'
      coverage_paths: '**/target/site/jacoco/jacoco.xml'
      maven_skip_tests: true
      sonar_extra_properties: |
        -Dsonar.modules=module1,module2,module3
        -Dsonar.java.coveragePlugin=jacoco
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

### 4. Sem Artifacts (Coverage inline)

```yaml
jobs:
  sonar:
    uses: fiap-soat-grupo36/reusable-actions/.github/workflows/_reusable-sonar-java.yml@v1
    with:
      sonar_org: 'my-org'
      sonar_project_key: 'simple-app'
      maven_skip_tests: false  # Roda testes no próprio job
      download_jacoco_artifacts: false
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

### 5. Gradle Project

```yaml
# Para Gradle, crie wrapper customizado ou use action diferente
# Este workflow é específico para Maven

# Alternativa: use sonar-gradle plugin
jobs:
  sonar:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          java-version: '21'
      - name: Sonar with Gradle
        run: ./gradlew sonar
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

---

## 🎯 Casos de Uso

### Pipeline Típico

```yaml
1. Build + Test (gera JaCoCo)
   ↓
2. Upload JaCoCo artifacts
   ↓
3. Sonar (baixa artifacts, analisa)
```

### Coverage Paths Customizados

```yaml
# Estrutura diferente de paths
coverage_paths: 'modules/*/target/jacoco.xml'

# Múltiplos patterns (separados por vírgula)
coverage_paths: '**/jacoco.xml,**/coverage/*.xml'
```

### Exclusões Comuns

```yaml
sonar_extra_properties: |
  -Dsonar.exclusions=**/generated/**,**/dto/**,**/entity/**
  -Dsonar.test.exclusions=**/integration/**
  -Dsonar.cpd.exclusions=**/config/**
```

---

## 🔄 Pipeline Completo Java

```yaml
name: Java CI/CD

on:
  push:
    branches: [main]

jobs:
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
      
      - name: Upload JaCoCo
        uses: actions/upload-artifact@v4
        with:
          name: jacoco-reports
          path: '**/target/site/jacoco/jacoco.xml'

  sonar:
    needs: build
    uses: fiap-soat-grupo36/reusable-actions/.github/workflows/_reusable-sonar-java.yml@v1
    with:
      java_version: '21'
      sonar_org: 'my-org'
      sonar_project_key: 'my-app'
      maven_skip_tests: true
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

  docker:
    needs: [build, sonar]
    uses: fiap-soat-grupo36/reusable-actions/.github/workflows/_reusable-dockerhub.yml@v1
    with:
      modules: '["api", "worker"]'
      registry_namespace: 'mycompany'
      push_images: true
    secrets:
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}
```

---

## 🔍 Troubleshooting

### Coverage zerada

```yaml
# ✅ Certifique-se de upload correto
- uses: actions/upload-artifact@v4
  with:
    name: jacoco-reports  # Nome exato!
    path: '**/target/site/jacoco/jacoco.xml'

# ✅ E ative download
download_jacoco_artifacts: true
```

### JaCoCo não gerado

```bash
# Adicione no pom.xml:
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

### Sonar timeout

```yaml
# Aumente memória Maven
maven_args: '-Xmx2048m'
```

### Módulos não detectados

```yaml
sonar_extra_properties: |
  -Dsonar.modules=module1,module2
  -Dsonar.sources=src/main/java
```

---

## 📖 Ver Também

- [Pipelines Completos](./PIPELINES.md)
- [Docker Build](./DOCKER.md)
- [Troubleshooting](./TROUBLESHOOTING.md)
