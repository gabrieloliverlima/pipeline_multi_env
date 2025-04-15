# APLICAÇÃO KUBENEWS COM K8S

Este repositório contém os manifestos do Kubernetes e o Dockerfile para implantação do aplicativo KubeNews e seu banco de dados PostgreSQL.

## Índice

- [Visão Geral da Arquitetura](#visão-geral-da-arquitetura)
- [Objetivo do Projeto](#objetivo-do-projeto)
- [Pré-requisitos](#pré-requisitos)
- [Instruções de Configuração](#instruções-de-configuração)
- [Manifestos do Kubernetes](#manifestos-do-kubernetes)
  - [Implantação e Serviço do PostgreSQL](#implantação-e-serviço-do-postgresql)
  - [Implantação e Serviço do KubeNews](#implantação-e-serviço-do-kubenews)
- [Dockerfile](#dockerfile)
- [Variáveis de Ambiente](#variáveis-de-ambiente)
- [Acessando o Aplicativo](#acessando-o-aplicativo)

## Visão Geral da Arquitetura

O aplicativo KubeNews é construído em um backend Node.js e usa PostgreSQL como seu banco de dados. A arquitetura consiste em:

- Um banco de dados PostgreSQL implantado em um cluster Kubernetes.
- Um aplicativo Node.js containerizado e implantado no mesmo cluster.

## Objetivo do Projeto

O projeto KubeNews é uma aplicação escrita em NodeJS e tem como objetivo ser uma aplicação de exemplo para trabalhar com o uso de containers.

O Secret foi criado via linha de comando para manter boas práticas:
```bash
kubectl create secret generic kubenews-secret \
  --from-literal=DB_DATABASE=kubenews \
  --from-literal=DB_USERNAME=kubenews \
  --from-literal=DB_PASSWORD=Pg123 \
  --from-literal=DB_HOST=postgresql \
  --from-literal=POSTGRES_DB=kubenews \
  --from-literal=POSTGRES_PASSWORD=Pg123 \
  --from-literal=POSTGRES_USER=kubenews
```

## Pré-requisitos

- Configuração de um cluster Kubernetes (ex.: Minikube, AKS, EKS ou GKE).
- CLI `kubectl` instalado e configurado.
- Docker instalado para construir a imagem do aplicativo.

## Instruções de Configuração

1. **Clone o repositório:**
   ```bash
   git clone <url-do-repositorio>
   cd <diretorio-do-repositorio>
   ```

2. **Construa a imagem Docker:**
   ```bash
   docker build -t <seu-usuario-dockerhub>/k8s-kube-news:v1 .
   ```

3. **Envie a imagem para o Docker Hub:**
   ```bash
   docker push <seu-usuario-dockerhub>/k8s-kube-news:v1
   ```

4. **Aplique os manifestos do Kubernetes:**
   ```bash
   kubectl apply -f deployment.yaml
   ```

5. **Verifique as implantações:**
   ```bash
   kubectl get pods
   ```

## Manifestos do Kubernetes

### Implantação e Serviço do PostgreSQL

O banco de dados PostgreSQL é implantado com as seguintes características:

- Imagem: `postgres:14.15-alpine3.21`
- Recursos:
  - Limite de Memória: 128Mi
  - Limite de CPU: 500m
- O serviço expõe o banco de dados na porta `5432`.

### Implantação e Serviço do KubeNews

O aplicativo Node.js é implantado com:

- Imagem: `gabrieloliver001/k8s-kube-news:v1`
- Recursos:
  - Limite de Memória: 128Mi
  - Limite de CPU: 500m
- O serviço expõe o aplicativo na porta `32000` (NodePort).

## Dockerfile

O Dockerfile do aplicativo:

```dockerfile
FROM node:22.12.0-alpine3.20
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 8080
CMD ["node", "server.js"]
```

## Acessando o Aplicativo

Uma vez implantado, o aplicativo pode ser acessado através do serviço NodePort:

1. **Encontre o NodePort:**
   ```bash
   kubectl get svc app-kubenews
   ```

2. **Acesse o aplicativo:**
   Abra seu navegador e navegue para `http://<ip-do-node>:32000`.

Substitua `<ip-do-node>` pelo IP do seu nó Kubernetes.

---

Fique à vontade para relatar problemas ou contribuir com este repositório!

