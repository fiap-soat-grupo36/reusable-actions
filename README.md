# 📚 Reusable Workflows - Guia Completo

> **Versão:** 2.0  
> **Última atualização:** Janeiro 2026

## 📖 Documentação por Categoria

### 🏗️ Infrastructure & Deployment
- **[Terraform](./TERRAFORM.md)** - Deploy de infraestrutura e aplicações
- **[Docker Build](./DOCKER.md)** - Build e push de imagens Docker

### 🐍 Python Workflows
- **[Python Build & Test](./PYTHON.md)** - Build, lint e testes Python
- **[SonarCloud Python](./PYTHON.md#sonarcloud-analysis)** - Análise de qualidade Python

### ☕ Java Workflows
- **[SonarCloud Java](./JAVA.md)** - Análise de qualidade Java com Maven

### 📦 Package Management
- **[Upload Lambda Package](./DEPLOYMENT.md)** - Build e upload de pacotes Lambda

### 🤖 Automation
- **[Create Pull Request](./AUTOMATION.md)** - Criação automática de PRs

### 🔄 Pipelines Completos
- **[End-to-End Examples](./PIPELINES.md)** - Exemplos completos de CI/CD

### 🆘 Suporte
- **[Troubleshooting & Best Practices](./TROUBLESHOOTING.md)** - Solução de problemas e boas práticas

---

## 🚀 Quick Start

### 1. Terraform Deployment
```yaml
uses: your-org/reusable-actions/.github/workflows/_reusable-terraform.yml@v1
with:
  workspace: production
  terraform_vars: '{"image_tag": "v1.0"}'
secrets:
  AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
  AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

### 2. Python Build
```yaml
uses: your-org/reusable-actions/.github/workflows/_reusable-build-python.yml@v1
with:
  python_version: '3.11'
  run_tests: true
```

### 3. Docker Build
```yaml
uses: your-org/reusable-actions/.github/workflows/_reusable-dockerhub.yml@v1
with:
  modules: '["api", "worker"]'
  registry_namespace: 'mycompany'
secrets:
  DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
  DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}
```

---

## 📊 Workflows Disponíveis

| Workflow | Arquivo | Documentação | Uso Principal |
|----------|---------|--------------|---------------|
| Terraform | `_reusable-terraform.yml` | [📖](./TERRAFORM.md) | Deploy infra/apps |
| Python Build | `_reusable-build-python.yml` | [📖](./PYTHON.md) | CI Python |
| Sonar Python | `_reusable-sonar-python.yml` | [📖](./PYTHON.md#sonarcloud) | Quality gate Python |
| Sonar Java | `_reusable-sonar-java.yml` | [📖](./JAVA.md) | Quality gate Java |
| Docker Build | `_reusable-dockerhub.yml` | [📖](./DOCKER.md) | Build/push containers |
| Upload Package | `_reusable-upload-package.yml` | [📖](./DEPLOYMENT.md) | Lambda packaging |
| Create PR | `_reusable-create-pr.yml` | [📖](./AUTOMATION.md) | Auto PR |

---

## 🎯 Casos de Uso Comuns

### Microservices Java
```
Build → Sonar Java → Docker Build → Terraform Deploy
```
👉 [Ver exemplo completo](./PIPELINES.md#java-microservices)

### Lambda Python
```
Build Python → Sonar Python → Upload Package → Terraform Deploy
```
👉 [Ver exemplo completo](./PIPELINES.md#python-lambda)

### Feature Branch
```
Build → Tests → Create PR
```
👉 [Ver exemplo completo](./PIPELINES.md#feature-branch)

---

## 🔒 Segurança

Todos os workflows seguem as melhores práticas de segurança:
- ✅ Secrets nunca expostos em logs
- ✅ Uso de actions oficiais
- ✅ Credentials via GitHub Secrets
- ✅ Minimal permissions

---

## 🆕 Novidades v2.0

- ✅ **Terraform**: Variáveis e secrets dinâmicos
- ✅ **Docker**: Estratégias de tagging flexíveis
- ✅ **Sonar Java**: Plugin Maven oficial (sempre atualizado)
- ✅ **Sonar Python**: Action oficial SonarCloud
- ✅ **Upload Package**: Credenciais AWS seguras
- ✅ **Build Python**: Outputs e controle de lint
- ✅ **Create PR**: Reviewers, labels, draft mode

---

## 📞 Suporte

- 📖 [Documentação completa](./README.md)
- 🐛 [Issues](https://github.com/your-org/reusable-actions/issues)
- 💬 [Discussions](https://github.com/your-org/reusable-actions/discussions)

---

## 📄 Licença

MIT License - ver [LICENSE](../../LICENSE)
