# Entendendo e Instalando o kubectl

O **kubectl** é a ferramenta de linha de comando para interagir com clusters Kubernetes. É o principal meio de comunicação entre você e o cluster.

## O que é o kubectl?

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        KUBECTL OVERVIEW                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────┐                                                        │
│  │    User     │                                                        │
│  │             │                                                        │
│  └─────────────┘                                                        │
│         │                                                               │
│         │ Commands                                                      │
│         ▼                                                               │
│  ┌─────────────┐                                                        │
│  │   kubectl   │                                                        │
│  │             │                                                        │
│  │ • CLI Tool  │                                                        │
│  │ • REST API  │                                                        │
│  │ • Config    │                                                        │
│  └─────────────┘                                                        │
│         │                                                               │
│         │ HTTPS/6443                                                    │
│         ▼                                                               │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                    KUBERNETES CLUSTER                              ││
│  │                                                                     ││
│  │  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────────┐ ││
│  │  │ API Server  │    │    etcd     │    │    Worker Nodes         │ ││
│  │  │   :6443     │    │             │    │                         │ ││
│  │  └─────────────┘    └─────────────┘    └─────────────────────────┘ ││
│  └─────────────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────────────┘
```

### Características Principais
- **Interface CLI**: Linha de comando para Kubernetes
- **REST Client**: Comunica via API REST com o cluster
- **Multiplataforma**: Linux, macOS, Windows
- **Configurável**: Múltiplos clusters e contextos
- **Extensível**: Plugins e customizações

## Arquitetura do kubectl

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      KUBECTL ARCHITECTURE                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                      LOCAL MACHINE                                 ││
│  │                                                                     ││
│  │  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────────┐ ││
│  │  │   kubectl   │    │ kubeconfig  │    │      Plugins            │ ││
│  │  │             │    │             │    │                         │ ││
│  │  │ • Commands  │───▶│ • Clusters  │    │ • helm                  │ ││
│  │  │ • Flags     │    │ • Users     │    │ • kustomize             │ ││
│  │  │ • Output    │    │ • Contexts  │    │ • Custom tools          │ ││
│  │  └─────────────┘    └─────────────┘    └─────────────────────────┘ ││
│  └─────────────────────────────────────────────────────────────────────┘│
│                                │                                        │
│                                │ Authentication                         │
│                                │ Authorization                          │
│                                │ HTTPS Request                          │
│                                ▼                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                    KUBERNETES API SERVER                           ││
│  │                                                                     ││
│  │  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────────┐ ││
│  │  │Authentication│   │Authorization│    │      Admission          │ ││
│  │  │             │    │             │    │      Controllers        │ ││
│  │  │ • Certs     │───▶│ • RBAC      │───▶│                         │ ││
│  │  │ • Tokens    │    │ • Policies  │    │ • Validation            │ ││
│  │  │ • OIDC      │    │ • Rules     │    │ • Mutation              │ ││
│  │  └─────────────┘    └─────────────┘    └─────────────────────────┘ ││
│  └─────────────────────────────────────────────────────────────────────┘│
│                                │                                        │
│                                ▼                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                         etcd                                       ││
│  │                   (Cluster State)                                  ││
│  └─────────────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────────────┘
```

## Instalação do kubectl

### Linux

#### Método 1: Download Direto
```bash
# Download da versão mais recente
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Tornar executável
chmod +x kubectl

# Mover para PATH
sudo mv kubectl /usr/local/bin/

# Verificar instalação
kubectl version --client
```

#### Método 2: Gerenciador de Pacotes
```bash
# Ubuntu/Debian
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl

# CentOS/RHEL/Fedora
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
sudo yum install -y kubectl
```

### macOS
```bash
# Homebrew
brew install kubectl

# Download direto
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

### Windows
```powershell
# Chocolatey
choco install kubernetes-cli

