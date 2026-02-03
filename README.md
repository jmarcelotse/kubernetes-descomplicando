# Kubernetes Descomplicando

Guia completo e prático para aprender Kubernetes do básico ao avançado.

## 📚 Conteúdo

### Fundamentos
- [O que é Container](o-que-e-container.md)
- [O que é Container Engine](o-que-e-container-engine.md)
- [O que é Container Runtime](o-que-e-container-runtime.md)
- [O que é OCI](o-que-e-oci.md)
- [O que é Kubernetes](o-que-e-kubernetes.md)

### Arquitetura
- [Workers e Control Plane](workers-e-control-plane.md)
- [Componentes Workers Kubernetes](componentes-workers-kubernetes.md)
- [Diagramas Kubernetes](diagramas-kubernetes.md)
- [Portas Kubernetes](portas-kubernetes.md)

### Ferramentas
- [Entendendo e Instalando kubectl](entendendo-instalando-kubectl.md)
- [Primeiros Passos com kubectl](primeiros-passos-kubectl.md)
- [Conhecendo YAML e kubectl dry-run](conhecendo-yaml-kubectl-dry-run.md)

### Ambiente Prático
- [Criando Primeiro Cluster com kind](criando-primeiro-cluster-kind.md)
- [Introdução a Pods, Deployments e Services](introducao-pods-deployments-services.md)

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
│   └── kind/
│       └── kind-config.yaml
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
