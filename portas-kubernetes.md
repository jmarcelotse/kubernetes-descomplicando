# Portas TCP e UDP dos Componentes do Kubernetes

Este documento lista todas as portas utilizadas pelos componentes do Kubernetes com diagramas para melhor visualização.

## Diagrama Geral de Portas

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    KUBERNETES CLUSTER PORTS                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                     CONTROL PLANE                                  ││
│  │                                                                     ││
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────────┐ ││
│  │  │ API Server  │  │    etcd     │  │ Scheduler/Controller Mgr    │ ││
│  │  │             │  │             │  │                             │ ││
│  │  │ 6443/tcp ◄──┼──┤ 2379/tcp ◄──┼──┤ 10251/tcp (scheduler)       │ ││
│  │  │ 8080/tcp    │  │ 2380/tcp    │  │ 10252/tcp (controller-mgr)  │ ││
│  │  └─────────────┘  └─────────────┘  └─────────────────────────────┘ ││
│  └─────────────────────────────────────────────────────────────────────┘│
│                                │                                        │
│                                │ API Calls                              │
│                                ▼                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                      WORKER NODES                                  ││
│  │                                                                     ││
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────────┐ ││
│  │  │   kubelet   │  │ kube-proxy  │  │        Pods/Services        │ ││
│  │  │             │  │             │  │                             │ ││
│  │  │ 10250/tcp ◄─┼──┤ 10256/tcp   │  │ 30000-32767/tcp (NodePort)  │ ││
│  │  │ 10255/tcp   │  │             │  │ 53/tcp,udp (DNS)            │ ││
│  │  │ 4194/tcp    │  │             │  │ 80,443/tcp (Ingress)        │ ││
│  │  └─────────────┘  └─────────────┘  └─────────────────────────────┘ ││
│  └─────────────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────────────┘
```

## Control Plane (Master Node)

### API Server
| Porta | Protocolo | Descrição | Usado por |
|-------|-----------|-----------|-----------|
| 6443 | TCP | Kubernetes API Server (HTTPS) | kubectl, kubelet, kube-proxy, scheduler, controller-manager |
| 8080 | TCP | API Server (HTTP) - Inseguro | Deprecated, não usar em produção |

### etcd
| Porta | Protocolo | Descrição | Usado por |
|-------|-----------|-----------|-----------|
| 2379 | TCP | etcd client requests | API Server |
| 2380 | TCP | etcd peer communication | etcd cluster members |

### Scheduler
| Porta | Protocolo | Descrição | Usado por |
|-------|-----------|-----------|-----------|
| 10251 | TCP | kube-scheduler health check | Monitoring, health checks |
| 10259 | TCP | kube-scheduler secure port | Metrics, health checks |

### Controller Manager
| Porta | Protocolo | Descrição | Usado por |
|-------|-----------|-----------|-----------|
| 10252 | TCP | kube-controller-manager health check | Monitoring, health checks |
| 10257 | TCP | kube-controller-manager secure port | Metrics, health checks |

## Worker Nodes

### kubelet
| Porta | Protocolo | Descrição | Usado por |
|-------|-----------|-----------|-----------|
| 10250 | TCP | kubelet API (HTTPS) | API Server, kubectl, monitoring |
| 10255 | TCP | kubelet read-only port (HTTP) | Monitoring (deprecated) |
| 4194 | TCP | cAdvisor (container metrics) | Monitoring systems |

### kube-proxy
| Porta | Protocolo | Descrição | Usado por |
|-------|-----------|-----------|-----------|
| 10256 | TCP | kube-proxy health check | Monitoring, health checks |

## Diagrama de Comunicação entre Componentes

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    COMPONENT COMMUNICATION                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  kubectl ──────────────────────────────────────────────────────────────┐│
│     │                                                                  ││
│     │ 6443/tcp                                                         ││
│     ▼                                                                  ││
│  ┌─────────────┐                                                       ││
│  │ API Server  │                                                       ││
│  │   :6443     │                                                       ││
│  └─────────────┘                                                       ││
│         │                                                              ││
│         │ 2379/tcp                                                     ││
│         ▼                                                              ││
│  ┌─────────────┐    2380/tcp    ┌─────────────┐                       ││
│  │    etcd     │◄──────────────▶│    etcd     │                       ││
│  │   :2379     │                │   :2379     │                       ││
│  └─────────────┘                └─────────────┘                       ││
│         ▲                                                              ││
│         │ 6443/tcp                                                     ││
│         │                                                              ││
│  ┌─────────────┐                ┌─────────────────────────────────────┐││
│  │ Scheduler   │                │      Controller Manager             │││
│  │  :10251     │                │           :10252                    │││
│  └─────────────┘                └─────────────────────────────────────┘││
│         │                                       │                     ││
│         │ 6443/tcp                              │ 6443/tcp            ││
│         └───────────────────┬───────────────────┘                     ││
│                             ▼                                         ││
│                      ┌─────────────┐                                  ││
│                      │   kubelet   │                                  ││
│                      │   :10250    │                                  ││
│                      └─────────────┘                                  ││
│                             │                                         ││
│                             │ CRI                                     ││
│                             ▼                                         ││
│                      ┌─────────────┐                                  ││
│                      │ Container   │                                  ││
│                      │  Runtime    │                                  ││
│                      └─────────────┘                                  ││
└─────────────────────────────────────────────────────────────────────────┘
```
## Networking (CNI) - Diagrama de Portas

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        CNI NETWORKING PORTS                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                        FLANNEL                                     ││
│  │                                                                     ││
│  │  Node A ──────────────────────────────────────────────── Node B    ││
│  │    │                    8285/udp                             │      ││
│  │    │                    8472/udp                             │      ││
│  │    └─────────────────── VXLAN ──────────────────────────────┘      ││
│  └─────────────────────────────────────────────────────────────────────┘│
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                         CALICO                                     ││
│  │                                                                     ││
│  │  Node A ──────────────────────────────────────────────── Node B    ││
│  │    │                     179/tcp (BGP)                       │      ││
│  │    │                    4789/udp (VXLAN)                     │      ││
│  │    │                   51820/udp (WireGuard)                 │      ││
│  │    └─────────────────── Routing ──────────────────────────────┘     ││
│  └─────────────────────────────────────────────────────────────────────┘│
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐│
│  │                       WEAVE NET                                    ││
│  │                                                                     ││
│  │  Node A ──────────────────────────────────────────────── Node B    ││
│  │    │                  6783/tcp,udp (Control)                 │      ││
│  │    │                   6784/udp (Data)                       │      ││
│  │    └─────────────────── Mesh ────────────────────────────────┘      ││
│  └─────────────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────────────┘
```

### Flannel
| Porta | Protocolo | Descrição | Usado por |
|-------|-----------|-----------|-----------|
| 8285 | UDP | Flannel VXLAN | Inter-node communication |
| 8472 | UDP | Flannel VXLAN (default) | Inter-node communication |

### Calico
| Porta | Protocolo | Descrição | Usado por |
|-------|-----------|-----------|-----------|
| 179 | TCP | BGP | Calico nodes |
| 4789 | UDP | VXLAN | Inter-node communication |
| 51820 | UDP | WireGuard | Encrypted communication |
| 51821 | UDP | WireGuard IPv6 | Encrypted communication |

### Weave Net
| Porta | Protocolo | Descrição | Usado por |
|-------|-----------|-----------|-----------|
| 6783 | TCP/UDP | Weave control | Weave peers |
| 6784 | UDP | Weave data | Inter-node communication |

## Services e Addons

### DNS (CoreDNS)
| Porta | Protocolo | Descrição | Usado por |
|-------|-----------|-----------|-----------|
| 53 | TCP/UDP | DNS queries | Pods, Services |
| 9153 | TCP | CoreDNS metrics | Monitoring |

### Ingress Controllers
| Porta | Protocolo | Descrição | Usado por |
|-------|-----------|-----------|-----------|
| 80 | TCP | HTTP traffic | External clients |
| 443 | TCP | HTTPS traffic | External clients |
| 8080 | TCP | Health check/metrics | Monitoring |
| 10254 | TCP | NGINX Ingress metrics | Monitoring |

### NodePort Services
| Porta | Protocolo | Descrição | Usado por |
|-------|-----------|-----------|-----------|
| 30000-32767 | TCP/UDP | NodePort range | External clients |

## Diagrama de Fluxo de Tráfego

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      TRAFFIC FLOW DIAGRAM                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  External Client                                                        │
│         │                                                               │
│         │ 80/443                                                        │
│         ▼                                                               │
│  ┌─────────────┐                                                        │
│  │Load Balancer│                                                        │
│  │             │                                                        │
│  └─────────────┘                                                        │
│         │                                                               │
│         │ 30000-32767                                                   │
│         ▼                                                               │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────────────┐ │
│  │ Worker Node │    │ Worker Node │    │       Worker Node           │ │
│  │             │    │             │    │                             │ │
│  │ ┌─────────┐ │    │ ┌─────────┐ │    │ ┌─────────────────────────┐ │ │
│  │ │kube-proxy│ │    │ │kube-proxy│ │    │ │      kube-proxy         │ │ │
│  │ │:10256   │ │    │ │:10256   │ │    │ │        :10256           │ │ │
│  │ └─────────┘ │    │ └─────────┘ │    │ └─────────────────────────┘ │ │
│  │      │      │    │      │      │    │              │              │ │
│  │      │ iptables  │      │ iptables  │              │ iptables     │ │
│  │      ▼      │    │      ▼      │    │              ▼              │ │
│  │ ┌─────────┐ │    │ ┌─────────┐ │    │ ┌─────────────────────────┐ │ │
│  │ │  Pod A  │ │    │ │  Pod B  │ │    │ │         Pod C           │ │ │
│  │ │  :8080  │ │    │ │  :8080  │ │    │ │         :8080           │ │ │
│  │ └─────────┘ │    │ └─────────┘ │    │ └─────────────────────────┘ │ │
│  └─────────────┘    └─────────────┘    └─────────────────────────────┘ │
│                                                                         │
│  Inter-pod communication via CNI:                                      │
│  Pod A ←──────────── 8285/udp (Flannel) ──────────────→ Pod B          │
│  Pod B ←──────────── 179/tcp (Calico BGP) ─────────────→ Pod C         │
└─────────────────────────────────────────────────────────────────────────┘
```