# Download direto
curl -LO "https://dl.k8s.io/release/v1.28.0/bin/windows/amd64/kubectl.exe"
```

## Configuração do kubectl

### Estrutura do kubeconfig

```
┌─────────────────────────────────────────────────────────────────────────┐
│                       KUBECONFIG STRUCTURE                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ~/.kube/config                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                      KUBECONFIG FILE                               ││
│  │                                                                     ││
│  │  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────────┐ ││
│  │  │  Clusters   │    │    Users    │    │       Contexts          │ ││
│  │  │             │    │             │    │                         │ ││
│  │  │ • dev       │    │ • dev-user  │    │ • dev-context           │ ││
│  │  │ • staging   │    │ • admin     │    │ • staging-context       │ ││
│  │  │ • prod      │    │ • service   │    │ • prod-context          │ ││
│  │  │             │    │   account   │    │                         │ ││
│  │  └─────────────┘    └─────────────┘    └─────────────────────────┘ ││
│  │         │                   │                         │             ││
│  │         └───────────────────┼─────────────────────────┘             ││
│  │                             ▼                                       ││
│  │                    ┌─────────────┐                                  ││
│  │                    │   Context   │                                  ││
│  │                    │             │                                  ││
│  │                    │ cluster +   │                                  ││
│  │                    │ user +      │                                  ││
│  │                    │ namespace   │                                  ││
│  │                    └─────────────┘                                  ││
│  └─────────────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────────────┘
```

### Exemplo de kubeconfig
```yaml
apiVersion: v1
kind: Config
current-context: dev-context

clusters:
- name: dev-cluster
  cluster:
    server: https://dev-k8s.example.com:6443
    certificate-authority-data: LS0tLS1CRUdJTi...

users:
- name: dev-user
  user:
    client-certificate-data: LS0tLS1CRUdJTi...
    client-key-data: LS0tLS1CRUdJTi...

contexts:
- name: dev-context
  context:
    cluster: dev-cluster
    user: dev-user
    namespace: development
```

## Comandos Básicos do kubectl

### Sintaxe Geral
```
kubectl [command] [TYPE] [NAME] [flags]
```

### Diagrama de Comandos

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      KUBECTL COMMANDS                                  │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                    BASIC OPERATIONS                                ││
│  │                                                                     ││
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────────┐ ││
│  │  │    GET      │  │   DESCRIBE  │  │         CREATE              │ ││
│  │  │             │  │             │  │                             │ ││
│  │  │ • List      │  │ • Details   │  │ • From file                 │ ││
│  │  │ • Resources │  │ • Events    │  │ • Imperative                │ ││
│  │  │ • Status    │  │ • Config    │  │ • Generate                  │ ││
│  │  └─────────────┘  └─────────────┘  └─────────────────────────────┘ ││
│  └─────────────────────────────────────────────────────────────────────┘│
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                   MANAGEMENT OPERATIONS                            ││
│  │                                                                     ││
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────────┐ ││
│  │  │   APPLY     │  │   DELETE    │  │         EDIT                │ ││
│  │  │             │  │             │  │                             │ ││
│  │  │ • Declarative│ │ • Remove    │  │ • Modify                    │ ││
│  │  │ • Update    │  │ • Resources │  │ • In-place                  │ ││
│  │  │ • Create    │  │ • Force     │  │ • Validation                │ ││
│  │  └─────────────┘  └─────────────┘  └─────────────────────────────┘ ││
│  └─────────────────────────────────────────────────────────────────────┘│
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                   TROUBLESHOOTING                                  ││
│  │                                                                     ││
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────────┐ ││
│  │  │    LOGS     │  │    EXEC     │  │      PORT-FORWARD           │ ││
│  │  │             │  │             │  │                             │ ││
│  │  │ • Container │  │ • Shell     │  │ • Local access              │ ││
│  │  │ • Follow    │  │ • Commands  │  │ • Debug                     │ ││
│  │  │ • Previous  │  │ • Debug     │  │ • Testing                   │ ││
│  │  └─────────────┘  └─────────────┘  └─────────────────────────────┘ ││
│  └─────────────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────────────┘
```
### Comandos Essenciais

#### Informações do Cluster
```bash
# Versão do kubectl e cluster
kubectl version

# Informações do cluster
kubectl cluster-info

# Nós do cluster
kubectl get nodes

# Recursos disponíveis
kubectl api-resources
```

#### Gerenciamento de Recursos
```bash
# Listar recursos
kubectl get pods
kubectl get deployments
kubectl get services
kubectl get all

# Detalhes de um recurso
kubectl describe pod <pod-name>
kubectl describe deployment <deployment-name>

# Criar recursos
kubectl create deployment nginx --image=nginx
kubectl apply -f deployment.yaml

# Editar recursos
kubectl edit deployment nginx
kubectl patch deployment nginx -p '{"spec":{"replicas":3}}'

# Deletar recursos
kubectl delete pod <pod-name>
kubectl delete deployment nginx
kubectl delete -f deployment.yaml
```

