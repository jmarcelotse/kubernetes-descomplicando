# Primeiros Passos com Deployments

Guia prático para criar e gerenciar seus primeiros Deployments no Kubernetes.

## 🎯 Objetivos

Ao final deste guia, você será capaz de:
- Criar Deployments usando kubectl e YAML
- Gerenciar réplicas e atualizações
- Monitorar e fazer troubleshooting
- Realizar rollbacks quando necessário

## 📋 Pré-requisitos

- Cluster Kubernetes funcionando (kind, minikube, etc.)
- kubectl configurado
- Conhecimento básico de [Pods](o-que-e-pod.md)

## 🚀 Criando seu Primeiro Deployment

### Método 1: Usando kubectl create

```bash
# Criar deployment simples
kubectl create deployment nginx-app --image=nginx:1.21 --replicas=3

# Verificar criação
kubectl get deployments
kubectl get pods
```

### Método 2: Usando arquivo YAML

Crie o arquivo `primeiro-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-app
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
```

```bash
# Aplicar o deployment
kubectl apply -f primeiro-deployment.yaml

# Verificar status
kubectl get deployments
kubectl describe deployment nginx-app
```

## 📊 Monitorando o Deployment

### Comandos Básicos de Monitoramento

```bash
# Status geral
kubectl get deployments

# Detalhes completos
kubectl describe deployment nginx-app

# Pods do deployment
kubectl get pods -l app=nginx

# ReplicaSets
kubectl get rs

# Logs dos pods
kubectl logs -l app=nginx --tail=10
```

### Saída Esperada

```bash
$ kubectl get deployments
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
nginx-app   3/3     3            3           2m

$ kubectl get pods -l app=nginx
NAME                         READY   STATUS    RESTARTS   AGE
nginx-app-7d5b8c8f9d-abc12   1/1     Running   0          2m
nginx-app-7d5b8c8f9d-def34   1/1     Running   0          2m
nginx-app-7d5b8c8f9d-ghi56   1/1     Running   0          2m
```

## 🔄 Gerenciando Réplicas

### Escalonamento Manual

```bash
# Aumentar para 5 réplicas
kubectl scale deployment nginx-app --replicas=5

# Verificar escalonamento
kubectl get pods -l app=nginx

# Diminuir para 2 réplicas
kubectl scale deployment nginx-app --replicas=2
```

### Escalonamento via YAML

```bash
# Editar deployment
kubectl edit deployment nginx-app

# Ou atualizar arquivo e aplicar
kubectl apply -f primeiro-deployment.yaml
```

## 🔄 Atualizando o Deployment

### Atualização de Imagem

```bash
# Atualizar versão do nginx
kubectl set image deployment/nginx-app nginx=nginx:1.22

# Acompanhar o rollout
kubectl rollout status deployment/nginx-app

# Ver histórico
kubectl rollout history deployment/nginx-app
```

### Processo de Rolling Update

```bash
# Durante a atualização, observe:
kubectl get pods -l app=nginx -w

# Você verá pods sendo criados e terminados gradualmente
```

## 🔙 Rollback

### Rollback Simples

```bash
# Voltar para versão anterior
kubectl rollout undo deployment/nginx-app

# Verificar status
kubectl rollout status deployment/nginx-app
```

### Rollback para Versão Específica

```bash
# Ver histórico detalhado
kubectl rollout history deployment/nginx-app --revision=1

# Rollback para revisão específica
kubectl rollout undo deployment/nginx-app --to-revision=1
```

## 🛠️ Exercícios Práticos

### Exercício 1: Deployment Básico

1. Crie um deployment do Apache:
```bash
kubectl create deployment apache-app --image=httpd:2.4 --replicas=2
```

2. Verifique se os pods estão rodando
3. Escale para 4 réplicas
4. Verifique novamente

### Exercício 2: Atualização e Rollback

1. Atualize a imagem do Apache:
```bash
kubectl set image deployment/apache-app httpd=httpd:alpine
```

2. Monitore o rollout
3. Faça rollback para a versão anterior
4. Verifique o histórico de revisões

### Exercício 3: Deployment com YAML

Crie o arquivo `web-app-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  labels:
    app: web-app
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
      tier: frontend
  template:
    metadata:
      labels:
        app: web-app
        tier: frontend
    spec:
      containers:
      - name: web-server
        image: nginx:1.21
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
```

1. Aplique o deployment
2. Verifique os recursos dos pods
3. Teste escalonamento
4. Faça uma atualização de imagem

## 🔍 Troubleshooting Comum

### Pods não Iniciam

```bash
# Verificar eventos
kubectl describe deployment web-app
kubectl get events --sort-by=.metadata.creationTimestamp

# Verificar logs
kubectl logs -l app=web-app
```

### Deployment Travado

```bash
# Verificar status do rollout
kubectl rollout status deployment/web-app --timeout=60s

# Se travado, fazer rollback
kubectl rollout undo deployment/web-app
```

### Recursos Insuficientes

```bash
# Verificar recursos do cluster
kubectl top nodes
kubectl describe nodes

# Ajustar requests/limits no deployment
kubectl edit deployment web-app
```

## 📈 Monitoramento Avançado

### Usando Labels para Filtragem

```bash
# Pods por app
kubectl get pods -l app=nginx-app

# Pods por tier
kubectl get pods -l tier=frontend

# Múltiplos labels
kubectl get pods -l app=web-app,tier=frontend
```

### Watch Mode

```bash
# Monitorar mudanças em tempo real
kubectl get pods -l app=nginx-app -w

# Monitorar deployments
kubectl get deployments -w
```

### Logs Agregados

```bash
# Logs de todos os pods do deployment
kubectl logs -f deployment/nginx-app

# Logs com timestamps
kubectl logs -f deployment/nginx-app --timestamps=true

# Logs das últimas 2 horas
kubectl logs deployment/nginx-app --since=2h
```

## 🎯 Boas Práticas Iniciais

### Naming Convention
- Use nomes descritivos: `frontend-web`, `api-backend`
- Inclua ambiente se necessário: `web-prod`, `api-dev`
- Use labels consistentes

### Recursos
- Sempre defina `requests` e `limits`
- Comece conservador e ajuste conforme necessário
- Monitore uso real vs configurado

### Versionamento
- Use tags específicas de imagem (evite `:latest`)
- Mantenha histórico de rollouts
- Documente mudanças importantes

### Monitoramento
- Configure health checks
- Use labels para organização
- Monitore logs regularmente

## 🔗 Próximos Passos

Após dominar deployments básicos:

1. **Services**: Expor aplicações para acesso
2. **ConfigMaps**: Gerenciar configurações
3. **Secrets**: Lidar com dados sensíveis
4. **Ingress**: Roteamento HTTP avançado
5. **HPA**: Auto-escalonamento baseado em métricas

## 📚 Comandos de Referência Rápida

```bash
# Criação
kubectl create deployment <name> --image=<image> --replicas=<n>
kubectl apply -f deployment.yaml

# Monitoramento
kubectl get deployments
kubectl describe deployment <name>
kubectl get pods -l app=<label>

# Escalonamento
kubectl scale deployment <name> --replicas=<n>

# Atualizações
kubectl set image deployment/<name> <container>=<new-image>
kubectl rollout status deployment/<name>
kubectl rollout history deployment/<name>

# Rollback
kubectl rollout undo deployment/<name>
kubectl rollout undo deployment/<name> --to-revision=<n>

# Limpeza
kubectl delete deployment <name>
```

Pratique estes comandos e conceitos antes de avançar para tópicos mais complexos!
