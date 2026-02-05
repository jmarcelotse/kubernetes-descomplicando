# Criando Múltiplos Deployments e Analisando Detalhes

Guia prático para criar vários Deployments simultaneamente e analisar seus detalhes no Kubernetes.

## 🎯 Objetivos

- Criar múltiplos Deployments diferentes
- Analisar detalhes e configurações
- Comparar comportamentos
- Gerenciar múltiplas aplicações
- Monitorar recursos e status

## 🚀 Criando Múltiplos Deployments

### Deployment 1: Frontend Web

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-web
  labels:
    app: frontend
    tier: web
    component: ui
spec:
  replicas: 4
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
        tier: web
        component: ui
    spec:
      containers:
      - name: nginx
        image: nginx:1.22
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "200m"
```

### Deployment 2: API Backend

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-backend
  labels:
    app: api
    tier: backend
    component: service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
        tier: backend
        component: service
    spec:
      containers:
      - name: api-server
        image: node:16-alpine
        command: ["node", "server.js"]
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          value: "production"
        resources:
          requests:
            memory: "128Mi"
            cpu: "250m"
          limits:
            memory: "256Mi"
            cpu: "500m"
```

### Deployment 3: Database

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: database
  labels:
    app: postgres
    tier: database
    component: storage
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
        tier: database
        component: storage
    spec:
      containers:
      - name: postgres
        image: postgres:13
        ports:
        - containerPort: 5432
        env:
        - name: POSTGRES_DB
          value: "myapp"
        - name: POSTGRES_USER
          value: "admin"
        - name: POSTGRES_PASSWORD
          value: "password123"
        resources:
          requests:
            memory: "256Mi"
            cpu: "500m"
          limits:
            memory: "512Mi"
            cpu: "1000m"
```

## 📊 Comandos para Análise Detalhada

### Listagem Geral

```bash
# Listar todos os deployments
kubectl get deployments

# Com mais detalhes
kubectl get deployments -o wide

# Com labels
kubectl get deployments --show-labels

# Formato customizado
kubectl get deployments -o custom-columns=NAME:.metadata.name,REPLICAS:.spec.replicas,READY:.status.readyReplicas,IMAGE:.spec.template.spec.containers[0].image
```

### Análise Individual

```bash
# Detalhes completos de um deployment
kubectl describe deployment frontend-web

# YAML completo
kubectl get deployment frontend-web -o yaml

# JSON para análise programática
kubectl get deployment frontend-web -o json

# Apenas spec
kubectl get deployment frontend-web -o jsonpath='{.spec}'
```

### Monitoramento de Status

```bash
# Watch em tempo real
kubectl get deployments -w

# Status de rollout
kubectl rollout status deployment/frontend-web

# Histórico de revisões
kubectl rollout history deployment/frontend-web

# Eventos relacionados
kubectl get events --field-selector involvedObject.name=frontend-web
```

## 🔍 Analisando Detalhes Específicos

### ReplicaSets Gerados

```bash
# Listar ReplicaSets
kubectl get rs

# ReplicaSets de um deployment específico
kubectl get rs -l app=frontend

# Detalhes do ReplicaSet
kubectl describe rs <replicaset-name>
```

### Pods Criados

```bash
# Pods por deployment
kubectl get pods -l app=frontend
kubectl get pods -l app=api
kubectl get pods -l app=postgres

# Pods com mais informações
kubectl get pods -o wide -l tier=web

# Status detalhado dos pods
kubectl describe pods -l app=frontend
```

### Recursos Utilizados

```bash
# Uso de recursos por pods
kubectl top pods -l app=frontend

# Uso de recursos por nós
kubectl top nodes

# Recursos solicitados vs limites
kubectl describe nodes | grep -A 5 "Allocated resources"
```

## 📈 Comparando Deployments

### Por Tier/Camada

```bash
# Frontend
kubectl get deployments -l tier=web
kubectl get pods -l tier=web

# Backend
kubectl get deployments -l tier=backend
kubectl get pods -l tier=backend

# Database
kubectl get deployments -l tier=database
kubectl get pods -l tier=database
```

### Por Recursos

```bash
# Deployments ordenados por réplicas
kubectl get deployments --sort-by=.spec.replicas

# Pods ordenados por idade
kubectl get pods --sort-by=.metadata.creationTimestamp

# Pods com mais CPU
kubectl top pods --sort-by=cpu
```

## 🛠️ Exercícios Práticos

### Exercício 1: Criar Múltiplos Deployments

1. Crie os três deployments acima
2. Verifique se todos estão rodando
3. Liste pods de cada tier
4. Compare recursos utilizados

```bash
# Aplicar todos os deployments
kubectl apply -f frontend-deployment.yaml
kubectl apply -f api-deployment.yaml
kubectl apply -f database-deployment.yaml

