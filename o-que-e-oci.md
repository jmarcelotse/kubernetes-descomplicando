# O que é o OCI (Open Container Initiative)?

O **OCI (Open Container Initiative)** é um projeto da Linux Foundation que estabelece padrões abertos para formatos de containers e runtimes, garantindo interoperabilidade entre diferentes tecnologias de containerização.

## Objetivo Principal

Criar especificações abertas e padronizadas para:
- Formato de imagens de containers
- Runtime de containers
- Distribuição de imagens

## Especificações OCI

### 1. Runtime Specification (runtime-spec)
Define como executar um container:
- Configuração do container (config.json)
- Ciclo de vida do container
- Operações do runtime (create, start, kill, delete)
- Hooks de execução

### 2. Image Specification (image-spec)
Define o formato de imagens:
- Estrutura de layers
- Manifesto da imagem
- Configuração da imagem
- Índice de imagens

### 3. Distribution Specification (distribution-spec)
Define como distribuir imagens:
- APIs para registries
- Protocolo de push/pull
- Autenticação e autorização

## Implementações Compatíveis

### Runtimes OCI
- **runc**: Implementação de referência
- **crun**: Runtime em C
- **kata-runtime**: Containers em micro-VMs
- **gVisor (runsc)**: Sandbox de segurança

### Engines que Suportam OCI
- Docker Engine
- Podman
- containerd
- CRI-O

### Registries OCI
- Docker Hub
- Amazon ECR
- Google Container Registry
- Harbor
- Quay.io

## Benefícios do OCI

### Interoperabilidade
- Imagens funcionam em qualquer runtime compatível
- Evita vendor lock-in
- Facilita migração entre plataformas

### Padronização
- Especificações claras e abertas
- Desenvolvimento colaborativo
- Governança transparente

### Inovação
- Base sólida para novas tecnologias
- Compatibilidade garantida
- Ecossistema robusto

## Estrutura de uma Imagem OCI

```
┌─────────────────────────────────────────────────────────────┐
│                    OCI IMAGE LAYOUT                         │
├─────────────────────────────────────────────────────────────┤
│  image/                                                     │
│  ├── blobs/                                                 │
│  │   └── sha256/                                            │
│  │       ├── abc123... ◄─── Config (JSON metadata)         │
│  │       ├── def456... ◄─── Layer 1 (tar.gz)               │
│  │       ├── ghi789... ◄─── Layer 2 (tar.gz)               │
│  │       └── jkl012... ◄─── Layer 3 (tar.gz)               │
│  ├── index.json     ◄─── Image Index                       │
│  └── oci-layout     ◄─── Layout Version                    │
└─────────────────────────────────────────────────────────────┘
```

### Fluxo de Build e Distribuição OCI

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Source    │───▶│   Build     │───▶│   Registry  │───▶│   Runtime   │
│    Code     │    │   Engine    │    │             │    │             │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
       │                   │                   │                   │
       ▼                   ▼                   ▼                   ▼
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│ Dockerfile  │    │ OCI Image   │    │ Image Store │    │ Container   │
│ Build Ctx   │    │ Layers      │    │ Manifest    │    │ Process     │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
```

## Exemplo de Manifest OCI

```json
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.manifest.v1+json",
  "config": {
    "mediaType": "application/vnd.oci.image.config.v1+json",
    "size": 1234,
    "digest": "sha256:abc123..."
  },
  "layers": [
    {
      "mediaType": "application/vnd.oci.image.layer.v1.tar+gzip",
      "size": 5678,
      "digest": "sha256:def456..."
    }
  ]
}
```

## Governança

### Membros Fundadores
- Docker
- CoreOS
- Red Hat
- Intel
- Google
- Microsoft

### Processo de Desenvolvimento
- Propostas via GitHub Issues
- Discussões públicas
- Consenso da comunidade
- Implementações de referência
