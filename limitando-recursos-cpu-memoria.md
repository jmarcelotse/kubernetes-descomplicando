# Limitando Consumo de Recursos - CPU e Memória

Controle de recursos é fundamental para garantir estabilidade e performance em clusters Kubernetes.

## Conceitos Fundamentais

### Requests vs Limits
- **Requests**: Recursos garantidos para o container
- **Limits**: Recursos máximos que o container pode usar

### Unidades de Medida
**CPU:**
- `1` = 1 vCPU/core
- `100m` = 0.1 CPU (100 milicores)
- `500m` = 0.5 CPU

**Memória:**
- `128Mi` = 128 Mebibytes
- `1Gi` = 1 Gibibyte
- `512M` = 512 Megabytes

## Configuração Básica

### Pod com Recursos Limitados
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-limited-pod
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

### Múltiplos Containers
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-resources
spec:
  containers:
  - name: web
    image: nginx
    resources:
      requests:
        memory: "128Mi"
        cpu: "100m"
      limits:
        memory: "256Mi"
        cpu: "200m"
  - name: sidecar
    image: busybox
    command: ["sleep", "3600"]
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
```

## Exemplo Prático

### Manifesto Completo
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-demo
  labels:
    app: demo
spec:
  containers:
  - name: nginx-container
    image: nginx:latest
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "128Mi"
        cpu: "100m"
      limits:
        memory: "256Mi"
        cpu: "200m"
    env:
    - name: NGINX_PORT
      value: "80"
      
  - name: stress-container
    image: polinux/stress
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "50M", "--timeout", "3600s"]
    resources:
      requests:
        memory: "64Mi"
        cpu: "50m"
      limits:
        memory: "100Mi"
        cpu: "100m"
```

## Diagrama de Recursos

```
┌─────────────────────────────────────────────────────────────┐
│                    Node Resources                           │
├─────────────────────────────────────────────────────────────┤
│  Total: 4 CPU, 8Gi Memory                                  │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                Pod A                                │   │
│  │  ┌─────────────┐    ┌─────────────────────────┐     │   │
│  │  │Container 1  │    │     Container 2         │     │   │
│  │  │             │    │                         │     │   │
│  │  │Requests:    │    │ Requests:               │     │   │
│  │  │CPU: 100m    │    │ CPU: 50m                │     │   │
│  │  │Mem: 128Mi   │    │ Mem: 64Mi               │     │   │
│  │  │             │    │                         │     │   │
│  │  │Limits:      │    │ Limits:                 │     │   │
│  │  │CPU: 200m    │    │ CPU: 100m               │     │   │
│  │  │Mem: 256Mi   │    │ Mem: 128Mi              │     │   │
│  │  └─────────────┘    └─────────────────────────┘     │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                Pod B                                │   │
│  │  ┌─────────────────────────────────────────────┐    │   │
│  │  │            Container 3                      │    │   │
│  │  │                                             │    │   │
│  │  │ Requests: CPU: 200m, Mem: 256Mi             │    │   │
│  │  │ Limits:   CPU: 500m, Mem: 512Mi             │    │   │
│  │  └─────────────────────────────────────────────┘    │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  Available: CPU: 3.65, Memory: 7.4Gi                       │
└─────────────────────────────────────────────────────────────┘
```

## QoS Classes (Quality of Service)

### 1. Guaranteed
```yaml
resources:
  requests:
    memory: "128Mi"
    cpu: "100m"
  limits:
    memory: "128Mi"    # Igual ao request
    cpu: "100m"        # Igual ao request
```

### 2. Burstable
```yaml
resources:
  requests:
    memory: "64Mi"
    cpu: "50m"
  limits:
    memory: "128Mi"    # Maior que request
    cpu: "100m"        # Maior que request
```

### 3. BestEffort
```yaml
# Sem requests nem limits definidos
resources: {}
```

## Criando Exemplos Práticos

### Pod com Stress Test
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: stress-test
spec:
  containers:
  - name: stress
    image: polinux/stress
    command: ["stress"]
    args: ["--cpu", "1", "--timeout", "60s"]
    resources:
      requests:
        cpu: "100m"
        memory: "50Mi"
      limits:
        cpu: "200m"
        memory: "100Mi"
```

### Pod com Aplicação Real
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-limited
spec:
  containers:
  - name: webapp
    image: nginx
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "128Mi"
        cpu: "100m"
      limits:
        memory: "256Mi"
        cpu: "500m"
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 30
      periodSeconds: 10
```

## Comandos para Monitoramento

### Verificar Uso de Recursos
```bash
# Ver recursos dos pods
kubectl top pods

# Ver recursos dos nodes
kubectl top nodes

# Detalhes de recursos de um pod específico
kubectl describe pod <pod-name>

# Ver recursos com containers
kubectl top pods --containers
```

### Verificar QoS Class
```bash
# Ver QoS class do pod
kubectl get pod <pod-name> -o jsonpath='{.status.qosClass}'

# Detalhes completos
kubectl describe pod <pod-name> | grep "QoS Class"
```

## Comportamentos por QoS