#### Troubleshooting
```bash
# Ver logs
kubectl logs <pod-name>
kubectl logs -f <pod-name>  # follow
kubectl logs <pod-name> -c <container-name>  # container específico

# Executar comandos no pod
kubectl exec -it <pod-name> -- /bin/bash
kubectl exec <pod-name> -- ls -la

# Port forwarding
kubectl port-forward pod/<pod-name> 8080:80
kubectl port-forward service/<service-name> 8080:80

# Copiar arquivos
kubectl cp <pod-name>:/path/to/file ./local-file
kubectl cp ./local-file <pod-name>:/path/to/file
```

## Gerenciamento de Contextos

### Diagrama de Contextos

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      CONTEXT MANAGEMENT                                │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                    MULTIPLE CLUSTERS                               ││
│  │                                                                     ││
│  │  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────────┐ ││
│  │  │Development  │    │   Staging   │    │      Production         │ ││
│  │  │  Cluster    │    │   Cluster   │    │       Cluster           │ ││
│  │  │             │    │             │    │                         │ ││
│  │  │ dev-k8s.com │    │stg-k8s.com  │    │    prod-k8s.com         │ ││
│  │  │   :6443     │    │   :6443     │    │       :6443             │ ││
│  │  └─────────────┘    └─────────────┘    └─────────────────────────┘ ││
│  │         ▲                   ▲                         ▲             ││
│  │         │                   │                         │             ││
│  │         │                   │                         │             ││
│  └─────────┼───────────────────┼─────────────────────────┼─────────────┘│
│            │                   │                         │              │
│            │                   │                         │              │
│  ┌─────────┼───────────────────┼─────────────────────────┼─────────────┐│
│  │         │                   │                         │             ││
│  │  ┌──────▼──────┐    ┌───────▼─────┐    ┌──────────────▼──────────┐ ││
│  │  │dev-context  │    │stg-context  │    │     prod-context        │ ││
│  │  │             │    │             │    │                         │ ││
│  │  │cluster: dev │    │cluster: stg │    │   cluster: prod         │ ││
│  │  │user: dev    │    │user: dev    │    │   user: admin           │ ││
│  │  │ns: default  │    │ns: staging  │    │   ns: production        │ ││
│  │  └─────────────┘    └─────────────┘    └─────────────────────────┘ ││
│  │                                                                     ││
│  │                    Current Context: dev-context                     ││
│  └─────────────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────────────┘
```

### Comandos de Contexto
```bash
# Listar contextos
kubectl config get-contexts

# Contexto atual
kubectl config current-context

# Mudar contexto
kubectl config use-context dev-context

# Criar contexto
kubectl config set-context dev-context \
  --cluster=dev-cluster \
  --user=dev-user \
  --namespace=development

# Definir namespace padrão
kubectl config set-context --current --namespace=kube-system
```

## Fluxo de Comunicação kubectl

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    KUBECTL COMMUNICATION FLOW                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  1. User Command                                                        │
│     │                                                                   │
│     │ kubectl get pods                                                  │
│     ▼                                                                   │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                      KUBECTL CLIENT                                ││
│  │                                                                     ││
│  │  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────────┐ ││
│  │  │   Parse     │───▶│ Validate    │───▶│      Build Request      │ ││
│  │  │  Command    │    │ Arguments   │    │                         │ ││
│  │  └─────────────┘    └─────────────┘    └─────────────────────────┘ ││
│  └─────────────────────────────────────────────────────────────────────┘│
│                                │                                        │
│  2. Authentication & Authorization                                      │
│                                │                                        │
│                                ▼                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                    HTTPS REQUEST                                   ││
│  │                                                                     ││
│  │  GET /api/v1/pods                                                   ││
│  │  Host: k8s-api-server:6443                                          ││
│  │  Authorization: Bearer <token>                                      ││
│  │  Content-Type: application/json                                     ││
│  └─────────────────────────────────────────────────────────────────────┘│
│                                │                                        │
│                                ▼                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                    API SERVER                                      ││
│  │                                                                     ││
│  │  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────────┐ ││
│  │  │Authenticate │───▶│ Authorize   │───▶│     Process Request     │ ││
│  │  │   User      │    │ Operation   │    │                         │ ││
│  │  └─────────────┘    └─────────────┘    └─────────────────────────┘ ││
│  └─────────────────────────────────────────────────────────────────────┘│
│                                │                                        │
│  3. Data Retrieval                                                      │
│                                │                                        │
│                                ▼                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                        etcd                                        ││
│  │                   Query Pod Data                                    ││
│  └─────────────────────────────────────────────────────────────────────┘│
│                                │                                        │
│  4. Response                   │                                        │
│                                ▼                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                    JSON RESPONSE                                   ││
│  │                                                                     ││
│  │  {                                                                  ││
│  │    "items": [                                                       ││
│  │      {                                                              ││
│  │        "metadata": {"name": "nginx-pod"},                           ││
│  │        "status": {"phase": "Running"}                               ││
│  │      }                                                              ││
│  │    ]                                                                ││
│  │  }                                                                  ││
│  └─────────────────────────────────────────────────────────────────────┘│
│                                │                                        │
│  5. Format Output              │                                        │
│                                ▼                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                    KUBECTL OUTPUT                                  ││
│  │                                                                     ││
│  │  NAME       READY   STATUS    RESTARTS   AGE                        ││
│  │  nginx-pod  1/1     Running   0          5m                         ││
│  └─────────────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────────────┘
```

