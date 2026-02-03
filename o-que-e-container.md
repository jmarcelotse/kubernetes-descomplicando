# O que é um Container?

Um **container** é uma unidade de software que empacota código e todas as suas dependências para que a aplicação execute de forma rápida e confiável em diferentes ambientes computacionais.

## Características Principais

- **Isolamento**: Cada container roda isoladamente do sistema host e de outros containers
- **Portabilidade**: Funciona consistentemente em qualquer ambiente que suporte containers
- **Leveza**: Compartilha o kernel do sistema operacional, sendo mais eficiente que VMs
- **Imutabilidade**: Uma vez criado, o container não muda (infraestrutura como código)

## Como Funciona

Os containers utilizam recursos do kernel Linux como:
- **Namespaces**: Isolam processos, rede, sistema de arquivos
- **Cgroups**: Limitam e controlam recursos (CPU, memória, I/O)
- **Union File Systems**: Permitem camadas de sistema de arquivos

## Vantagens

- Deployment mais rápido
- Uso eficiente de recursos
- Escalabilidade horizontal
- Consistência entre ambientes (dev, test, prod)
- Facilita DevOps e CI/CD

## Diferença entre Container e VM

| Container | Máquina Virtual |
|-----------|-----------------|
| Compartilha kernel do host | Kernel próprio |
| Inicialização em segundos | Inicialização em minutos |
| Menor uso de recursos | Maior uso de recursos |
| Isolamento a nível de processo | Isolamento completo de hardware |

### Diagrama: Container vs VM

```
┌─────────────────────────────────────────────────────────────┐
│                        CONTAINERS                           │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐        │
│  │  App A  │  │  App B  │  │  App C  │  │  App D  │        │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘        │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐        │
│  │ Bins/Libs│  │ Bins/Libs│  │ Bins/Libs│  │ Bins/Libs│        │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘        │
├─────────────────────────────────────────────────────────────┤
│                   Container Engine                          │
├─────────────────────────────────────────────────────────────┤
│                    Host OS Kernel                           │
├─────────────────────────────────────────────────────────────┤
│                   Physical Server                           │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                   VIRTUAL MACHINES                          │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐  ┌─────────────────┐                  │
│  │      VM 1       │  │      VM 2       │                  │
│  │  ┌───────────┐  │  │  ┌───────────┐  │                  │
│  │  │   App A   │  │  │  │   App B   │  │                  │
│  │  └───────────┘  │  │  └───────────┘  │                  │
│  │  ┌───────────┐  │  │  ┌───────────┐  │                  │
│  │  │ Bins/Libs │  │  │  │ Bins/Libs │  │                  │
│  │  └───────────┘  │  │  └───────────┘  │                  │
│  │  ┌───────────┐  │  │  ┌───────────┐  │                  │
│  │  │Guest OS   │  │  │  │Guest OS   │  │                  │
│  │  └───────────┘  │  │  └───────────┘  │                  │
│  └─────────────────┘  └─────────────────┘                  │
├─────────────────────────────────────────────────────────────┤
│                     Hypervisor                             │
├─────────────────────────────────────────────────────────────┤
│                    Host OS Kernel                           │
├─────────────────────────────────────────────────────────────┤
│                   Physical Server                           │
└─────────────────────────────────────────────────────────────┘
```

## Tecnologias Populares

- **Docker**: Plataforma mais popular para containers
- **Podman**: Alternativa ao Docker sem daemon
- **containerd**: Runtime de containers usado pelo Kubernetes
- **CRI-O**: Runtime otimizado para Kubernetes
