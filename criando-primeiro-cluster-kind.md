# Criando o Nosso Primeiro Cluster com o kind

O **kind** (Kubernetes in Docker) é uma ferramenta para executar clusters Kubernetes locais usando containers Docker como nós.

## O que é o kind?

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           KIND OVERVIEW                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                      HOST MACHINE                                  ││
│  │                                                                     ││
│  │  ┌─────────────┐                                                    ││
│  │  │   Docker    │                                                    ││
│  │  │   Engine    │                                                    ││
│  │  └─────────────┘                                                    ││
│  │         │                                                           ││
│  │         │ Creates containers as nodes                               ││
│  │         ▼                                                           ││
│  │  ┌─────────────────────────────────────────────────────────────────┐││
│  │  │                 KIND CLUSTER                                   │││
│  │  │                                                                 │││
│  │  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐ │││
│  │  │  │Control Plane│  │ Worker Node │  │      Worker Node        │ │││
│  │  │  │ Container   │  │ Container   │  │      Container          │ │││
│  │  │  │             │  │             │  │                         │ │││
│  │  │  │ • API Server│  │ • kubelet   │  │ • kubelet               │ │││
│  │  │  │ • etcd      │  │ • kube-proxy│  │ • kube-proxy            │ │││
│  │  │  │ • Scheduler │  │ • Container │  │ • Container Runtime     │ │││
│  │  │  │ • Controller│  │   Runtime   │  │                         │ │││
│  │  │  └─────────────┘  └─────────────┘  └─────────────────────────┘ │││
│  │  └─────────────────────────────────────────────────────────────────┘││
│  └─────────────────────────────────────────────────────────────────────┘│
│                                                                         │
│  Características:                                                       │
│  • Rápido para criar/destruir                                          │
│  • Ideal para desenvolvimento e testes                                 │
│  • Suporte a múltiplos nós                                             │
│  • Configuração via YAML                                               │
└─────────────────────────────────────────────────────────────────────────┘
```

### Vantagens do kind
- **Rápido**: Criação de cluster em segundos
- **Leve**: Usa containers em vez de VMs
- **Isolado**: Não interfere no sistema host
- **Configurável**: Suporte a configurações customizadas
- **CI/CD**: Ideal para pipelines de teste

## Instalação do kind

### Linux
```bash
# Download da versão mais recente
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

# Verificar instalação
kind version
```

### macOS
```bash
# Homebrew
brew install kind

# Download direto
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-darwin-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

### Windows
```powershell
# Chocolatey
choco install kind

# Download direto
curl.exe -Lo kind-windows-amd64.exe https://kind.sigs.k8s.io/dl/v0.20.0/kind-windows-amd64
Move-Item .\kind-windows-amd64.exe c:\some-dir-in-your-PATH\kind.exe
```

## Pré-requisitos

### Docker
```bash
# Verificar se Docker está rodando
docker version
docker info

# Se não estiver instalado (Ubuntu/Debian)
sudo apt-get update
sudo apt-get install docker.io
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER
```

## Criando o Primeiro Cluster

### Cluster Simples (1 nó)

```bash
# Criar cluster com configuração padrão
kind create cluster

# Verificar cluster
kubectl cluster-info --context kind-kind
kubectl get nodes
```

### Diagrama do Cluster Simples

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      SINGLE NODE CLUSTER                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                    Docker Container                                ││
│  │                   kind-control-plane                                ││
│  │                                                                     ││
│  │  ┌─────────────────────────────────────────────────────────────────┐││
│  │  │                 Control Plane + Worker                         │││
│  │  │                                                                 │││
│  │  │  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────┐ │││
│  │  │  │ API Server  │    │   kubelet   │    │      Pods           │ │││
│  │  │  │   :6443     │    │   :10250    │    │                     │ │││
│  │  │  └─────────────┘    └─────────────┘    └─────────────────────┘ │││
│  │  │                                                                 │││
│  │  │  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────┐ │││
│  │  │  │    etcd     │    │ kube-proxy  │    │   Container         │ │││
│  │  │  │   :2379     │    │   :10256    │    │   Runtime           │ │││
│  │  │  └─────────────┘    └─────────────┘    └─────────────────────┘ │││
│  │  │                                                                 │││
│  │  │  ┌─────────────┐    ┌─────────────┐                            │││
│  │  │  │ Scheduler   │    │Controller   │                            │││
│  │  │  │  :10259     │    │Manager      │                            │││
│  │  │  └─────────────┘    └─────────────┘                            │││
│  │  └─────────────────────────────────────────────────────────────────┘││
│  └─────────────────────────────────────────────────────────────────────┘│
│                                                                         │
│  Acesso via kubectl:                                                    │
│  kubectl --context kind-kind get nodes                                  │
└─────────────────────────────────────────────────────────────────────────┘
```

### Cluster Multi-nó

#### Arquivo de Configuração
```yaml
# kind-config.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
```

#### Criando o Cluster
```bash
# Criar cluster multi-nó
kind create cluster --config kind-config.yaml --name meu-cluster

