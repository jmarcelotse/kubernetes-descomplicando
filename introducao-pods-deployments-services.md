# Introdução aos Objetos Fundamentais do Kubernetes

Este documento apresenta os conceitos básicos dos objetos principais do Kubernetes: Pods, ReplicaSets, Deployments e Services.

## Hierarquia dos Objetos

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    KUBERNETES OBJECTS HIERARCHY                        │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                       DEPLOYMENT                                   ││
│  │                    (Manages ReplicaSets)                           ││
│  │  ┌─────────────────────────────────────────────────────────────────┐││
│  │  │                     REPLICASET                                 │││
│  │  │                   (Manages Pods)                               │││
│  │  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐│││
│  │  │  │    POD 1    │  │    POD 2    │  │         POD 3           ││││
│  │  │  │             │  │             │  │                         ││││
│  │  │  │ ┌─────────┐ │  │ ┌─────────┐ │  │ ┌─────────────────────┐ ││││
│  │  │  │ │Container│ │  │ │Container│ │  │ │      Container      │ ││││
│  │  │  │ └─────────┘ │  │ └─────────┘ │  │ └─────────────────────┘ ││││
│  │  │  └─────────────┘  └─────────────┘  └─────────────────────────┘│││
│  │  └─────────────────────────────────────────────────────────────────┘││
│  └─────────────────────────────────────────────────────────────────────┘│
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                        SERVICE                                     ││
│  │                   (Load Balancer)                                  ││
│  │                                                                     ││
│  │    External Traffic ──────────────────────────────────────────────┐ ││
│  │           │                                                       │ ││
│  │           ▼                                                       │ ││
│  │    ┌─────────────┐                                                │ ││
│  │    │   Service   │ ──────────────────────────────────────────────┘ ││
│  │    │ 10.96.0.100 │                                                  ││
│  │    └─────────────┘                                                  ││
│  │           │                                                          ││
│  │           └──────────── Routes to Pods ─────────────────────────────┤│
│  └─────────────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────────────┘
```

## 1. Pod

O **Pod** é a menor unidade deployável no Kubernetes.

### Características
- Contém um ou mais containers
- Compartilha rede e storage
- Tem um IP único no cluster
- É efêmero (pode ser criado/destruído)

### Diagrama do Pod

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           POD ANATOMY                                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                        POD                                         ││
│  │                   IP: 10.244.1.10                                  ││
│  │                                                                     ││
│  │  ┌─────────────────┐              ┌─────────────────────────────────┐││
│  │  │   Container 1   │              │         Container 2             │││
│  │  │                 │              │                                 │││
│  │  │ ┌─────────────┐ │              │ ┌─────────────────────────────┐ │││
│  │  │ │    App      │ │              │ │        Sidecar              │ │││
│  │  │ │   :8080     │ │              │ │       (logging)             │ │││
│  │  │ └─────────────┘ │              │ └─────────────────────────────┘ │││
│  │  └─────────────────┘              └─────────────────────────────────┘││
│  │                                                                     ││
│  │  ┌─────────────────────────────────────────────────────────────────┐││
│  │  │                    SHARED RESOURCES                            │││
│  │  │                                                                 │││
│  │  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐ │││
│  │  │  │  Network    │  │   Storage   │  │      Environment        │ │││
│  │  │  │ Namespace   │  │   Volumes   │  │      Variables          │ │││
│  │  │  └─────────────┘  └─────────────┘  └─────────────────────────┘ │││
│  │  └─────────────────────────────────────────────────────────────────┘││
│  └─────────────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────────────┘
```

### Exemplo YAML
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.20
    ports:
    - containerPort: 80
