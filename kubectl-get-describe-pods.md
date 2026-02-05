# Os Sensacionais kubectl get pods e kubectl describe pods

Os comandos `kubectl get pods` e `kubectl describe pods` são fundamentais para monitoramento e troubleshooting de pods no Kubernetes.

## kubectl get pods

O comando `kubectl get pods` lista todos os pods em um namespace, fornecendo uma visão geral rápida do estado dos containers.

### Sintaxe Básica
```bash
kubectl get pods [flags]
```

### Saída Padrão
```bash
kubectl get pods
```
```
NAME       READY   STATUS    RESTARTS   AGE
nginx-pod  1/1     Running   0          5m
app-pod    0/1     Pending   0          30s
db-pod     1/1     Running   2          1h
```

### Colunas Explicadas

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        KUBECTL GET PODS OUTPUT                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  NAME        READY    STATUS      RESTARTS    AGE                       │
│  ┌─────────┐ ┌─────┐  ┌────────┐  ┌────────┐  ┌─────┐                   │
│  │Pod Name │ │ 1/1 │  │Running │  │   0    │  │ 5m  │                   │
│  └─────────┘ └─────┘  └────────┘  └────────┘  └─────┘                   │
│      │         │         │          │          │                       │
│      │         │         │          │          └─ Tempo desde criação   │
│      │         │         │          └─ Número de reinicializações       │
│      │         │         └─ Estado atual do pod                         │
│      │         └─ Containers prontos / Total de containers              │
│      └─ Nome único do pod                                               │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

#### NAME
- Nome único do pod no namespace
- Definido no metadata.name do YAML

#### READY
- **Formato**: `containers_prontos/total_containers`
- **1/1**: 1 container pronto de 1 total
- **2/3**: 2 containers prontos de 3 total
- **0/1**: nenhum container pronto

#### STATUS
Estados possíveis do pod:
- **Pending**: aguardando agendamento ou download de imagens
- **Running**: pelo menos um container está executando
- **Succeeded**: todos os containers terminaram com sucesso
- **Failed**: pelo menos um container falhou
- **Unknown**: estado não pode ser determinado

#### RESTARTS
- Número total de reinicializações dos containers
- Indica problemas de estabilidade se muito alto

#### AGE
- Tempo desde a criação do pod
- Formatos: s (segundos), m (minutos), h (horas), d (dias)

## Variações do kubectl get pods

### Informações Detalhadas (-o wide)
```bash
kubectl get pods -o wide
```
```
NAME      READY  STATUS   RESTARTS  AGE  IP          NODE     NOMINATED NODE
nginx-pod 1/1    Running  0         5m   10.244.1.4  worker1  <none>
```

**Colunas adicionais:**
- **IP**: endereço IP interno do pod
- **NODE**: nó onde o pod está executando
- **NOMINATED NODE**: nó reservado para o pod

### Todos os Namespaces (-A ou --all-namespaces)
```bash
kubectl get pods -A
```
```
NAMESPACE     NAME                          READY   STATUS    RESTARTS   AGE
default       nginx-pod                     1/1     Running   0          5m
kube-system   coredns-7d764666f9-abc123     1/1     Running   0          1h
kube-system   kube-proxy-xyz789             1/1     Running   0          1h
```

### Com Labels (--show-labels)
```bash
kubectl get pods --show-labels
```
```
NAME      READY  STATUS   RESTARTS  AGE  LABELS
nginx-pod 1/1    Running  0         5m   app=nginx,version=1.20
```

### Filtrar por Labels (-l ou --selector)
```bash
kubectl get pods -l app=nginx
kubectl get pods --selector="app=nginx,version=1.20"
```

### Filtrar por Field (--field-selector)
```bash
kubectl get pods --field-selector status.phase=Running
kubectl get pods --field-selector metadata.namespace=default
```

### Formatos de Saída (-o)

#### YAML
```bash
kubectl get pods nginx-pod -o yaml
```

#### JSON
```bash
kubectl get pods nginx-pod -o json
```

#### Custom Columns
```bash
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase,IP:.status.podIP
```

#### JSONPath
```bash
kubectl get pods -o jsonpath='{.items[*].metadata.name}'
```

## kubectl describe pods

O comando `kubectl describe pods` fornece informações detalhadas sobre um ou mais pods, incluindo eventos e configurações.