# Verificar nós
kubectl get nodes
```

### Diagrama do Cluster Multi-nó

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      MULTI-NODE CLUSTER                                │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                    Docker Network                                  ││
│  │                      kind                                           ││
│  │                                                                     ││
│  │  ┌─────────────────┐                                                ││
│  │  │ Control Plane   │                                                ││
│  │  │   Container     │                                                ││
│  │  │                 │                                                ││
│  │  │ ┌─────────────┐ │                                                ││
│  │  │ │ API Server  │ │                                                ││
│  │  │ │   :6443     │ │                                                ││
│  │  │ └─────────────┘ │                                                ││
│  │  │ ┌─────────────┐ │                                                ││
│  │  │ │    etcd     │ │                                                ││
│  │  │ │   :2379     │ │                                                ││
│  │  │ └─────────────┘ │                                                ││
│  │  │ ┌─────────────┐ │                                                ││
│  │  │ │ Scheduler   │ │                                                ││
│  │  │ │Controller   │ │                                                ││
│  │  │ └─────────────┘ │                                                ││
│  │  └─────────────────┘                                                ││
│  │           │                                                         ││
│  │           │ API Calls                                               ││
│  │           ▼                                                         ││
│  │  ┌─────────────────┐              ┌─────────────────────────────────┐││
│  │  │  Worker Node 1  │              │        Worker Node 2            │││
│  │  │   Container     │              │         Container               │││
│  │  │                 │              │                                 │││
│  │  │ ┌─────────────┐ │              │ ┌─────────────────────────────┐ │││
│  │  │ │   kubelet   │ │              │ │          kubelet            │ │││
│  │  │ │   :10250    │ │              │ │          :10250             │ │││
│  │  │ └─────────────┘ │              │ └─────────────────────────────┘ │││
│  │  │ ┌─────────────┐ │              │ ┌─────────────────────────────┐ │││
│  │  │ │ kube-proxy  │ │              │ │        kube-proxy           │ │││
│  │  │ │   :10256    │ │              │ │         :10256              │ │││
│  │  │ └─────────────┘ │              │ └─────────────────────────────┘ │││
│  │  │ ┌─────────────┐ │              │ ┌─────────────────────────────┐ │││
│  │  │ │Container    │ │              │ │      Container              │ │││
│  │  │ │Runtime      │ │              │ │      Runtime                │ │││
│  │  │ └─────────────┘ │              │ └─────────────────────────────┘ │││
│  │  │                 │              │                                 │││
│  │  │    [Pods]       │              │         [Pods]                  │││
│  │  └─────────────────┘              └─────────────────────────────────┘││
│  └─────────────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────────────┘
```

## Configurações Avançadas

### Cluster com Ingress
```yaml
# kind-ingress-config.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
- role: worker
- role: worker
```

### Cluster com Versão Específica
```yaml
# kind-version-config.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  image: kindest/node:v1.27.3@sha256:3966ac761ae0136263ffdb6cfd4db23ef8a83cba8a463690e98317add2c9ba72
- role: worker
  image: kindest/node:v1.27.3@sha256:3966ac761ae0136263ffdb6cfd4db23ef8a83cba8a463690e98317add2c9ba72
```
## Comandos Essenciais do kind

### Gerenciamento de Clusters

```bash
# Criar cluster
kind create cluster                           # cluster padrão
kind create cluster --name meu-cluster        # cluster nomeado
kind create cluster --config config.yaml     # com configuração

# Listar clusters
kind get clusters

# Obter kubeconfig
kind get kubeconfig --name meu-cluster

# Deletar cluster
kind delete cluster                           # cluster padrão
kind delete cluster --name meu-cluster       # cluster específico
```

