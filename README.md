# boazIV — Infraestrutura Azure com Terraform

Infraestrutura como código para ambientes **HML** e **PRO** no Microsoft Azure, usando Terraform e GitHub Actions.

---

## 📦 Stack

| Camada | Tecnologia |
|--------|------------|
| Cloud | Microsoft Azure |
| IaC | Terraform (≥ 1.6) |
| CI/CD | GitHub Actions |
| Runtime | Azure Container Apps |
| Container Registry | Azure Container Registry (ACR) |
| Observabilidade | Azure Monitor / Log Analytics |

---

## 🏗️ Estrutura do Repositório

```
.
├── .github/workflows/     # Pipelines CI/CD
├── environments/          # Configurações por ambiente
│   ├── hml/
│   └── pro/
├── modules/               # Módulos Terraform reutilizáveis
│   ├── acr/
│   ├── aca/
│   ├── networking/
│   ├── log-analytics/
│   └── ...
├── main.tf                # Entry point root
├── variables.tf           # Variáveis globais
├── outputs.tf             # Outputs globais
├── providers.tf           # Providers & backend
├── versions.tf            # Version constraints
└── terraform.tfvars.example  # Exemplo de variáveis (sem secrets)
```

---

## 🚀 Como Usar

### Pré-requisitos

- [Terraform](https://developer.hashicorp.com/terraform/downloads) ≥ 1.6
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli) autenticado (`az login`)
- Acesso à subscription Azure alvo
- GitHub repository com secrets configurados (ver abaixo)

### Variáveis de Ambiente

Copie o exemplo e preencha:

```bash
cp terraform.tfvars.example terraform.tfvars
# Edite terraform.tfvars com seus valores
```

> **Nunca** commite `terraform.tfvars` — ele está no `.gitignore`.

### Inicializar

```bash
terraform init
```

### Planejar (HML)

```bash
terraform plan -var-file="environments/hml/terraform.tfvars"
```

### Aplicar (HML)

```bash
terraform apply -var-file="environments/hml/terraform.tfvars"
```

### Planejar (PRO) — Requer aprovação

```bash
terraform plan -var-file="environments/pro/terraform.tfvars"
```

---

## 🔐 GitHub Actions Secrets

Configure no repositório (Settings → Secrets → Actions):

| Secret | Descrição |
|--------|-----------|
| `AZURE_CLIENT_ID` | Client ID do Service Principal (OIDC) |
| `AZURE_TENANT_ID` | Tenant ID |
| `AZURE_SUBSCRIPTION_ID` | Subscription ID |
| `AZURE_CREDENTIALS` | JSON completo do SP (para actions antigas) |

> **Preferência:** Use **OIDC** com `azure/login` — não armazene `AZURE_CREDENTIALS` se possível.

---

## 🌍 Ambientes

| Ambiente | Finalidade | Custo |
|----------|------------|-------|
| **HML** | Homologação, testes, validação | Mínimo (Free Tier / Dev SKUs) |
| **PRO** | Produção | Dimensionado para carga real |

> Em HML, sempre priorize a alternativa mais barata ou gratuita elegível (conforme `CLAUDE.md` do projeto).

---

## 📋 Pipelines

| Workflow | Trigger | Descrição |
|----------|---------|-----------|
| `terraform-plan.yml` | PR para `main` | `terraform plan` nos ambientes alterados |
| `terraform-apply-hml.yml` | Merge em `main` | Apply automático em HML |
| `terraform-apply-pro.yml` | Manual (`workflow_dispatch`) | Apply em PRO **com aprovação** |

---

## 🛡️ Segurança

- **Zero secrets no repositório** — use GitHub Secrets + OIDC
- **Managed Identity** para recursos Azure (ACA → ACR, Key Vault, etc.)
- **RBAC least privilege** — roles customizadas por módulo
- **Private endpoints** para PaaS (ACR, Log Analytics, Key Vault)
- **Policy as Code** — Azure Policy para compliance (opcional)

---

## 📖 Documentação Adicional

- [Decisões Arquiteturais (ADR)](docs/adr/) — Registros de decisões relevantes
- [Runbooks](docs/runbooks/) — Procedimentos operacionais
- [Troubleshooting](docs/troubleshooting.md) — Problemas conhecidos

---

## 🤝 Contribuindo

1. Crie branch: `git checkout -b feat/nome-da-feature`
2. Faça alterações seguindo padrões do projeto
3. Rode `terraform fmt` e `terraform validate`
4. Abra PR com descrição clara
5. Aguarde revisão + plan no CI
6. Merge após aprovação

---

## 📄 Licença

Uso interno — não redistribuir.