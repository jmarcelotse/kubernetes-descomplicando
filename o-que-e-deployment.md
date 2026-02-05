# O que é um Deployment no Kubernetes?

Um **Deployment** é um objeto do Kubernetes que gerencia um conjunto de pods idênticos, garantindo que um número específico de réplicas esteja sempre em execução. É uma das principais formas de executar aplicações no Kubernetes.

## 🎯 Principais Características

### Gerenciamento de ReplicaSets
- O Deployment cria e gerencia ReplicaSets automaticamente
- Cada atualização gera um novo ReplicaSet
- Mantém histórico de versões para rollback

### Declarativo
- Define o estado desejado da aplicação
- O Kubernetes garante que o estado atual corresponda ao desejado
- Auto-recuperação em caso de falhas

### Atualizações Controladas
- Rolling updates (atualizações graduais)
- Rollback para versões anteriores
- Controle de estratégias de deployment

## 🏗️ Arquitetura do Deployment

```
┌─────────────────────────────────────────────────────────────┐
│                        DEPLOYMENT                           │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                   REPLICASET                        │   │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐            │   │
│  │  │  POD 1  │  │  POD 2  │  │  POD 3  │            │   │
│  │  │         │  │         │  │         │            │   │
│  │  │ nginx:1 │  │ nginx:1 │  │ nginx:1 │            │   │
│  │  └─────────┘  └─────────┘  └─────────┘            │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

## 📋 Estrutura YAML Básica

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
```

## 🔄 Ciclo de Vida do Deployment

### 1. Criação
```bash
kubectl apply -f deployment.yaml
```

### 2. Monitoramento
```bash
kubectl get deployments
kubectl describe deployment nginx-deployment
kubectl get rs  # ReplicaSets
kubectl get pods
```

### 3. Atualização
```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.22
```

### 4. Rollback
```bash
kubectl rollout undo deployment/nginx-deployment
```

## 🚀 Estratégias de Deployment

### Rolling Update (Padrão)
- Atualiza pods gradualmente
- Zero downtime
- Controla quantos pods podem estar indisponíveis

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
```

### Recreate
- Para todos os pods antigos antes de criar novos
- Causa downtime temporário
- Útil quando não é possível ter versões simultâneas

```yaml
spec:
  strategy:
    type: Recreate
```

## 📊 Diagrama de Rolling Update

```
Estado Inicial (v1):
┌─────────┐  ┌─────────┐  ┌─────────┐
│ Pod v1  │  │ Pod v1  │  │ Pod v1  │
└─────────┘  └─────────┘  └─────────┘

Passo 1 - Criar novo pod:
┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐
│ Pod v1  │  │ Pod v1  │  │ Pod v1  │  │ Pod v2  │
└─────────┘  └─────────┘  └─────────┘  └─────────┘

Passo 2 - Remover pod antigo:
┌─────────┐  ┌─────────┐  ┌─────────┐
│ Pod v1  │  │ Pod v1  │  │ Pod v2  │
└─────────┘  └─────────┘  └─────────┘

Passo 3 - Continuar até completar:
┌─────────┐  ┌─────────┐  ┌─────────┐
│ Pod v2  │  │ Pod v2  │  │ Pod v2  │
└─────────┘  └─────────┘  └─────────┘
```

## 🎛️ Comandos Essenciais

### Criação e Gerenciamento
```bash
# Criar deployment
kubectl create deployment nginx --image=nginx:1.21 --replicas=3

# Aplicar arquivo YAML
kubectl apply -f deployment.yaml

# Listar deployments
kubectl get deployments

# Detalhes do deployment
kubectl describe deployment nginx-deployment
```

### Escalonamento
```bash
# Escalar manualmente
kubectl scale deployment nginx-deployment --replicas=5

# Auto-escalonamento
kubectl autoscale deployment nginx-deployment --min=2 --max=10 --cpu-percent=80
```

### Atualizações
```bash
# Atualizar imagem
kubectl set image deployment/nginx-deployment nginx=nginx:1.22

# Ver status da atualização
kubectl rollout status deployment/nginx-deployment

# Histórico de rollouts
kubectl rollout history deployment/nginx-deployment

# Rollback para versão anterior
kubectl rollout undo deployment/nginx-deployment

# Rollback para versão específica
kubectl rollout undo deployment/nginx-deployment --to-revision=2
```

## 🔍 Monitoramento e Debug

### Status e Logs
```bash
# Status dos pods
kubectl get pods -l app=nginx

# Logs de um pod específico
kubectl logs deployment/nginx-deployment

# Logs de todos os pods do deployment
kubectl logs -f deployment/nginx-deployment --all-containers=true

# Eventos relacionados
kubectl get events --field-selector involvedObject.name=nginx-deployment
```

### Troubleshooting
```bash
# Verificar ReplicaSets
kubectl get rs

# Descrever ReplicaSet problemático
kubectl describe rs <replicaset-name>

# Verificar recursos
kubectl top pods -l app=nginx
```

## ⚙️ Configurações Avançadas

### Health Checks
```yaml
spec:
  template:
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

### Recursos e Limites
```yaml
spec:
  template:
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
```

## 🆚 Deployment vs Outros Objetos

| Objeto | Uso | Características |
|--------|-----|----------------|
| **Deployment** | Aplicações stateless | Rolling updates, rollback, escalonamento |
| **StatefulSet** | Aplicações stateful | Ordem de criação, identidade persistente |
| **DaemonSet** | Um pod por nó | Logs, monitoramento, storage |
| **Job** | Tarefas batch | Execução única, não reinicia |
| **CronJob** | Tarefas agendadas | Execução periódica |

## 🎯 Boas Práticas

### Labels e Seletores
- Use labels consistentes e descritivas
- Evite alterar seletores após criação
- Use labels para organização e filtragem

### Recursos
- Sempre defina requests e limits
- Monitore uso de recursos
- Use HPA para escalonamento automático

### Atualizações
- Teste em ambiente de desenvolvimento
- Use rolling updates para zero downtime
- Mantenha histórico de rollouts
- Configure health checks adequados

### Segurança
- Use imagens específicas (evite :latest)
- Configure security contexts
- Use secrets para dados sensíveis
- Implemente network policies

## 🔗 Próximos Passos

Após entender Deployments, explore:
- [Services](o-que-e-service.md) - Exposição de aplicações
- [ConfigMaps](o-que-e-configmap.md) - Configurações
- [Secrets](o-que-e-secret.md) - Dados sensíveis
- [Ingress](o-que-e-ingress.md) - Roteamento HTTP
- [HPA](horizontal-pod-autoscaler.md) - Auto-escalonamento

## 📚 Referências

- [Documentação Oficial - Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Rolling Updates](https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/)
- [Kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