### Diagrama de Comandos kind

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        KIND COMMANDS                                    │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                    CLUSTER LIFECYCLE                               ││
│  │                                                                     ││
│  │  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────────┐ ││
│  │  │   CREATE    │───▶│   MANAGE    │───▶│        DELETE           │ ││
│  │  │             │    │             │    │                         │ ││
│  │  │ • Config    │    │ • Get info  │    │ • Clean up              │ ││
│  │  │ • Name      │    │ • Load imgs │    │ • Remove containers     │ ││
│  │  │ • Version   │    │ • Export    │    │ • Reset kubeconfig      │ ││
│  │  └─────────────┘    └─────────────┘    └─────────────────────────┘ ││
│  └─────────────────────────────────────────────────────────────────────┘│
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                      WORKFLOW                                      ││
│  │                                                                     ││
│  │  1. kind create cluster                                             ││
│  │     │                                                               ││
│  │     ▼                                                               ││
│  │  2. Docker containers created                                       ││
│  │     │                                                               ││
│  │     ▼                                                               ││
│  │  3. Kubernetes components installed                                 ││
│  │     │                                                               ││
│  │     ▼                                                               ││
│  │  4. kubeconfig updated                                              ││
│  │     │                                                               ││
│  │     ▼                                                               ││
│  │  5. Cluster ready for use                                           ││
│  └─────────────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────────────┘
```

## Testando o Cluster

### Deploy de Aplicação de Teste

```bash
# Criar deployment
kubectl create deployment nginx --image=nginx

# Expor como service
kubectl expose deployment nginx --port=80 --type=NodePort

# Verificar recursos
kubectl get all
```

### Exemplo Completo - Aplicação Web

```yaml
# test-app.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: test-web-app
  template:
    metadata:
      labels:
        app: test-web-app
    spec:
      containers:
      - name: web
        image: nginx:1.20
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: test-web-service
spec:
  selector:
    app: test-web-app
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
  type: NodePort
```

```bash
# Aplicar configuração
kubectl apply -f test-app.yaml

# Verificar pods
kubectl get pods -l app=test-web-app

# Testar acesso
curl http://localhost:30080
```

### Diagrama da Aplicação de Teste

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      TEST APPLICATION                                  │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Host Machine                                                           │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                                                                     ││
│  │  Browser ──────────────────────────────────────────────────────────┐││
│  │     │                                                              │││
│  │     │ http://localhost:30080                                       │││
│  │     ▼                                                              │││
│  │  ┌─────────────────────────────────────────────────────────────────┐│││
│  │  │                 KIND CLUSTER                                   ││││
│  │  │                                                                 ││││
│  │  │  ┌─────────────┐                                                ││││
│  │  │  │   Service   │                                                ││││
│  │  │  │ NodePort    │                                                ││││
│  │  │  │   :30080    │                                                ││││
│  │  │  └─────────────┘                                                ││││
│  │  │         │                                                       ││││
│  │  │         │ Load Balance                                          ││││
│  │  │         ▼                                                       ││││
│  │  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐ ││││
│  │  │  │   Pod 1     │  │   Pod 2     │  │         Pod 3           │ ││││
│  │  │  │             │  │             │  │                         │ ││││
│  │  │  │ nginx:1.20  │  │ nginx:1.20  │  │      nginx:1.20         │ ││││
│  │  │  │   :80       │  │   :80       │  │        :80              │ ││││
│  │  │  └─────────────┘  └─────────────┘  └─────────────────────────┘ ││││
│  │  │                                                                 ││││
│  │  │  Distributed across worker nodes                                ││││
│  │  └─────────────────────────────────────────────────────────────────┘│││
│  └─────────────────────────────────────────────────────────────────────┘││
└─────────────────────────────────────────────────────────────────────────┘│
```

## Carregando Imagens Locais

### Construir e Carregar Imagem

```bash
# Construir imagem local
docker build -t minha-app:latest .

# Carregar no cluster kind
kind load docker-image minha-app:latest --name meu-cluster

# Verificar imagens no cluster
docker exec -it meu-cluster-control-plane crictl images
```

### Exemplo com Dockerfile

```dockerfile
# Dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/
EXPOSE 80
```

