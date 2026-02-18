# LocalStack - Configuração

## O que é LocalStack?

LocalStack simula serviços AWS localmente, permitindo desenvolvimento e testes sem custos. Com a **API Key**, você ganha persistência de estado entre diferentes workflows e repositórios usando **Cloud Pods**.

## Como funciona

### 1. Persistência com LOCALSTACK_API_KEY
- Estado é salvo automaticamente no Cloud Pod
- Cloud Pod identificado por: `environment` (dev, prod, etc.)
- Estado é restaurado no início da próxima execução
- **Sem API Key**: LocalStack roda local sem persistência

### 2. Compartilhamento entre repos

Com LOCALSTACK_API_KEY configurado, todos os repos que usarem o mesmo `environment` compartilham o estado:

```
Repo 1 (aws-network):
  └─> Cria VPC/EKS → Salva no Cloud Pod "prod"

Repo 2 (infra-kubernetes):
  └─> Carrega Cloud Pod "prod" → EKS já existe! → Deploy pods
```

**Importante**: Todos os repos precisam ter o `LOCALSTACK_API_KEY` configurado como secret.

## Configuração

### Passo 1: Obter API Key do LocalStack

1. Criar conta em https://app.localstack.cloud/
2. Copiar API Key em Account Settings

### Passo 2: Configurar Secret no GitHub

Para **cada repositório**:

```bash
# No GitHub: Settings → Secrets → Actions
Nome: LOCALSTACK_API_KEY
Valor: <sua-api-key>
```

### 3. Usar no Workflow

Todos os workflows **sempre usam LocalStack**. Não é necessário configurar `use_localstack`.

```yaml
jobs:
  deploy:
    uses: fiap-soat-grupo36/reusable-actions/.github/workflows/_reusable-terraform.yml@main
    with:
      environment: prod  # Nome do Cloud Pod
      workspace: prod
      working_directory: ./infra
    secrets:
      LOCALSTACK_API_KEY: ${{ secrets.LOCALSTACK_API_KEY }}
```

## Ambientes (Cloud Pods)

Com LOCALSTACK_API_KEY configurado, cada `environment` cria um Cloud Pod isolado:

| Environment | Uso |
|------------|-----|
| `dev` | Desenvolvimento |
| `staging` | Testes |
| `prod` | Simulação produção |

## Comandos úteis

### Listar Cloud Pods
```bash
localstack pod list
```

### Salvar estado manualmente
```bash
localstack pod save fiap-microservices/prod
```

### Carregar estado manualmente
```bash
localstack pod load fiap-microservices/prod
```

### Deletar Cloud Pod
```bash
localstack pod delete fiap-microservices/prod
```

## Exemplo: Fluxo Multi-Repo

### 1. aws-network (Cria infraestrutura base)
```yaml
# .github/workflows/ci-cd.yml
deploy-network:
  with:
    environment: prod
    workspace: prod
  secrets:
    LOCALSTACK_API_KEY: ${{ secrets.LOCALSTACK_API_KEY }}
# Resultado: VPC, Subnets, EKS criados no Cloud Pod "prod"
```

### 2. infra-kubernetes (Deploy de aplicações)
```yaml
# .github/workflows/deploy.yml
deploy-apps:
  with:
    environment: prod
    workspace: prod
  secrets:
    LOCALSTACK_API_KEY: ${{ secrets.LOCALSTACK_API_KEY }}
# Resultado: Carrega Cloud Pod "prod" → EKS existe → Deploy pods
```

## Troubleshooting

### Erro: "LOCALSTACK_API_KEY not found"
- Adicione o secret no repositório (Settings → Secrets)

### Estado corrupto
```bash
localstack pod delete fiap-microservices/prod
# Re-executar workflow para criar novo estado
```

### Estado não persiste
- Verifique se LOCALSTACK_API_KEY está configurado
- Verifique se API Key é válida
- Sem API Key, LocalStack roda localmente sem persistência

## Custos

LocalStack Cloud oferece:
- **Free tier**: 10 Cloud Pods, 5GB storage
- **Pro**: Pods ilimitados, features avançadas

Link: https://localstack.cloud/pricing