### Guaranteed (Prioridade Alta)
- **Scheduling**: Sempre agendado se recursos disponíveis
- **Eviction**: Último a ser removido em pressão de recursos
- **Performance**: Recursos garantidos sempre disponíveis

### Burstable (Prioridade Média)
- **Scheduling**: Agendado baseado em requests
- **Eviction**: Removido após BestEffort
- **Performance**: Pode usar mais que request até o limit

### BestEffort (Prioridade Baixa)
- **Scheduling**: Agendado apenas se sobrar recursos
- **Eviction**: Primeiro a ser removido
- **Performance**: Sem garantias de recursos

## Exemplo de Pressão de Recursos

```
┌─────────────────────────────────────────────────────────────┐
│                Resource Pressure Scenario                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Node Memory: 2Gi (Available: 1.8Gi)                       │
│                                                             │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────┐ │
│  │   Guaranteed    │  │   Burstable     │  │ BestEffort  │ │
│  │   Pod A         │  │   Pod B         │  │   Pod C     │ │
│  │                 │  │                 │  │             │ │
│  │ Req: 512Mi      │  │ Req: 256Mi      │  │ No limits   │ │
│  │ Lim: 512Mi      │  │ Lim: 1Gi        │  │             │ │
│  │                 │  │                 │  │             │ │
│  │ Status: Safe    │  │ Status: Risk    │  │Status: Kill │ │
│  └─────────────────┘  └─────────────────┘  └─────────────┘ │
│                                                             │
│  Eviction Order: Pod C → Pod B → Pod A                     │
└─────────────────────────────────────────────────────────────┘
```

## Boas Práticas

### Definição de Recursos
```yaml
# ✅ Bom: Requests e limits definidos
resources:
  requests:
    memory: "128Mi"
    cpu: "100m"
  limits:
    memory: "256Mi"
    cpu: "200m"

# ❌ Evitar: Sem limites
resources: {}

# ❌ Evitar: Limits muito altos
resources:
  requests:
    memory: "64Mi"
    cpu: "50m"
  limits:
    memory: "8Gi"     # Muito alto
    cpu: "4"          # Muito alto
```

### Valores Recomendados

**Aplicações Web:**
```yaml
resources:
  requests:
    memory: "128Mi"
    cpu: "100m"
  limits:
    memory: "512Mi"
    cpu: "500m"
```

**Bancos de Dados:**
```yaml
resources:
  requests:
    memory: "1Gi"
    cpu: "500m"
  limits:
    memory: "2Gi"
    cpu: "1"
```

**Sidecars/Utilitários:**
```yaml
resources:
  requests:
    memory: "32Mi"
    cpu: "50m"
  limits:
    memory: "64Mi"
    cpu: "100m"
```

## Testando Limites

### Criar Pod de Teste
```bash
# Aplicar manifesto
kubectl apply -f resource-demo.yaml

# Monitorar recursos
kubectl top pods

# Ver detalhes
kubectl describe pod resource-demo
```

### Simular Pressão de CPU
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cpu-stress
spec:
  containers:
  - name: stress
    image: polinux/stress
    command: ["stress"]
    args: ["--cpu", "2", "--timeout", "300s"]
    resources:
      requests:
        cpu: "100m"
      limits:
        cpu: "200m"  # Será limitado aqui
```

### Simular Pressão de Memória
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: memory-stress
spec:
  containers:
  - name: stress
    image: polinux/stress
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "200M", "--timeout", "300s"]
    resources:
      requests:
        memory: "50Mi"
      limits:
        memory: "100Mi"  # Pod será morto se exceder
```

## Troubleshooting

### Pod OOMKilled
```bash
# Verificar eventos
kubectl describe pod <pod-name>

# Ver logs anteriores
kubectl logs <pod-name> --previous

# Verificar limites
kubectl get pod <pod-name> -o yaml | grep -A 10 resources
```

### CPU Throttling
```bash
# Monitorar uso de CPU
kubectl top pods --containers

# Ver métricas detalhadas
kubectl describe pod <pod-name>

# Verificar se está sendo limitado
# CPU usage próximo ao limit = throttling
```

### Pending Pods
```bash
# Ver motivo do pending
kubectl describe pod <pod-name>

# Verificar recursos do node
kubectl describe nodes

# Ver eventos do cluster
kubectl get events --sort-by=.metadata.creationTimestamp
```

## ResourceQuotas e LimitRanges

### ResourceQuota (Namespace)
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: development
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    pods: "10"
```

### LimitRange (Padrões)
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: development
spec:
  limits:
  - default:
      memory: "256Mi"
      cpu: "200m"
    defaultRequest:
      memory: "128Mi"
      cpu: "100m"
    type: Container
```

## Conclusão

Limitar recursos é essencial para:
- **Estabilidade**: Evitar que um pod consuma todos os recursos
- **Previsibilidade**: Garantir performance consistente
- **Eficiência**: Melhor utilização dos recursos do cluster
- **Isolamento**: Proteger aplicações críticas

**Regra de ouro**: Sempre defina requests e limits baseados no comportamento real da aplicação em produção.
