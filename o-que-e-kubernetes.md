# O que é o Kubernetes?

O **Kubernetes** (também conhecido como K8s) é uma plataforma de orquestração de containers open-source que automatiza o deployment, scaling e gerenciamento de aplicações containerizadas.

## Origem e História

- Desenvolvido originalmente pelo Google (baseado no sistema interno Borg)
- Open-sourced em 2014
- Doado para a CNCF (Cloud Native Computing Foundation) em 2015
- Tornou-se o padrão de facto para orquestração de containers

## Principais Funcionalidades

### Orquestração de Containers
- Deploy automático de aplicações
- Scaling horizontal e vertical
- Rolling updates e rollbacks
- Health checks e auto-healing

### Gerenciamento de Recursos
- Scheduling inteligente de workloads
- Balanceamento de carga
- Gerenciamento de storage
- Configuração de rede

### Alta Disponibilidade
- Distribuição de aplicações em múltiplos nós
- Recuperação automática de falhas
- Backup e disaster recovery
- Multi-region deployment

## Arquitetura do Kubernetes

### Control Plane (Master)
- **API Server**: Interface principal do cluster
- **etcd**: Banco de dados distribuído
- **Scheduler**: Decide onde executar pods
- **Controller Manager**: Gerencia controladores

### Worker Nodes
- **kubelet**: Agente que executa containers
- **kube-proxy**: Gerencia networking
- **Container Runtime**: Docker, containerd, CRI-O

```
┌─────────────────────────────────────────────────────────────┐
│                     CONTROL PLANE                          │
│  ┌─────────────────────────────────────────────────────────┐│
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐││
│  │  │ API Server  │ │  Scheduler  │ │ Controller Manager  │││
│  │  └─────────────┘ └─────────────┘ └─────────────────────┘││
│  │  ┌─────────────────────────────────────────────────────┐││
│  │  │                    etcd                             │││
│  │  │            (Distributed Database)                  │││
│  │  └─────────────────────────────────────────────────────┘││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────┬───────────────────────────────────┘
                          │ API Calls
        ┌─────────────────┼─────────────────┐
        │                 │                 │
┌───────▼──────┐ ┌────────▼─────────┐ ┌─────▼──────┐
│ Worker Node 1│ │  Worker Node 2   │ │Worker Node 3│
│ ┌──────────┐ │ │  ┌──────────────┐│ │ ┌──────────┐│
│ │ kubelet  │ │ │  │   kubelet    ││ │ │ kubelet  ││
│ └──────────┘ │ │  └──────────────┘│ │ └──────────┘│
│ ┌──────────┐ │ │  ┌──────────────┐│ │ ┌──────────┐│
│ │kube-proxy│ │ │  │  kube-proxy  ││ │ │kube-proxy││
│ └──────────┘ │ │  └──────────────┘│ │ └──────────┘│
│ ┌──────────┐ │ │  ┌──────────────┐│ │ ┌──────────┐│
│ │Container │ │ │  │  Container   ││ │ │Container ││
│ │ Runtime  │ │ │  │   Runtime    ││ │ │ Runtime  ││
│ └──────────┘ │ │  └──────────────┘│ │ └──────────┘│
│              │ │                  │ │             │
│ ┌──────────┐ │ │  ┌──────────────┐│ │ ┌──────────┐│
│ │   Pods   │ │ │  │    Pods      ││ │ │   Pods   ││
│ └──────────┘ │ │  └──────────────┘│ │ └──────────┘│
└──────────────┘ └──────────────────┘ └─────────────┘
```

### Fluxo de Deploy de Aplicação

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   kubectl   │───▶│ API Server  │───▶│  Scheduler  │───▶│   kubelet   │
│   apply     │    │             │    │             │    │             │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
       │                   │                   │                   │
       ▼                   ▼                   ▼                   ▼
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│ YAML Spec   │    │ Validation  │    │ Node        │    │ Pod         │
│ Deployment  │    │ Storage     │    │ Selection   │    │ Creation    │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
```

## Objetos Principais do Kubernetes

### Pod
- Menor unidade deployável
- Contém um ou mais containers
- Compartilha rede e storage

### Deployment
- Gerencia ReplicaSets
- Rolling updates
- Rollback automático

### Service
- Abstração para acesso a pods
- Load balancing
- Service discovery

### ConfigMap e Secret
- Gerenciamento de configurações
- Separação de código e config
- Dados sensíveis (secrets)

### Namespace
- Isolamento lógico de recursos
- Multi-tenancy
- Organização de ambientes

## Benefícios do Kubernetes

### Escalabilidade
- Auto-scaling baseado em métricas
- Scaling horizontal de aplicações
- Gerenciamento eficiente de recursos

### Portabilidade
- Funciona em qualquer cloud ou on-premises
- Abstração da infraestrutura
- Evita vendor lock-in

### Automação
- Self-healing de aplicações
- Deployment automatizado
- Gerenciamento declarativo

### Produtividade
- Acelera time-to-market
- Facilita DevOps e CI/CD
- Reduz overhead operacional

## Casos de Uso

- **Microserviços**: Orquestração de arquiteturas distribuídas
- **CI/CD**: Pipelines de deployment automatizado
- **Multi-cloud**: Aplicações híbridas e multi-cloud
- **Edge Computing**: Deployment em edge locations
- **Machine Learning**: Orquestração de workloads de ML

## Ecossistema CNCF

O Kubernetes faz parte de um ecossistema maior:
- **Helm**: Gerenciador de pacotes
- **Istio**: Service mesh
- **Prometheus**: Monitoramento
- **Fluentd**: Logging
- **Harbor**: Registry de containers