## Configuração Avançada

### Múltiplos kubeconfig
```bash
# Usar arquivo específico
kubectl --kubeconfig=/path/to/config get pods

# Mesclar configs
export KUBECONFIG=~/.kube/config:~/.kube/config-dev:~/.kube/config-prod
kubectl config view --flatten > ~/.kube/merged-config

# Variável de ambiente
export KUBECONFIG=~/.kube/dev-config
```

### Aliases e Shortcuts
```bash
# Adicionar ao ~/.bashrc ou ~/.zshrc
alias k=kubectl
alias kgp='kubectl get pods'
alias kgs='kubectl get services'
alias kgd='kubectl get deployments'
alias kdp='kubectl describe pod'
alias kl='kubectl logs'
alias ke='kubectl exec -it'

# Autocompletion
source <(kubectl completion bash)  # bash
source <(kubectl completion zsh)   # zsh
```

### Plugins do kubectl
```bash
# Instalar krew (plugin manager)
(
  set -x; cd "$(mktemp -d)" &&
  OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
  ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
  KREW="krew-${OS}_${ARCH}" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/${KREW}.tar.gz" &&
  tar zxvf "${KREW}.tar.gz" &&
  ./"${KREW}" install krew
)

# Adicionar ao PATH
export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"

# Instalar plugins úteis
kubectl krew install ctx      # context switcher
kubectl krew install ns       # namespace switcher
kubectl krew install tree     # resource tree view
kubectl krew install top      # resource usage
```

## Troubleshooting Comum

### Problemas de Conectividade
```bash
# Verificar conectividade
kubectl cluster-info

# Debug de conexão
kubectl cluster-info dump

# Verificar certificados
kubectl config view --raw

# Testar API diretamente
curl -k https://<api-server>:6443/api/v1/pods \
  -H "Authorization: Bearer <token>"
```

### Problemas de Autenticação
```bash
# Verificar contexto atual
kubectl config current-context

# Verificar permissões
kubectl auth can-i get pods
kubectl auth can-i create deployments
kubectl auth can-i "*" "*" --all-namespaces

# Debug de RBAC
kubectl describe clusterrolebinding
kubectl describe rolebinding -n <namespace>
```

## Resumo de Comandos Essenciais

| Categoria | Comando | Descrição |
|-----------|---------|-----------|
| **Info** | `kubectl version` | Versão do kubectl/cluster |
| **Info** | `kubectl cluster-info` | Informações do cluster |
| **Resources** | `kubectl get <resource>` | Listar recursos |
| **Resources** | `kubectl describe <resource> <name>` | Detalhes do recurso |
| **Resources** | `kubectl apply -f <file>` | Aplicar configuração |
| **Resources** | `kubectl delete <resource> <name>` | Deletar recurso |
| **Debug** | `kubectl logs <pod>` | Ver logs do pod |
| **Debug** | `kubectl exec -it <pod> -- bash` | Shell no pod |
| **Debug** | `kubectl port-forward <pod> 8080:80` | Port forwarding |
| **Context** | `kubectl config get-contexts` | Listar contextos |
| **Context** | `kubectl config use-context <name>` | Mudar contexto |