### Sintaxe
```bash
kubectl describe pod <nome-do-pod>
kubectl describe pods  # todos os pods
```

### Estrutura da Saída

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      KUBECTL DESCRIBE POD OUTPUT                       │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                        METADATA                                     ││
│  │  Name, Namespace, Labels, Annotations                               ││
│  └─────────────────────────────────────────────────────────────────────┘│
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                         STATUS                                      ││
│  │  Phase, IP, Node, Start Time                                        ││
│  └─────────────────────────────────────────────────────────────────────┘│
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                       CONTAINERS                                    ││
│  │  Image, Ports, Environment, Mounts, Resources                       ││
│  └─────────────────────────────────────────────────────────────────────┘│
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                        CONDITIONS                                   ││
│  │  PodScheduled, ContainersReady, Initialized, Ready                  ││
│  └─────────────────────────────────────────────────────────────────────┘│
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                         VOLUMES                                     ││
│  │  Mounted volumes and their sources                                  ││
│  └─────────────────────────────────────────────────────────────────────┘│
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                         EVENTS                                      ││
│  │  Timeline of pod lifecycle events                                   ││
│  └─────────────────────────────────────────────────────────────────────┘│
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Exemplo de Saída Completa

```bash
kubectl describe pod nginx-pod
```

```
Name:         nginx-pod
Namespace:    default
Priority:     0
Node:         worker1/192.168.1.100
Start Time:   Mon, 03 Feb 2024 10:00:00 +0000
Labels:       app=nginx
              version=1.20
Annotations:  <none>
Status:       Running
IP:           10.244.1.4
IPs:
  IP:  10.244.1.4

Containers:
  nginx:
    Container ID:   containerd://abc123def456
    Image:          nginx:1.20
    Image ID:       docker.io/library/nginx@sha256:xyz789
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Mon, 03 Feb 2024 10:00:30 +0000
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     500m
      memory:  128Mi
    Requests:
      cpu:        250m
      memory:     64Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-abc123 (ro)

Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True

Volumes:
  kube-api-access-abc123:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true

QoS Class:       Burstable
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s

Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  5m    default-scheduler  Successfully assigned default/nginx-pod to worker1
  Normal  Pulling    5m    kubelet            Pulling image "nginx:1.20"
  Normal  Pulled     4m    kubelet            Successfully pulled image "nginx:1.20"
  Normal  Created    4m    kubelet            Created container nginx
  Normal  Started    4m    kubelet            Started container nginx
```

### Seções Importantes

#### Metadata
- **Name**: nome do pod
- **Namespace**: namespace onde está o pod
- **Labels**: labels aplicados ao pod
- **Annotations**: metadados adicionais

#### Status
- **Phase**: estado atual (Running, Pending, etc.)
- **IP**: endereço IP do pod
- **Node**: nó onde está executando
- **Start Time**: quando o pod foi iniciado

#### Containers
Para cada container:
- **Image**: imagem Docker utilizada
- **State**: estado do container (Running, Waiting, Terminated)
- **Restart Count**: número de reinicializações
- **Resources**: limits e requests de CPU/memória
- **Environment**: variáveis de ambiente
- **Mounts**: volumes montados

#### Conditions
Estados booleanos do pod:
- **PodScheduled**: pod foi agendado em um nó
- **Initialized**: init containers completaram
- **ContainersReady**: todos os containers estão prontos
- **Ready**: pod está pronto para receber tráfego

#### Events
Timeline cronológica dos eventos do pod:
- **Scheduled**: pod foi agendado
- **Pulling**: baixando imagem
- **Pulled**: imagem baixada
- **Created**: container criado
- **Started**: container iniciado
- **Failed**: falhas ocorridas

## Fluxo de Troubleshooting

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      TROUBLESHOOTING WORKFLOW                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  1. OVERVIEW           2. DETAILS            3. DEEP DIVE              │
│  ┌─────────────┐      ┌─────────────┐       ┌─────────────┐             │
│  │kubectl get  │ ───▶ │kubectl      │ ───▶  │kubectl logs │             │
│  │pods         │      │describe pod │       │kubectl exec │             │
│  └─────────────┘      └─────────────┘       └─────────────┘             │
│                                                                         │
│  • Status geral       • Configuração        • Logs detalhados          │
│  • Problemas óbvios   • Eventos             • Debug interativo         │
│  • Estado dos pods    • Resources           • Comandos no container     │
│                       • Conditions                                      │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## Cenários Comuns de Troubleshooting