## Resumo de Portas Críticas

### Sempre Abertas (Essenciais)
```
┌─────────────────────────────────────────────────────────────────────────┐
│                      CRITICAL PORTS                                    │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Control Plane:                                                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────────────┐ │
│  │ 6443/tcp    │  │ 2379/tcp    │  │ 2380/tcp                        │ │
│  │ API Server  │  │ etcd client │  │ etcd peer                       │ │
│  └─────────────┘  └─────────────┘  └─────────────────────────────────┘ │
│                                                                         │
│  Worker Nodes:                                                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────────────┐ │
│  │ 10250/tcp   │  │ 53/tcp,udp  │  │ 30000-32767/tcp                 │ │
│  │ kubelet API │  │ DNS         │  │ NodePort range                  │ │
│  └─────────────┘  └─────────────┘  └─────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
```

## Comandos de Verificação

### Verificar Portas Ativas
```bash
# Verificar portas do Control Plane
netstat -tlnp | grep -E "(6443|2379|2380|10251|10252)"

# Verificar portas dos Workers
netstat -tlnp | grep -E "(10250|10256|53)"

# Verificar conectividade
telnet <master-ip> 6443
nc -zv <worker-ip> 10250
```

### Firewall Configuration
```bash
# Control Plane
iptables -A INPUT -p tcp --dport 6443 -j ACCEPT
iptables -A INPUT -p tcp --dport 2379:2380 -j ACCEPT
iptables -A INPUT -p tcp --dport 10250:10252 -j ACCEPT

# Worker Nodes
iptables -A INPUT -p tcp --dport 10250 -j ACCEPT
iptables -A INPUT -p tcp --dport 30000:32767 -j ACCEPT
iptables -A INPUT -p udp --dport 53 -j ACCEPT
```