```

## 2. ReplicaSet

O **ReplicaSet** garante que um número específico de pods esteja sempre executando.

### Características
- Mantém número desejado de réplicas
- Substitui pods que falham
- Usa seletores para identificar pods
- Raramente criado diretamente

### Diagrama do ReplicaSet

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        REPLICASET OPERATION                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                      REPLICASET                                    ││
│  │                    Desired: 3 pods                                 ││
│  │                    Current: 3 pods                                 ││
│  │                                                                     ││
│  │  Selector: app=nginx                                                ││
│  └─────────────────────────────────────────────────────────────────────┘│
│                                │                                        │
│                                │ Manages                                │
│                                ▼                                        │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────────────┐ │
│  │   Pod 1     │    │   Pod 2     │    │         Pod 3               │ │
│  │             │    │             │    │                             │ │
│  │ app=nginx   │    │ app=nginx   │    │       app=nginx             │ │
│  │ Running     │    │ Running     │    │       Running               │ │
│  └─────────────┘    └─────────────┘    └─────────────────────────────┘ │
│                                                                         │
│  Cenário: Pod 2 falha                                                  │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────────────┐ │
│  │   Pod 1     │    │   Pod 2     │    │         Pod 3               │ │
│  │             │    │             │    │                             │ │
│  │ app=nginx   │    │ app=nginx   │    │       app=nginx             │ │
│  │ Running     │    │ Failed ❌   │    │       Running               │ │
│  └─────────────┘    └─────────────┘    └─────────────────────────────┘ │
│                                │                                        │
│                                │ ReplicaSet detects                     │
│                                ▼                                        │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────────────┐ │
│  │   Pod 1     │    │   Pod 4     │    │         Pod 3               │ │
│  │             │    │   (NEW)     │    │                             │ │
│  │ app=nginx   │    │ app=nginx   │    │       app=nginx             │ │
│  │ Running     │    │ Running     │    │       Running               │ │
│  └─────────────┘    └─────────────┘    └─────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
```

### Exemplo YAML
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
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
        image: nginx:1.20
```
## 3. Deployment

O **Deployment** é o objeto mais usado para gerenciar aplicações no Kubernetes.

### Características
- Gerencia ReplicaSets automaticamente
- Permite rolling updates
- Suporte a rollback
- Declarativo (estado desejado)

### Diagrama do Deployment

```
┌─────────────────────────────────────────────────────────────────────────┐
│                       DEPLOYMENT LIFECYCLE                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                      DEPLOYMENT                                    ││
│  │                   nginx-deployment                                  ││
│  │                                                                     ││
│  │  Strategy: RollingUpdate                                            ││
│  │  Replicas: 3                                                        ││
│  │  Image: nginx:1.20 → nginx:1.21                                     ││
│  └─────────────────────────────────────────────────────────────────────┘│
│                                │                                        │
│                                │ Creates/Manages                        │
│                                ▼                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                    ROLLING UPDATE PROCESS                          ││
│  │                                                                     ││
│  │  Step 1: Current State                                              ││
│  │  ┌─────────────────┐                                                ││
│  │  │ ReplicaSet v1   │ ──────────────────────────────────────────────┐││
│  │  │ nginx:1.20      │                                               │││
│  │  │ 3 pods          │                                               │││
│  │  └─────────────────┘                                               │││
│  │  ┌───┐ ┌───┐ ┌───┐                                                 │││
│  │  │v1 │ │v1 │ │v1 │                                                 │││
│  │  └───┘ └───┘ └───┘                                                 │││
│  │                                                                     │││
│  │  Step 2: Create New ReplicaSet                                      ││
│  │  ┌─────────────────┐  ┌─────────────────────────────────────────┐  │││
│  │  │ ReplicaSet v1   │  │        ReplicaSet v2                    │  │││
│  │  │ nginx:1.20      │  │        nginx:1.21                       │  │││
│  │  │ 3 pods          │  │        1 pod                            │  │││
│  │  └─────────────────┘  └─────────────────────────────────────────┘  │││
│  │  ┌───┐ ┌───┐ ┌───┐    ┌───┐                                        │││
│  │  │v1 │ │v1 │ │v1 │    │v2 │                                        │││
│  │  └───┘ └───┘ └───┘    └───┘                                        │││
│  │                                                                     │││
│  │  Step 3: Scale Down Old, Scale Up New                               ││
│  │  ┌─────────────────┐  ┌─────────────────────────────────────────┐  │││
│  │  │ ReplicaSet v1   │  │        ReplicaSet v2                    │  │││
│  │  │ nginx:1.20      │  │        nginx:1.21                       │  │││
│  │  │ 2 pods          │  │        2 pods                           │  │││
│  │  └─────────────────┘  └─────────────────────────────────────────┘  │││
│  │  ┌───┐ ┌───┐          ┌───┐ ┌───┐                                  │││
│  │  │v1 │ │v1 │          │v2 │ │v2 │                                  │││
│  │  └───┘ └───┘          └───┘ └───┘                                  │││
│  │                                                                     │││
│  │  Step 4: Complete Migration                                         ││
│  │  ┌─────────────────┐  ┌─────────────────────────────────────────┐  │││
│  │  │ ReplicaSet v1   │  │        ReplicaSet v2                    │  │││
│  │  │ nginx:1.20      │  │        nginx:1.21                       │  │││
│  │  │ 0 pods          │  │        3 pods                           │  │││
│  │  └─────────────────┘  └─────────────────────────────────────────┘  │││
│  │                        ┌───┐ ┌───┐ ┌───┐                          │││
│  │                        │v2 │ │v2 │ │v2 │                          │││
│  │                        └───┘ └───┘ └───┘                          │││
│  └─────────────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────────────┘
```

### Exemplo YAML
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
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
        image: nginx:1.20
        ports:
        - containerPort: 80
```