```html
<!-- index.html -->
<!DOCTYPE html>
<html>
<head>
    <title>Minha App no kind</title>
</head>
<body>
    <h1>Hello from kind cluster!</h1>
    <p>Esta aplicação está rodando no Kubernetes com kind</p>
</body>
</html>
```

```bash
# Build e deploy
docker build -t minha-app:v1.0 .
kind load docker-image minha-app:v1.0
kubectl create deployment minha-app --image=minha-app:v1.0
kubectl expose deployment minha-app --port=80 --type=NodePort
```

## Configuração de Rede

### Port Mapping

```yaml
# kind-port-config.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 30080
    hostPort: 8080
    protocol: TCP
  - containerPort: 30443
    hostPort: 8443
    protocol: TCP
```

### Diagrama de Port Mapping

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        PORT MAPPING                                     │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Host Machine                                                           │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                                                                     ││
│  │  Application ──────────────────────────────────────────────────────┐││
│  │       │                                                            │││
│  │       │ localhost:8080                                             │││
│  │       ▼                                                            │││
│  │  ┌─────────────┐                                                   │││
│  │  │ Host Port   │                                                   │││
│  │  │   :8080     │                                                   │││
│  │  └─────────────┘                                                   │││
│  │         │                                                          │││
│  │         │ Port Forward                                             │││
│  │         ▼                                                          │││
│  │  ┌─────────────────────────────────────────────────────────────────┐│││
│  │  │                 Docker Container                               ││││
│  │  │              kind-control-plane                                ││││
│  │  │                                                                 ││││
│  │  │  ┌─────────────┐                                                ││││
│  │  │  │Container    │                                                ││││
│  │  │  │Port :30080  │                                                ││││
│  │  │  └─────────────┘                                                ││││
│  │  │         │                                                       ││││
│  │  │         │ NodePort Service                                      ││││
│  │  │         ▼                                                       ││││
│  │  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐ ││││
│  │  │  │   Pod 1     │  │   Pod 2     │  │         Pod 3           │ ││││
│  │  │  │    :80      │  │    :80      │  │          :80            │ ││││
│  │  │  └─────────────┘  └─────────────┘  └─────────────────────────┘ ││││
│  │  └─────────────────────────────────────────────────────────────────┘│││
│  └─────────────────────────────────────────────────────────────────────┘││
└─────────────────────────────────────────────────────────────────────────┘│
```

## Troubleshooting

### Problemas Comuns

```bash
# Verificar status do Docker
docker info

# Verificar containers do kind
docker ps | grep kind

# Logs do cluster
kind export logs --name meu-cluster

# Verificar configuração do kubectl
kubectl config current-context
kubectl config get-contexts

# Debug de nós
kubectl describe nodes
kubectl get nodes -o wide
```

### Limpeza e Reset

```bash
# Deletar todos os clusters
kind get clusters | xargs -I {} kind delete cluster --name {}

# Limpar imagens Docker do kind
docker images | grep kindest | awk '{print $3}' | xargs docker rmi

# Reset completo do kubectl context
kubectl config unset current-context
```

## Comparação: kind vs outras ferramentas

| Ferramenta | Uso | Vantagens | Desvantagens |
|------------|-----|-----------|--------------|
| **kind** | Dev/Test | Rápido, leve, CI/CD | Apenas local |
| **minikube** | Dev/Test | GUI, addons | Mais pesado |
| **k3s** | Edge/Prod | Leve, produção | Menos features |
| **kubeadm** | Produção | Completo, oficial | Complexo |

## Comandos de Referência Rápida

```bash
# Cluster básico
kind create cluster

# Cluster nomeado
kind create cluster --name dev

# Com configuração
kind create cluster --config cluster.yaml --name staging

# Listar clusters
kind get clusters

# Kubeconfig
kind get kubeconfig --name dev

# Carregar imagem
kind load docker-image nginx:latest --name dev

# Deletar cluster
kind delete cluster --name dev

# Logs
kind export logs --name dev

# Verificar cluster
kubectl cluster-info --context kind-dev
kubectl get nodes
```

## Próximos Passos

Após criar seu cluster kind:

1. **Instalar Ingress Controller**
2. **Configurar monitoramento**
3. **Testar deployments**
4. **Experimentar com Helm**
5. **Configurar CI/CD**

O kind é perfeito para aprender Kubernetes sem a complexidade de um cluster real!
