# Primeiros Passos no Kubernetes com kubectl

O **kubectl** é a ferramenta de linha de comando para interagir com clusters Kubernetes. Este guia apresenta os comandos essenciais para começar.

## Verificando o Cluster

### Informações do Cluster
```bash
kubectl cluster-info
kubectl get nodes
kubectl get namespaces
```

### Status dos Componentes
```bash
kubectl get componentstatuses
kubectl get all --all-namespaces
```

## Comandos Básicos do kubectl

### Estrutura do Comando
```
kubectl [comando] [tipo] [nome] [flags]
```

### Principais Comandos

#### GET - Listar Recursos
```bash
kubectl get pods
kubectl get services
kubectl get deployments
kubectl get nodes -o wide
```

#### DESCRIBE - Detalhes do Recurso
```bash
kubectl describe node <nome-do-node>
kubectl describe pod <nome-do-pod>
```

#### CREATE - Criar Recursos
```bash
kubectl create deployment nginx --image=nginx
kubectl create service clusterip my-service --tcp=80:80
```

#### APPLY - Aplicar Configurações
```bash
kubectl apply -f arquivo.yaml
kubectl apply -f diretorio/
```

#### DELETE - Remover Recursos
```bash
kubectl delete pod <nome-do-pod>
kubectl delete deployment <nome-do-deployment>
```

## Trabalhando com Pods

### Criar um Pod Simples
```bash
kubectl run nginx --image=nginx --port=80
```

### Verificar Pods
```bash
kubectl get pods
kubectl get pods -o wide
kubectl describe pod nginx
```

### Logs do Pod
```bash
kubectl logs nginx
kubectl logs -f nginx  # follow logs
```

### Executar Comandos no Pod
```bash
kubectl exec nginx -- ls /
kubectl exec -it nginx -- /bin/bash
```

## Arquitetura do kubectl

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        KUBECTL WORKFLOW                                │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────┐    HTTP/HTTPS     ┌─────────────────────────────────┐  │
│  │   kubectl   │ ──────────────────▶│        API Server               │  │
│  │  (client)   │                   │     (kube-apiserver)            │  │
│  └─────────────┘                   └─────────────────────────────────┘  │
│                                                    │                    │
│                                                    ▼                    │
│                                     ┌─────────────────────────────────┐  │
│                                     │         etcd                    │  │
│                                     │    (cluster store)              │  │
│                                     └─────────────────────────────────┘  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## Contextos e Configuração

### Verificar Contexto Atual
```bash
kubectl config current-context
kubectl config get-contexts
```

### Trocar Contexto
```bash
kubectl config use-context <nome-do-contexto>
```

### Configurar Namespace Padrão
```bash
kubectl config set-context --current --namespace=<namespace>
```

## Namespaces

### Listar Namespaces
```bash
kubectl get namespaces
```

### Criar Namespace
```bash
kubectl create namespace desenvolvimento
```

### Trabalhar com Namespace Específico
```bash
kubectl get pods -n kube-system
kubectl get all -n desenvolvimento
```

## Outputs Úteis

### Formatos de Saída
```bash
kubectl get pods -o yaml
kubectl get pods -o json
kubectl get pods -o wide
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase
```

### Filtros e Seletores
```bash
kubectl get pods --selector app=nginx
kubectl get pods --field-selector status.phase=Running
```

## Comandos de Troubleshooting

### Debug de Pods
```bash
kubectl describe pod <nome-do-pod>
kubectl logs <nome-do-pod>
kubectl get events --sort-by=.metadata.creationTimestamp
```

### Port Forward
```bash
kubectl port-forward pod/nginx 8080:80
kubectl port-forward service/nginx 8080:80
```

### Proxy para API
```bash
kubectl proxy --port=8080
```

## Fluxo de Trabalho Típico

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    KUBERNETES WORKFLOW                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  1. CREATE/APPLY     2. SCHEDULE        3. DEPLOY         4. MONITOR    │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐   ┌──────────┐   │
│  │   kubectl   │───▶│  Scheduler  │───▶│   kubelet   │──▶│   Pod    │   │
│  │   apply     │    │             │    │             │   │ Running  │   │
│  └─────────────┘    └─────────────┘    └─────────────┘   └──────────┘   │
│                                                                         │
│  5. VERIFY                                                              │
│  ┌─────────────┐                                                        │
│  │   kubectl   │                                                        │
│  │   get/logs  │                                                        │
│  └─────────────┘                                                        │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## Exemplo Prático

### 1. Criar um Deployment
```bash
kubectl create deployment hello-world --image=nginx:latest
```

### 2. Verificar o Deployment
```bash
kubectl get deployments
kubectl get pods
```

### 3. Expor o Serviço
```bash
kubectl expose deployment hello-world --port=80 --type=NodePort
```

### 4. Verificar o Serviço
```bash
kubectl get services
kubectl describe service hello-world
```

### 5. Testar a Aplicação
```bash
kubectl port-forward service/hello-world 8080:80
# Acesse http://localhost:8080
```

### 6. Limpar Recursos
```bash
kubectl delete service hello-world
kubectl delete deployment hello-world
```

## Dicas Importantes

• Use `kubectl explain` para documentação: `kubectl explain pod.spec`
• Sempre verifique o contexto antes de executar comandos
• Use `--dry-run=client -o yaml` para gerar YAML sem aplicar
• Mantenha backups das configurações importantes
• Use labels e annotations para organizar recursos

## Próximos Passos

- Aprender sobre Deployments e ReplicaSets
- Trabalhar com Services e Ingress
- Configurar ConfigMaps e Secrets
- Implementar Health Checks
- Explorar Persistent Volumes