# Verificar status
kubectl get deployments
kubectl get pods --show-labels
```

### Exercício 2: Análise Detalhada

1. Analise o deployment do frontend:
```bash
kubectl describe deployment frontend-web
kubectl get deployment frontend-web -o yaml
```

2. Compare com o backend:
```bash
kubectl describe deployment api-backend
kubectl get deployment api-backend -o yaml
```

3. Identifique diferenças nas configurações

### Exercício 3: Monitoramento

1. Monitore em tempo real:
```bash
kubectl get deployments -w
```

2. Em outro terminal, escale um deployment:
```bash
kubectl scale deployment frontend-web --replicas=6
```

3. Observe as mudanças no primeiro terminal

## 🔧 Configurações Avançadas

### Health Checks Diferenciados

```yaml
# Frontend com health checks HTTP
livenessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /health
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 5
```

```yaml
# API com health checks customizados
livenessProbe:
  httpGet:
    path: /api/health
    port: 3000
  initialDelaySeconds: 60
  periodSeconds: 30

readinessProbe:
  httpGet:
    path: /api/ready
    port: 3000
  initialDelaySeconds: 10
  periodSeconds: 10
```

```yaml
# Database com TCP probe
livenessProbe:
  tcpSocket:
    port: 5432
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  exec:
    command:
    - pg_isready
    - -U
    - admin
  initialDelaySeconds: 5
  periodSeconds: 5
```

### Estratégias Diferentes

```yaml
# Frontend - Rolling Update rápido
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0
    maxSurge: 2
```

```yaml
# API - Rolling Update conservador
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1
```

```yaml
# Database - Recreate (stateful)
strategy:
  type: Recreate
```

## 📊 Análise de Performance

### Métricas por Deployment

```bash
# CPU e memória por deployment
kubectl top pods -l app=frontend --sort-by=cpu
kubectl top pods -l app=api --sort-by=memory
kubectl top pods -l app=postgres --sort-by=cpu

# Resumo de recursos
kubectl describe nodes | grep -E "(Name:|Allocated resources:)" -A 5
```

### Comparação de Recursos

```bash
# Requests vs Limits
kubectl get deployments -o custom-columns=\
NAME:.metadata.name,\
CPU_REQ:.spec.template.spec.containers[0].resources.requests.cpu,\
CPU_LIM:.spec.template.spec.containers[0].resources.limits.cpu,\
MEM_REQ:.spec.template.spec.containers[0].resources.requests.memory,\
MEM_LIM:.spec.template.spec.containers[0].resources.limits.memory
```

## 🚨 Troubleshooting Múltiplos Deployments

### Problemas Comuns

#### 1. Recursos Insuficientes
```bash
# Verificar recursos disponíveis
kubectl describe nodes
kubectl top nodes

# Pods pendentes
kubectl get pods --field-selector=status.phase=Pending
```

#### 2. Conflitos de Porta
```bash
# Verificar services conflitantes
kubectl get services
kubectl describe service <service-name>
```

#### 3. Dependências entre Deployments
```bash
# Verificar ordem de inicialização
kubectl get pods --sort-by=.metadata.creationTimestamp
kubectl logs -l app=api --tail=50
```

### Debug Sistemático

```bash
# 1. Status geral
kubectl get deployments,rs,pods

# 2. Eventos do cluster
kubectl get events --sort-by=.metadata.creationTimestamp

# 3. Logs por aplicação
kubectl logs -l app=frontend --tail=20
kubectl logs -l app=api --tail=20
kubectl logs -l app=postgres --tail=20

# 4. Recursos do cluster
kubectl top nodes
kubectl top pods
```

## 📋 Comandos de Referência

### Criação e Aplicação
```bash
# Aplicar múltiplos arquivos
kubectl apply -f ./deployments/

# Aplicar com validação
kubectl apply -f deployment.yaml --validate=true

# Dry run
kubectl apply -f deployment.yaml --dry-run=client
```

### Monitoramento
```bash
# Status geral
kubectl get deployments,rs,pods -o wide

# Por labels
kubectl get all -l tier=web
kubectl get all -l component=service

# Watch múltiplos recursos
kubectl get deployments,pods -w
```

### Análise
```bash
# Detalhes completos
kubectl describe deployment <name>

# YAML/JSON
kubectl get deployment <name> -o yaml
kubectl get deployment <name> -o json

# Campos específicos
kubectl get deployments -o custom-columns=<columns>
```

### Limpeza
```bash
# Deletar deployment específico
kubectl delete deployment <name>

# Deletar por label
kubectl delete deployments -l tier=web

# Deletar múltiplos
kubectl delete -f ./deployments/
```

## 🎯 Boas Práticas

### Organização
- Use labels consistentes para agrupamento
- Organize deployments por namespace
- Documente dependências entre aplicações

### Recursos
- Defina requests/limits apropriados
- Monitore uso real vs configurado
- Ajuste conforme necessário

### Monitoramento
- Configure health checks adequados
- Use diferentes estratégias por tipo de aplicação
- Monitore logs e métricas regularmente

### Segurança
- Use diferentes service accounts
- Configure network policies
- Gerencie secrets adequadamente

## 🔗 Próximos Passos

Após dominar múltiplos deployments:
- [Services](criando-services-multiplos.md) - Expor aplicações
- [Namespaces](organizando-namespaces.md) - Organização
- [ConfigMaps](gerenciando-configs.md) - Configurações
- [Monitoring](monitoramento-deployments.md) - Observabilidade

## 📚 Referências

- [Deployment Best Practices](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#writing-a-deployment-spec)
- [Resource Management](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
- [Labels and Selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)