## 4. Service

O **Service** fornece uma forma estável de acessar pods.

### Características
- IP e DNS estáveis
- Load balancing automático
- Service discovery
- Diferentes tipos (ClusterIP, NodePort, LoadBalancer)

### Diagrama do Service

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        SERVICE TYPES                                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                      CLUSTERIP                                     ││
│  │                   (Internal Only)                                   ││
│  │                                                                     ││
│  │    Pod A ──────────────────────────────────────────────── Pod B    ││
│  │      │                                                       │      ││
│  │      │                 ┌─────────────┐                       │      ││
│  │      └────────────────▶│   Service   │◄──────────────────────┘      ││
│  │                        │ 10.96.0.100 │                              ││
│  │                        │ ClusterIP   │                              ││
│  │                        └─────────────┘                              ││
│  │                               │                                      ││
│  │                               │ Load Balance                         ││
│  │                               ▼                                      ││
│  │    ┌─────────────┐    ┌─────────────┐    ┌─────────────────────────┐││
│  │    │   Pod 1     │    │   Pod 2     │    │         Pod 3           │││
│  │    │ app=nginx   │    │ app=nginx   │    │       app=nginx         │││
│  │    │ :80         │    │ :80         │    │       :80               │││
│  │    └─────────────┘    └─────────────┘    └─────────────────────────┘││
│  └─────────────────────────────────────────────────────────────────────┘│
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                       NODEPORT                                     ││
│  │                   (External Access)                                 ││
│  │                                                                     ││
│  │  External Client                                                    ││
│  │         │                                                           ││
│  │         │ :30080                                                    ││
│  │         ▼                                                           ││
│  │  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────────┐ ││
│  │  │ Worker Node │    │ Worker Node │    │       Worker Node       │ ││
│  │  │ :30080      │    │ :30080      │    │       :30080            │ ││
│  │  └─────────────┘    └─────────────┘    └─────────────────────────┘ ││
│  │         │                   │                         │             ││
│  │         └───────────────────┼─────────────────────────┘             ││
│  │                             ▼                                       ││
│  │                    ┌─────────────┐                                  ││
│  │                    │   Service   │                                  ││
│  │                    │ 10.96.0.100 │                                  ││
│  │                    │ NodePort    │                                  ││
│  │                    └─────────────┘                                  ││
│  │                             │                                       ││
│  │                             ▼                                       ││
│  │    ┌─────────────┐    ┌─────────────┐    ┌─────────────────────────┐││
│  │    │   Pod 1     │    │   Pod 2     │    │         Pod 3           │││
│  │    │ app=nginx   │    │ app=nginx   │    │       app=nginx         │││
│  │    └─────────────┘    └─────────────┘    └─────────────────────────┘││
│  └─────────────────────────────────────────────────────────────────────┘│
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                     LOADBALANCER                                   ││
│  │                   (Cloud Provider)                                  ││
│  │                                                                     ││
│  │  Internet                                                           ││
│  │     │                                                               ││
│  │     ▼                                                               ││
│  │  ┌─────────────┐                                                    ││
│  │  │Cloud Load   │                                                    ││
│  │  │Balancer     │                                                    ││
│  │  │External IP  │                                                    ││
│  │  └─────────────┘                                                    ││
│  │         │                                                           ││
│  │         ▼                                                           ││
│  │  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────────┐ ││
│  │  │ Worker Node │    │ Worker Node │    │       Worker Node       │ ││
│  │  └─────────────┘    └─────────────┘    └─────────────────────────┘ ││
│  │         │                   │                         │             ││
│  │         └───────────────────┼─────────────────────────┘             ││
│  │                             ▼                                       ││
│  │                    ┌─────────────┐                                  ││
│  │                    │   Service   │                                  ││
│  │                    │LoadBalancer │                                  ││
│  │                    └─────────────┘                                  ││
│  │                             │                                       ││
│  │                             ▼                                       ││
│  │    ┌─────────────┐    ┌─────────────┐    ┌─────────────────────────┐││
│  │    │   Pod 1     │    │   Pod 2     │    │         Pod 3           │││
│  │    │ app=nginx   │    │ app=nginx   │    │       app=nginx         │││
│  │    └─────────────┘    └─────────────┘    └─────────────────────────┘││
│  └─────────────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────────────┘
```

### Exemplo YAML
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

## Fluxo Completo de Deploy

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      COMPLETE DEPLOYMENT FLOW                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  1. kubectl apply -f deployment.yaml                                    │
│     │                                                                   │
│     ▼                                                                   │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                      API SERVER                                    ││
│  │                   Validates YAML                                    ││
│  │                   Stores in etcd                                    ││
│  └─────────────────────────────────────────────────────────────────────┘│
│     │                                                                   │
│     ▼                                                                   │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                 DEPLOYMENT CONTROLLER                              ││
│  │                Creates ReplicaSet                                   ││
│  └─────────────────────────────────────────────────────────────────────┘│
│     │                                                                   │
│     ▼                                                                   │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                REPLICASET CONTROLLER                               ││
│  │                  Creates Pods                                       ││
│  └─────────────────────────────────────────────────────────────────────┘│
│     │                                                                   │
│     ▼                                                                   │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                     SCHEDULER                                      ││
│  │              Assigns Pods to Nodes                                  ││
│  └─────────────────────────────────────────────────────────────────────┘│
│     │                                                                   │
│     ▼                                                                   │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                      KUBELET                                       ││
│  │                 Starts Containers                                   ││
│  └─────────────────────────────────────────────────────────────────────┘│
│     │                                                                   │
│     ▼                                                                   │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────────────┐ │
│  │   Pod 1     │    │   Pod 2     │    │         Pod 3               │ │
│  │   Running   │    │   Running   │    │       Running               │ │
│  └─────────────┘    └─────────────┘    └─────────────────────────────┘ │
│                                                                         │
│  2. kubectl apply -f service.yaml                                       │
│     │                                                                   │
│     ▼                                                                   │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                       SERVICE                                      ││
│  │                 Exposes Pods                                        ││
│  │               Load Balances Traffic                                  ││
│  └─────────────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────────────┘
```

