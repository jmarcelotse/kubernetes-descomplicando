# Kubernetes Descomplicando

Guia completo e prático para aprender Kubernetes do básico ao avançado.

## 📚 Conteúdo

### Fundamentos
- [O que é Container](o-que-e-container.md)
- [O que é Container Engine](o-que-e-container-engine.md)
- [O que é Container Runtime](o-que-e-container-runtime.md)
- [O que é OCI](o-que-e-oci.md)
- [O que é Kubernetes](o-que-e-kubernetes.md)
- [O que é Pod](o-que-e-pod.md)

### Arquitetura
- [Workers e Control Plane](workers-e-control-plane.md)
- [Componentes Workers Kubernetes](componentes-workers-kubernetes.md)
- [Diagramas Kubernetes](diagramas-kubernetes.md)
- [Portas Kubernetes](portas-kubernetes.md)

### Ferramentas
- [Entendendo e Instalando kubectl](entendendo-instalando-kubectl.md)
- [Primeiros Passos com kubectl](primeiros-passos-kubectl.md)
- [Conhecendo YAML e kubectl dry-run](conhecendo-yaml-kubectl-dry-run.md)
- [kubectl get e describe pods](kubectl-get-describe-pods.md)
- [kubectl attach e kubectl exec](kubectl-attach-exec.md)

### Ambiente Prático
- [Criando Primeiro Cluster com kind](criando-primeiro-cluster-kind.md)
- [Introdução a Pods, Deployments e Services](introducao-pods-deployments-services.md)
- [Pod Multi-Container](pod-multi-container.md)
- [Limitando Recursos CPU e Memória](limitando-recursos-cpu-memoria.md)
- [Configurando Volume EmptyDir](configurando-volume-emptydir.md)

### Deployments
- [O que é Deployment](o-que-e-deployment.md)
- [Diagramas de Deployment](diagramas-deployment.md)
- [Primeiros Passos com Deployment](primeiros-passos-deployment.md)
- [Criando Deployment via Manifesto](criando-deployment-manifesto.md)
- [Diagramas: Deployment via Manifesto](diagramas-deployment-manifesto.md)
- [Criando Múltiplos Deployments](criando-multiplos-deployments.md)
- [Diagramas: Múltiplos Deployments](diagramas-multiplos-deployments.md)
- [Atualizando Deployments](atualizando-deployment.md)
- [Diagramas: Atualização de Deployments](diagramas-atualizacao-deployment.md)

## 🚀 Início Rápido

### Pré-requisitos
- Docker instalado
- kubectl instalado
- kind instalado

### Criando seu primeiro cluster
```bash
# Criar cluster com kind
kind create cluster --config k8s/kind/kind-config.yaml --name meu-cluster

# Verificar cluster
kubectl cluster-info
kubectl get nodes
```

### Primeiro pod
```bash
# Criar pod nginx
kubectl run nginx --image=nginx --port=80

# Verificar pod
kubectl get pods
kubectl describe pod nginx
```

## 📁 Estrutura do Projeto

```
kubernetes-descomplicando/
├── README.md
├── k8s/
│   ├── kind/
│   │   └── kind-config.yaml
│   ├── pod/
│   │   ├── pod.yaml                    # Pod nginx básico
│   │   ├── pod1.yaml                   # Pod multi-container (nginx + apache)
│   │   ├── pod2.yaml                   # Pod ubuntu com limitação de recursos
│   │   ├── pod3.yaml                   # Pod ubuntu com volume EmptyDir
│   │   ├── multi-container-pod.yaml    # Pod com nginx + log collector
│   │   ├── resource-demo.yaml          # Pod com nginx + stress test
│   │   └── emptydir-demo.yaml          # Pod com volume EmptyDir compartilhado
│   └── deployment/
│       ├── deployment.yaml             # Deployment nginx básico
│       ├── web-app-deployment.yaml     # Deployment com recursos e health checks
│       ├── rolling-update-demo.yaml    # Deployment com estratégia rolling update
│       ├── primeiro-deployment.yaml    # Primeiro deployment via manifesto
│       ├── deployment-completo.yaml    # Deployment com todas as configurações
│       ├── api-backend.yaml            # Deployment de API backend
│       ├── frontend-web.yaml           # Deployment frontend web
│       ├── api-service.yaml            # Deployment API service
│       └── database-postgres.yaml      # Deployment PostgreSQL
├── *.md (documentação)
└── .gitignore
```

## 🎯 Objetivos

- Desmistificar o Kubernetes
- Aprendizado prático e hands-on
- Exemplos reais e aplicáveis
- Progressão do básico ao avançado

## 🤝 Contribuição

Contribuições são bem-vindas! Sinta-se à vontade para:
- Reportar bugs
- Sugerir melhorias
- Adicionar exemplos
- Corrigir documentação

## 📄 Licença

Este projeto é open source e está disponível sob a licença MIT.