### Pod em Estado Pending
```bash
kubectl get pods
# NAME      READY   STATUS    RESTARTS   AGE
# app-pod   0/1     Pending   0          2m

kubectl describe pod app-pod
# Verificar Events para:
# - Insufficient resources
# - Node selector issues
# - Image pull problems
```

### Pod com Restart Alto
```bash
kubectl get pods
# NAME      READY   STATUS    RESTARTS   AGE
# app-pod   1/1     Running   15         1h

kubectl describe pod app-pod
# Verificar:
# - Liveness probe failures
# - OOMKilled events
# - Application crashes
```

### Pod em CrashLoopBackOff
```bash
kubectl get pods
# NAME      READY   STATUS             RESTARTS   AGE
# app-pod   0/1     CrashLoopBackOff   5          5m

kubectl describe pod app-pod
# Verificar Events e depois:
kubectl logs app-pod
kubectl logs app-pod --previous  # logs da execução anterior
```

### Pod com ImagePullBackOff
```bash
kubectl get pods
# NAME      READY   STATUS             RESTARTS   AGE
# app-pod   0/1     ImagePullBackOff   0          2m

kubectl describe pod app-pod
# Verificar Events para:
# - Image name typos
# - Registry authentication
# - Network issues
```

## Comandos Avançados

### Watch Mode (-w)
```bash
kubectl get pods -w
# Monitora mudanças em tempo real
```

### Sort By
```bash
kubectl get pods --sort-by=.metadata.creationTimestamp
kubectl get pods --sort-by=.status.startTime
```

### No Headers (--no-headers)
```bash
kubectl get pods --no-headers
# nginx-pod   1/1   Running   0   5m
```

### Output para Scripts
```bash
# Apenas nomes dos pods
kubectl get pods -o name
# pod/nginx-pod
# pod/app-pod

# Status de todos os pods
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.phase}{"\n"}{end}'
```

### Describe com Seletores
```bash
kubectl describe pods -l app=nginx
kubectl describe pods --field-selector status.phase=Running
```

## Monitoramento Contínuo

### Script de Monitoramento
```bash
#!/bin/bash
# monitor-pods.sh

while true; do
  echo "=== $(date) ==="
  kubectl get pods -o wide
  echo
  
  # Pods com problemas
  kubectl get pods --field-selector status.phase!=Running
  
  sleep 30
done
```

### Aliases Úteis
```bash
# Adicionar ao ~/.bashrc ou ~/.zshrc
alias kgp='kubectl get pods'
alias kgpw='kubectl get pods -o wide'
alias kgpa='kubectl get pods -A'
alias kdp='kubectl describe pod'
alias kgpwatch='kubectl get pods -w'
```

## Boas Práticas

### 1. Monitoramento Regular
- Use `kubectl get pods` para visão geral diária
- Configure alertas para pods em estado não-Running
- Monitor restart counts regularmente

### 2. Troubleshooting Sistemático
- Sempre comece com `kubectl get pods`
- Use `kubectl describe pod` para detalhes
- Verifique logs com `kubectl logs`
- Use `kubectl exec` para debug interativo

### 3. Filtros Eficientes
- Use labels para organizar pods
- Aproveite field selectors para filtros específicos
- Combine com grep para buscas textuais

### 4. Automação
- Crie scripts para checks recorrentes
- Use watch mode para monitoramento em tempo real
- Configure aliases para comandos frequentes

## Limitações e Considerações

### Performance
- `kubectl get pods -A` pode ser lento em clusters grandes
- Use namespaces específicos quando possível
- Limite output com field selectors

### Segurança
- Verifique permissões RBAC para diferentes namespaces
- Alguns campos podem estar ocultos por políticas
- Logs podem conter informações sensíveis

### Rede
- Comandos dependem de conectividade com API server
- Timeouts podem ocorrer em redes lentas
- Use `--request-timeout` para ajustar timeouts

## Próximos Passos

Após dominar estes comandos, explore:
- **kubectl logs** - análise detalhada de logs
- **kubectl exec** - debug interativo
- **kubectl port-forward** - acesso direto aos pods
- **kubectl top** - métricas de recursos
- **kubectl events** - eventos do cluster