## Comandos Básicos

### Gerenciamento de Pods
```bash
# Listar pods
kubectl get pods

# Descrever pod
kubectl describe pod <pod-name>

# Ver logs
kubectl logs <pod-name>

# Executar comando no pod
kubectl exec -it <pod-name> -- /bin/bash
```

### Gerenciamento de Deployments
```bash
# Criar deployment
kubectl create deployment nginx --image=nginx

# Escalar deployment
kubectl scale deployment nginx --replicas=5

# Atualizar imagem
kubectl set image deployment/nginx nginx=nginx:1.21

# Ver histórico
kubectl rollout history deployment/nginx

# Rollback
kubectl rollout undo deployment/nginx
```

### Gerenciamento de Services
```bash
# Expor deployment
kubectl expose deployment nginx --port=80 --type=ClusterIP

# Listar services
kubectl get services

# Testar conectividade
kubectl port-forward service/nginx 8080:80
```

## Resumo dos Relacionamentos

| Objeto | Gerencia | Finalidade |
|--------|----------|------------|
| **Pod** | Containers | Menor unidade deployável |
| **ReplicaSet** | Pods | Garantir número de réplicas |
| **Deployment** | ReplicaSets | Rolling updates e rollbacks |
| **Service** | Acesso aos Pods | Load balancing e service discovery |
