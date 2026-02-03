# O que é um Container Runtime?

Um **Container Runtime** é o componente de baixo nível responsável pela execução real dos containers no sistema operacional, implementando o isolamento e gerenciando os processos containerizados.

## Função Principal

O container runtime é responsável por:

- Executar containers a partir de imagens
- Implementar isolamento usando namespaces e cgroups
- Gerenciar o ciclo de vida dos processos
- Aplicar políticas de segurança
- Configurar networking e storage

## Tipos de Container Runtime

### High-Level Runtime
Fornecem APIs e funcionalidades avançadas:
- **containerd**: Runtime padrão do Kubernetes
- **CRI-O**: Implementação do CRI para Kubernetes
- **Docker Engine**: Runtime completo com daemon

### Low-Level Runtime
Executam containers diretamente:
- **runc**: Implementação de referência da OCI
- **crun**: Runtime em C, mais rápido que runc
- **kata-runtime**: Containers em micro-VMs
- **gVisor (runsc)**: Sandbox de segurança

## Padrões e Especificações

### OCI (Open Container Initiative)
- **Runtime Specification**: Define como executar containers
- **Image Specification**: Define formato de imagens
- **Distribution Specification**: Define como distribuir imagens

### CRI (Container Runtime Interface)
- Interface padrão do Kubernetes
- Permite trocar runtimes sem modificar Kubernetes
- Implementado por containerd, CRI-O

## Arquitetura em Camadas

```
┌─────────────────────────────────────────────────────────────┐
│                      KUBERNETES                             │
│  ┌─────────────────────────────────────────────────────────┐│
│  │                    kubelet                              ││
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐││
│  │  │Pod Manager  │ │Node Manager │ │   Status Manager    │││
│  │  └─────────────┘ └─────────────┘ └─────────────────────┘││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────┬───────────────────────────────────┘
                          │ CRI (Container Runtime Interface)
┌─────────────────────────▼───────────────────────────────────┐
│                 HIGH-LEVEL RUNTIME                          │
│  ┌─────────────────────────────────────────────────────────┐│
│  │              containerd / CRI-O                         ││
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐││
│  │  │Image Service│ │Runtime Svc  │ │   gRPC Server       │││
│  │  └─────────────┘ └─────────────┘ └─────────────────────┘││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────┬───────────────────────────────────┘
                          │ OCI Runtime Spec
┌─────────────────────────▼───────────────────────────────────┐
│                  LOW-LEVEL RUNTIME                          │
│  ┌─────────────────────────────────────────────────────────┐│
│  │                runc / crun / kata                       ││
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐││
│  │  │ Namespace   │ │   Cgroups   │ │    Capabilities     │││
│  │  │ Management  │ │ Management  │ │    Management       │││
│  │  └─────────────┘ └─────────────┘ └─────────────────────┘││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────┬───────────────────────────────────┘
                          │
┌─────────────────────────▼───────────────────────────────────┐
│                 OPERATING SYSTEM                            │
│  ┌─────────────────────────────────────────────────────────┐│
│  │              Linux Kernel                               ││
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐││
│  │  │ Namespaces  │ │   Cgroups   │ │   Capabilities      │││
│  │  └─────────────┘ └─────────────┘ └─────────────────────┘││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

### Fluxo de Execução de Container

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   kubectl   │───▶│   kubelet   │───▶│ containerd  │───▶│    runc     │
│   create    │    │             │    │             │    │             │
│    pod      │    │             │    │             │    │             │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
                           │                   │                   │
                           ▼                   ▼                   ▼
                   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
                   │ Pod Spec    │    │ OCI Bundle  │    │ Container   │
                   │ Validation  │    │ Creation    │    │ Process     │
                   └─────────────┘    └─────────────┘    └─────────────┘
```

## Principais Runtimes

### runc
- Implementação de referência da OCI
- Usado pela maioria dos high-level runtimes
- Escrito em Go

### crun
- Alternativa ao runc escrita em C
- Menor uso de memória e inicialização mais rápida
- Compatível com OCI

### kata-runtime
- Executa containers em micro-VMs
- Maior isolamento de segurança
- Overhead adicional de performance

### gVisor (runsc)
- Sandbox de segurança para containers
- Intercepta system calls
- Reduz superfície de ataque

## Escolhendo um Runtime

| Cenário | Runtime Recomendado |
|---------|-------------------|
| Uso geral | runc |
| Performance crítica | crun |
| Segurança máxima | kata-runtime, gVisor |
| Kubernetes | containerd + runc |
| Workloads não confiáveis | gVisor |
