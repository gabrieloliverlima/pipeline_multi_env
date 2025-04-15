# 🚀 Automação de Deploy com GitHub Actions: CI/CD Multiambiente no Kubernetes

Este projeto realiza o **provisionamento de um cluster Kubernetes na DigitalOcean com Terraform** e o **deploy automatizado** de uma aplicação Node.js + PostgreSQL usando **GitHub Actions**, com ambientes separados para **Homologação** e **Produção**, organizados por **namespaces Kubernetes**.

---

## 📦 Tecnologias Utilizadas

- [Terraform](https://www.terraform.io/)
- [DigitalOcean Kubernetes](https://www.digitalocean.com/products/kubernetes)
- [Kubernetes](https://kubernetes.io/)
- [GitHub Actions](https://github.com/features/actions)
- [Docker Hub](https://hub.docker.com/)

---

## ⚙️ Arquitetura CI/CD

A pipeline é composta por dois workflows:

### ✅ CI - Build e Push da Imagem

O workflow principal realiza:

- Checkout do código
- Login no Docker Hub
- Build da imagem da aplicação Node.js (`/src`)
- Push da imagem com tag `v1.<RUN_NUMBER>` e `latest`

### 🚀 CD - Deploy com Reaproveitamento de Código

Após o CI, o workflow chama um **workflow reutilizável** (`deploy.yml`) que:

- Seta o contexto do cluster
- Garante que o namespace (`homolog` ou `producao`) exista
- Aplica os manifestos YAML no namespace correspondente
- Substitui dinamicamente a imagem da aplicação via input `images`

Isso garante que o mesmo código de deploy seja reaproveitado para múltiplos ambientes com apenas **1 definição de pipeline**.

---

## 🧪 Ambientes Separados por Namespace

A separação de ambientes é feita via **namespaces Kubernetes**, garantindo isolamento de recursos:

- `homolog` → ambiente de testes e validações
- `producao` → ambiente real de produção

O namespace é passado dinamicamente via GitHub Actions com a variável `APP_NAMESPACE`.

> Exemplo no deploy:
```yaml
with:
  manifests: k8s/deployment.yaml
  images: gabrieloliver001/k8s-kube-news:v1.${{ github.run_number }}
  environment: homolog
```

---

## 🌐 Estrutura do Projeto

```
.
├── .github/
│   └── workflows/
│       ├── main.yml             # CI build e Push da Image
│       └── deploy.yml            # Workflow reutilizável de deploy
├── terraform_k8s/
│   ├── main.tf                   # Provisionamento do cluster
│   ├── variables.tf
│   └── terraform.tfvars
├── k8s/
│   └── deployment.yaml           # Manifests Kubernetes (PostgreSQL + App)
├── src/                          # Código-fonte da aplicação Node.js
├── Dockerfile
└── README.md
```

---

## 🚀 Passo a passo

### 1️⃣ Provisionar o cluster com Terraform

```bash
cd terraform
terraform init
terraform apply
```

Isso cria o cluster Kubernetes na DigitalOcean.

---

### 2️⃣ Configurar `kubectl`

```bash
terraform output kubeconfig > ~/.kube/config
export KUBECONFIG=~/.kube/config
```

---

### 3️⃣ Criar Secrets Kubernetes (via GitHub Actions ou manual)

Crie um secret contendo as variáveis de ambiente do PostgreSQL:

```bash
kubectl create namespace homolog
kubectl create namespace producao

kubectl create secret generic kubenews-secret \
  --from-literal=POSTGRES_DB=kubenews \
  --from-literal=POSTGRES_USER=admin \
  --from-literal=POSTGRES_PASSWORD=admin123 \
  -n homolog

kubectl create secret generic kubenews-secret \
  --from-literal=POSTGRES_DB=kubenews \
  --from-literal=POSTGRES_USER=admin \
  --from-literal=POSTGRES_PASSWORD=admin123 \
  -n producao
```

---

### 4️⃣ Configurar Variáveis e Segredos no GitHub

**Secrets (Settings → Secrets and Variables → Actions):**

- `DOCKERHUB_USERNAME`
- `DOCKERHUB_TOKEN`
- `KUBECONFIG_MYCLUSTER` → conteúdo do kubeconfig (salvar como secret multi-line)

**Environments (`homolog` e `producao`) → Adicionar variável:**

- `APP_NAMESPACE` = `homolog` ou `producao` (para cada ambiente)

---

### 5️⃣ Push para branch `main` → CI/CD automático!

Ao fazer push para a branch `main`, o GitHub Actions:

1. Constrói a imagem e envia para o Docker Hub
2. Faz o deploy da aplicação em **homolog**
3. Em seguida, faz o deploy da aplicação em **produção**

---

## ✅ Verificando o Deploy

Acompanhe os serviços:

```bash
kubectl get svc -n homolog
kubectl get svc -n producao
```

Acesse o IP externo gerado pelo `LoadBalancer` de cada service para acessar a aplicação.
