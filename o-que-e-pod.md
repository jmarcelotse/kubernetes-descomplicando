# O que é um Pod?

O **Pod** é a menor unidade de deployment no Kubernetes. É um wrapper que contém um ou mais containers que compartilham recursos como rede, armazenamento e ciclo de vida.

## Conceito Fundamental

Um Pod representa um "host lógico" para sua aplicação, similar a uma VM ou máquina física, mas para containers.

### Características Principais
- **Unidade atômica** - criado, agendado e destruído como uma unidade
- **Recursos compartilhados** - rede, volumes e namespace
- **Efêmero** - pods são substituíveis e não persistem dados por padrão
- **IP único** - cada pod recebe um endereço IP interno do cluster

## Anatomia de um Pod

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              POD                                        │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                        SHARED NETWORK                               ││
│  │                      IP: 10.244.1.5                                ││
│  └─────────────────────────────────────────────────────────────────────┘│
│                                                                         │
│  ┌─────────────────┐                           ┌─────────────────────┐  │
│  │   CONTAINER 1   │                           │   CONTAINER 2       │  │
│  │                 │                           │   (sidecar)         │  │
│  │   nginx:1.20    │                           │   logging-agent     │  │
│  │   Port: 80      │                           │                     │  │
│  └─────────────────┘                           └─────────────────────┘  │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                       SHARED VOLUMES                                ││
│  │   /var/log  ←→  /app/logs  ←→  /var/log/app                        ││
│  └─────────────────────────────────────────────────────────────────────┘│
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## Tipos de Pods

### 1. Single Container Pod
O padrão mais comum - um container por pod.

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

### 2. Multi Container Pod
Múltiplos containers que trabalham juntos.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-pod
spec:
  containers:
  - name: webapp
    image: nginx:1.20
    ports:
    - containerPort: 80
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx
  - name: log-agent
    image: fluentd:latest
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log
  volumes:
  - name: shared-logs
    emptyDir: {}
```

## Padrões Multi-Container

### Sidecar Pattern
Container auxiliar que estende a funcionalidade do principal.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         SIDECAR PATTERN                                │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────┐                           ┌─────────────────────┐  │
│  │  MAIN CONTAINER │                           │  SIDECAR CONTAINER  │  │
│  │                 │                           │                     │  │
│  │   Web Server    │ ←─── shared volume ────→  │   Log Collector     │  │
│  │   (nginx)       │                           │   (fluentd)         │  │
│  │                 │                           │                     │  │
│  └─────────────────┘                           └─────────────────────┘  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

**Casos de uso:**
- Coleta de logs
- Monitoramento
- Proxy de rede
- Certificados SSL

### Ambassador Pattern
Container que atua como proxy para comunicação externa.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                       AMBASSADOR PATTERN                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────┐                           ┌─────────────────────┐  │
│  │  MAIN CONTAINER │                           │ AMBASSADOR CONTAINER│  │
│  │                 │                           │                     │  │
│  │   Application   │ ────── localhost ──────→  │   Redis Proxy       │  │
│  │                 │                           │                     │  │
│  └─────────────────┘                           └─────────────────────┘  │
│                                                        │                │
│                                                        ▼                │
│                                              External Redis Cluster     │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

**Casos de uso:**
- Proxy para bancos de dados
- Service discovery
- Circuit breaker
- Rate limiting

### Adapter Pattern
Container que transforma dados para compatibilidade.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         ADAPTER PATTERN                                │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────┐                           ┌─────────────────────┐  │
│  │  MAIN CONTAINER │                           │  ADAPTER CONTAINER  │  │
│  │                 │                           │                     │  │
│  │   Legacy App    │ ──── custom format ────→  │   Format Converter  │  │
│  │   (old logs)    │                           │   (to JSON)         │  │
│  │                 │                           │                     │  │
│  └─────────────────┘                           └─────────────────────┘  │
│                                                        │                │
│                                                        ▼                │
│                                              Monitoring System          │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

**Casos de uso:**
- Conversão de formatos
- Normalização de dados
- Compatibilidade de APIs
- Transformação de métricas

## Ciclo de Vida do Pod

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          POD LIFECYCLE                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  PENDING ──────▶ RUNNING ──────▶ SUCCEEDED                             │
│     │               │               │                                   │
│     │               │               │                                   │
│     ▼               ▼               ▼                                   │
│  FAILED ◄───────── FAILED ◄─── TERMINATED                              │
│                                                                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐    │
│  │   PENDING   │  │   RUNNING   │  │ SUCCEEDED   │  │   FAILED    │    │
│  │             │  │             │  │             │  │             │    │
│  │ • Scheduling│  │ • Containers│  │ • All done  │  │ • Error     │    │
│  │ • Pulling   │  │   running   │  │ • Exit 0    │  │ • Exit != 0 │    │
│  │   images    │  │ • Ready     │  │             │  │ • Timeout   │    │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Fases do Pod

#### 1. Pending
- Pod foi aceito pelo cluster
- Containers ainda não foram criados
- Pode estar baixando imagens ou aguardando agendamento

#### 2. Running
- Pod foi agendado em um nó
- Todos os containers foram criados
- Pelo menos um container está rodando

#### 3. Succeeded
- Todos os containers terminaram com sucesso
- Não serão reiniciados
- Comum em Jobs

#### 4. Failed
- Todos os containers terminaram
- Pelo menos um falhou (exit code != 0)

#### 5. Unknown
- Estado do pod não pode ser determinado
- Geralmente por problemas de comunicação com o nó

## Recursos Compartilhados

### Rede
```yaml
# Todos os containers no pod compartilham:
# - Endereço IP
# - Portas de rede
# - Interface loopback
apiVersion: v1
kind: Pod
metadata:
  name: network-pod
