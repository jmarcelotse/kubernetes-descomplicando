# O que é um Container Engine?

Um **Container Engine** é o software responsável por gerenciar o ciclo de vida completo dos containers, desde a criação até a execução e remoção.

## Função Principal

O container engine atua como uma camada de abstração entre as aplicações containerizadas e o sistema operacional, fornecendo APIs e ferramentas para:

- Criar containers a partir de imagens
- Iniciar, parar e remover containers
- Gerenciar recursos do sistema
- Implementar isolamento e segurança

## Componentes Principais

### Runtime de Container
- **High-level runtime**: Docker, Podman, containerd
- **Low-level runtime**: runc, crun, kata-runtime

### Gerenciamento de Imagens
- Pull/push de imagens de registries
- Armazenamento local de imagens
- Cache e otimização de layers

### Networking
- Criação de redes virtuais
- Isolamento de tráfego
- Port mapping e DNS

## Container Engines Populares

### Docker Engine
- Mais popular e amplamente adotado
- Arquitetura cliente-servidor com daemon
- Suporte completo ao ecossistema Docker

### Podman
- Alternativa sem daemon (daemonless)
- Compatível com comandos Docker
- Foco em segurança e rootless containers

### containerd
- Runtime usado pelo Kubernetes
- Mais leve e focado em orquestração
- Graduado pela CNCF

### CRI-O
- Implementação do Container Runtime Interface
- Otimizado especificamente para Kubernetes
- Suporte apenas a imagens OCI

## Arquitetura Típica

```
┌─────────────────────────────────────────────────────────────┐
│                     USER/CLIENT                             │
└─────────────────────────┬───────────────────────────────────┘
                          │
┌─────────────────────────▼───────────────────────────────────┐
│                 CONTAINER ENGINE                            │
│  ┌─────────────────────────────────────────────────────────┐│
│  │              API & CLI Interface                        ││
│  └─────────────────────────────────────────────────────────┘│
│  ┌─────────────────────────────────────────────────────────┐│
│  │               Image Management                          ││
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐││
│  │  │Image Storage│ │Image Builder│ │Registry Integration │││
│  │  └─────────────┘ └─────────────┘ └─────────────────────┘││
│  └─────────────────────────────────────────────────────────┘│
│  ┌─────────────────────────────────────────────────────────┐│
│  │              Container Management                       ││
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐││
│  │  │ Lifecycle   │ │ Networking  │ │    Monitoring       │││
│  │  └─────────────┘ └─────────────┘ └─────────────────────┘││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────┬───────────────────────────────────┘
                          │
┌─────────────────────────▼───────────────────────────────────┐
│                 CONTAINER RUNTIME                           │
│  ┌─────────────────────────────────────────────────────────┐│
│  │              High-Level Runtime                         ││
│  │            (containerd, CRI-O)                          ││
│  └─────────────────────────────────────────────────────────┘│
│  ┌─────────────────────────────────────────────────────────┐│
│  │               Low-Level Runtime                         ││
│  │                 (runc, crun)                            ││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────┬───────────────────────────────────┘
                          │
┌─────────────────────────▼───────────────────────────────────┐
│                 OPERATING SYSTEM                            │
│  ┌─────────────────────────────────────────────────────────┐│
│  │    Kernel (Namespaces, Cgroups, Capabilities)          ││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

## Diferença: Engine vs Runtime

| Container Engine | Container Runtime |
|------------------|-------------------|
| Interface de alto nível | Execução de baixo nível |
| APIs e ferramentas | Gerencia processos |
| Gerencia imagens | Implementa isolamento |
| Docker, Podman | runc, crun |
