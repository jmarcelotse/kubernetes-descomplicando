# Workers e Control Plane do Kubernetes

O cluster Kubernetes é dividido em duas partes principais: **Control Plane** (plano de controle) e **Worker Nodes** (nós de trabalho).

## Control Plane

O **Control Plane** é o cérebro do cluster, responsável por tomar decisões globais e detectar/responder a eventos do cluster.

### Componentes do Control Plane

#### API Server (kube-apiserver)
- Ponto de entrada para todas as operações do cluster
- Expõe a API REST do Kubernetes
- Autentica e autoriza requisições
- Valida e processa objetos da API

#### etcd
- Banco de dados distribuído key-value
- Armazena todo o estado do cluster
- Backup crítico para disaster recovery
- Usa algoritmo de consenso Raft

#### Scheduler (kube-scheduler)
- Decide em qual nó executar novos pods
- Considera recursos disponíveis, políticas e constraints
- Algoritmos de scheduling configuráveis
- Não executa pods, apenas agenda

#### Controller Manager (kube-controller-manager)
- Executa controladores que regulam o estado do cluster
- **Node Controller**: Monitora saúde dos nós
- **Replication Controller**: Mantém número correto de pods
- **Endpoints Controller**: Gerencia objetos Endpoints
- **Service Account Controller**: Cria contas padrão

#### Cloud Controller Manager
- Interage com APIs de provedores cloud
- Gerencia load balancers, volumes e redes
- Separa lógica específica do cloud

## Worker Nodes

Os **Worker Nodes** executam as aplicações containerizadas e fornecem o ambiente de runtime.

### Componentes dos Worker Nodes

#### kubelet
- Agente principal que roda em cada nó
- Comunica com o API Server
- Gerencia pods e containers no nó
- Executa health checks (liveness/readiness probes)
- Reporta status do nó e pods

#### kube-proxy
- Proxy de rede que roda em cada nó
- Implementa regras de rede para Services
- Balanceamento de carga entre pods
- Suporta iptables, IPVS ou userspace

#### Container Runtime
- Software que executa containers
- **containerd**: Runtime padrão
- **CRI-O**: Runtime otimizado para Kubernetes
- **Docker**: Suportado via dockershim (deprecated)

#### Addons Opcionais
- **DNS**: CoreDNS para service discovery
- **Dashboard**: Interface web
- **Monitoring**: Prometheus, Grafana
- **Logging**: Fluentd, Elasticsearch

## Arquitetura Completa

```
┌─────────────────────────────────────────────┐
│                Control Plane                │
│  ┌──────────────┐  ┌─────────────────────┐  │
│  │  API Server  │  │     Scheduler       │  │
│  └──────────────┘  └─────────────────────┘  │
│  ┌──────────────┐  ┌─────────────────────┐  │
│  │     etcd     │  │ Controller Manager  │  │
│  └──────────────┘  └─────────────────────┘  │
└─────────────────────────────────────────────┘
                       │
        ┌──────────────┼──────────────┐
        │              │              │
┌───────▼──────┐ ┌─────▼──────┐ ┌─────▼──────┐
│ Worker Node 1│ │Worker Node 2│ │Worker Node 3│
│ ┌──────────┐ │ │ ┌──────────┐│ │ ┌──────────┐│
│ │ kubelet  │ │ │ │ kubelet  ││ │ │ kubelet  ││
│ └──────────┘ │ │ └──────────┘│ │ └──────────┘│
│ ┌──────────┐ │ │ ┌──────────┐│ │ ┌──────────┐│
│ │kube-proxy│ │ │ │kube-proxy││ │ │kube-proxy││
│ └──────────┘ │ │ └──────────┘│ │ └──────────┘│
│ ┌──────────┐ │ │ ┌──────────┐│ │ ┌──────────┐│
│ │Container │ │ │ │Container ││ │ │Container ││
│ │ Runtime  │ │ │ │ Runtime  ││ │ │ Runtime  ││
│ └──────────┘ │ │ └──────────┘│ │ └──────────┘│
│              │ │             │ │             │
│   Pod Pod    │ │  Pod Pod    │ │  Pod Pod    │
└──────────────┘ └─────────────┘ └─────────────┘
```

## Comunicação entre Componentes

### Control Plane → Workers
- API Server envia instruções via kubelet
- Scheduler agenda pods em nós específicos
- Controllers monitoram e ajustam estado

### Workers → Control Plane
- kubelet reporta status de nós e pods
- Métricas e logs são coletados
- Health checks são enviados

### Entre Workers
- kube-proxy gerencia comunicação entre pods
- Service discovery via DNS
- Network policies aplicadas

## Alta Disponibilidade

### Control Plane HA
- Múltiplas instâncias do API Server
- etcd em cluster (3, 5 ou 7 nós)
- Load balancer para API Server
- Scheduler e Controller Manager em active/standby

### Worker Nodes HA
- Distribuição de pods entre múltiplos nós
- Node affinity e anti-affinity
- Taints e tolerations
- Pod disruption budgets

## Responsabilidades

| Componente | Responsabilidade |
|------------|------------------|
| Control Plane | Decisões globais, estado desejado |
| Worker Nodes | Execução de workloads |
| API Server | Interface e validação |
| etcd | Persistência de estado |
| Scheduler | Placement de pods |
| kubelet | Gerenciamento local de pods |
| kube-proxy | Networking e load balancing |