spec:
  containers:
  - name: container1
    image: nginx
    ports:
    - containerPort: 80
  - name: container2
    image: busybox
    command: ['sh', '-c', 'wget -qO- localhost:80']
```

### Volumes
```yaml
# Containers podem compartilhar volumes
apiVersion: v1
kind: Pod
metadata:
  name: volume-pod
spec:
  containers:
  - name: writer
    image: busybox
    command: ['sh', '-c', 'echo "Hello" > /shared/message.txt; sleep 3600']
    volumeMounts:
    - name: shared-data
      mountPath: /shared
  - name: reader
    image: busybox
    command: ['sh', '-c', 'cat /shared/message.txt; sleep 3600']
    volumeMounts:
    - name: shared-data
      mountPath: /shared
  volumes:
  - name: shared-data
    emptyDir: {}
```

## Configurações Importantes

### Resource Limits
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-pod
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

### Environment Variables
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-pod
spec:
  containers:
  - name: app
    image: nginx
    env:
    - name: ENV
      value: "production"
    - name: DEBUG
      value: "false"
    - name: DB_HOST
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: host
```

### Health Checks
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: health-pod
spec:
  containers:
  - name: app
    image: nginx
    ports:
    - containerPort: 80
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

## Comandos Úteis

### Criar Pod
```bash
# Imperativo
kubectl run nginx --image=nginx --port=80

# Declarativo
kubectl apply -f pod.yaml
```

### Gerenciar Pods
```bash
# Listar pods
kubectl get pods
kubectl get pods -o wide
kubectl get pods --show-labels

# Detalhes do pod
kubectl describe pod <nome-do-pod>

# Logs
kubectl logs <nome-do-pod>
kubectl logs <nome-do-pod> -c <nome-do-container>  # multi-container
kubectl logs -f <nome-do-pod>  # follow

# Executar comandos
kubectl exec <nome-do-pod> -- <comando>
kubectl exec -it <nome-do-pod> -- /bin/bash

# Port forward
kubectl port-forward pod/<nome-do-pod> 8080:80

# Deletar pod
kubectl delete pod <nome-do-pod>
kubectl delete -f pod.yaml
```

## Boas Práticas

### 1. Design de Containers
- **Um processo por container** - cada container deve ter uma responsabilidade
- **Containers stateless** - dados persistentes em volumes
- **Imagens leves** - use imagens base mínimas

### 2. Recursos
- **Sempre defina limits** - evita consumo excessivo de recursos
- **Use requests apropriados** - ajuda no agendamento
- **Monitor recursos** - acompanhe uso de CPU e memória

### 3. Health Checks
- **Implemente probes** - liveness e readiness
- **Timeouts adequados** - não muito agressivos
- **Endpoints específicos** - não use apenas "/"

### 4. Logs
- **Log para stdout/stderr** - Kubernetes coleta automaticamente
- **Structured logging** - JSON para melhor parsing
- **Não armazene logs em arquivos** - use volumes se necessário

### 5. Segurança
- **Não rode como root** - use usuários não privilegiados
- **Security contexts** - defina políticas de segurança
- **Secrets para dados sensíveis** - não use variáveis de ambiente

## Limitações dos Pods

### 1. Não são persistentes
- Pods são efêmeros por natureza
- Dados são perdidos quando o pod é deletado
- Use Deployments para aplicações stateless

### 2. Não fazem auto-scaling
- Pods individuais não escalam automaticamente
- Use Deployments ou ReplicaSets

### 3. Não fazem rolling updates
- Atualizações requerem recriação do pod
- Use Deployments para updates sem downtime

### 4. Scheduling limitado
- Pod é agendado como uma unidade
- Todos os containers devem caber no mesmo nó

## Quando Usar Pods Diretamente

### ✅ Casos Apropriados
- **Desenvolvimento e testes** - pods temporários
- **Jobs únicos** - tarefas que rodam uma vez
- **Debug e troubleshooting** - pods temporários para diagnóstico

### ❌ Evite Para
- **Aplicações de produção** - use Deployments
- **Serviços críticos** - use controladores
- **Aplicações que precisam escalar** - use ReplicaSets/Deployments

## Próximos Passos

Após entender pods, explore:
- **Deployments** - para gerenciar pods em produção
- **Services** - para expor pods na rede
- **ConfigMaps e Secrets** - para configurações
- **Persistent Volumes** - para dados persistentes
- **Ingress** - para acesso externo
