# Componentes dos Workers do Kubernetes

Os **Worker Nodes** são responsáveis por executar as aplicações containerizadas no cluster Kubernetes. Cada worker contém componentes essenciais para o funcionamento dos pods.

## Componentes Principais

### 1. kubelet

O **kubelet** é o agente principal que roda em cada worker node.

#### Responsabilidades
- Comunica com o API Server do Control Plane
- Gerencia o ciclo de vida dos pods no nó
- Executa containers conforme especificações dos pods
- Monitora saúde dos containers (health checks)
- Reporta status do nó e pods para o Control Plane
- Gerencia volumes e secrets

#### Funcionalidades
- **Pod Lifecycle**: Create, start, stop, delete pods
- **Health Probes**: Liveness, readiness e startup probes
- **Resource Management**: CPU, memória, storage
- **Volume Management**: Mount/unmount de volumes
- **Image Management**: Pull de imagens de containers

### 2. kube-proxy

O **kube-proxy** é responsável pelo networking e load balancing.

#### Responsabilidades
- Implementa regras de rede para Services
- Balanceamento de carga entre pods
- Roteamento de tráfego interno do cluster
- Mantém regras de iptables/IPVS
- Service discovery

#### Modos de Operação
- **iptables**: Modo padrão, usa regras iptables
- **IPVS**: Melhor performance para clusters grandes
- **userspace**: Modo legado (não recomendado)

### 3. Container Runtime

O **Container Runtime** executa os containers nos pods.

#### Runtimes Suportados
- **containerd**: Runtime padrão e recomendado
- **CRI-O**: Runtime otimizado para Kubernetes
- **Docker**: Suportado via dockershim (deprecated no K8s 1.24+)

#### Interface CRI
- **Container Runtime Interface**: Padrão para comunicação
- Permite trocar runtimes sem modificar Kubernetes
- Operações: RunPodSandbox, CreateContainer, StartContainer

## Componentes Opcionais

### 4. DNS (CoreDNS)

#### Função
- Service discovery interno do cluster
- Resolução de nomes de Services e pods
- Configuração via ConfigMaps

### 5. CNI (Container Network Interface)

#### Plugins Populares
- **Flannel**: Simples overlay network
- **Calico**: Network policies e BGP
- **Weave**: Mesh networking
- **Cilium**: eBPF-based networking

### 6. CSI (Container Storage Interface)

#### Drivers de Storage
- **AWS EBS**: Volumes na AWS
- **GCE PD**: Volumes no Google Cloud
- **Azure Disk**: Volumes no Azure
- **Ceph**: Storage distribuído

## Arquitetura do Worker Node

```
┌─────────────────────────────────────────┐
│              Worker Node                │
│                                         │
│  ┌─────────────────────────────────────┐│
│  │            kubelet                  ││
│  │  ┌─────────────┐ ┌───────────────┐ ││
│  │  │Pod Manager  │ │Volume Manager │ ││
│  │  └─────────────┘ └───────────────┘ ││
│  │  ┌─────────────┐ ┌───────────────┐ ││
│  │  │Image Manager│ │Status Manager │ ││
│  │  └─────────────┘ └───────────────┘ ││
│  └─────────────────────────────────────┘│
│                                         │
│  ┌─────────────────────────────────────┐│
│  │           kube-proxy                ││
│  │  ┌─────────────┐ ┌───────────────┐ ││
│  │  │Service Proxy│ │Load Balancer  │ ││
│  │  └─────────────┘ └───────────────┘ ││
│  └─────────────────────────────────────┘│
│                                         │
│  ┌─────────────────────────────────────┐│
│  │        Container Runtime            ││
│  │  ┌─────────────┐ ┌───────────────┐ ││
│  │  │containerd   │ │     runc      │ ││
│  │  └─────────────┘ └───────────────┘ ││
│  └─────────────────────────────────────┘│
│                                         │
│  ┌─────────────────────────────────────┐│
│  │             Addons                  ││
│  │  ┌─────────┐ ┌─────────┐ ┌────────┐││
│  │  │CoreDNS  │ │CNI Plugin│ │CSI     │││
│  │  └─────────┘ └─────────┘ └────────┘││
│  └─────────────────────────────────────┘│
└─────────────────────────────────────────┘
```

## Fluxo de Execução

### 1. Criação de Pod
```
API Server → kubelet → Container Runtime → Pod Running
```

### 2. Networking
```
Service Request → kube-proxy → iptables/IPVS → Target Pod
```

### 3. Storage
```
Pod Spec → kubelet → CSI Driver → Volume Mount
```

## Configuração dos Componentes

### kubelet Configuration
```yaml
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
address: 0.0.0.0
port: 10250
cgroupDriver: systemd
containerRuntimeEndpoint: unix:///var/run/containerd/containerd.sock
```

### kube-proxy Configuration
```yaml
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: "iptables"
clusterCIDR: "10.244.0.0/16"
```

## Monitoramento e Logs

### Métricas Importantes
- CPU e memória do nó
- Número de pods em execução
- Status dos containers
- Latência de rede

### Logs dos Componentes
- `/var/log/kubelet.log`
- `/var/log/kube-proxy.log`
- `journalctl -u kubelet`
- `journalctl -u kube-proxy`

## Troubleshooting Comum

| Problema | Componente | Solução |
|----------|------------|---------|
| Pod não inicia | kubelet | Verificar logs e recursos |
| Rede não funciona | kube-proxy | Verificar regras iptables |
| Container não roda | Container Runtime | Verificar runtime status |
| DNS não resolve | CoreDNS | Verificar configuração DNS |
