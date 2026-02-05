# Como Atualizar um Deployment no Kubernetes

Guia completo para atualizar Deployments no Kubernetes, incluindo diferentes métodos, estratégias e boas práticas.

## 🎯 O que é uma Atualização de Deployment?

Uma **atualização de deployment** é o processo de modificar um deployment existente para:
- Alterar a versão da imagem do container
- Modificar configurações (variáveis, recursos, etc.)
- Ajustar número de réplicas
- Atualizar labels ou annotations

O Kubernetes gerencia automaticamente o processo de atualização usando **Rolling Updates** por padrão.

## 🔄 Métodos de Atualização

### 1. Usando kubectl set image

```bash
# Atualizar imagem do container
kubectl set image deployment/nginx-deployment nginx=nginx:1.22

# Verificar status da atualização
kubectl rollout status deployment/nginx-deployment

# Ver histórico
kubectl rollout history deployment/nginx-deployment
```

### 2. Usando kubectl edit

```bash
# Editar deployment interativamente
kubectl edit deployment nginx-deployment

# O editor padrão abrirá com o YAML do deployment
# Modifique a imagem ou outras configurações
# Salve e feche para aplicar
```

### 3. Usando kubectl patch

```bash
# Patch específico para imagem
kubectl patch deployment nginx-deployment -p '{"spec":{"template":{"spec":{"containers":[{"name":"nginx","image":"nginx:1.22"}]}}}}'

# Patch para variáveis de ambiente
kubectl patch deployment nginx-deployment -p '{"spec":{"template":{"spec":{"containers":[{"name":"nginx","env":[{"name":"ENV","value":"production"}]}]}}}}'
```

### 4. Usando kubectl apply

```bash
# Modificar arquivo YAML
# deployment.yaml: image: nginx:1.22

# Aplicar mudanças
kubectl apply -f deployment.yaml

# Verificar atualização
kubectl rollout status deployment/nginx-deployment
```

### 5. Usando kubectl replace

```bash
# Substituir deployment completamente
kubectl replace -f deployment-updated.yaml

# Forçar substituição
kubectl replace -f deployment.yaml --force
```

## 📊 Processo de Rolling Update

### Como Funciona

1. **Nova versão especificada**: Kubernetes detecta mudança na spec
2. **Novo ReplicaSet criado**: Com a nova configuração
3. **Pods criados gradualmente**: Respeitando maxSurge
4. **Pods antigos terminados**: Respeitando maxUnavailable
5. **Processo continua**: Até todos os pods serem atualizados

### Exemplo Prático

```bash
# Estado inicial
kubectl get deployments
# nginx-deployment   3/3     3            3           30m

# Atualizar imagem
kubectl set image deployment/nginx-deployment nginx=nginx:1.22

# Durante a atualização
kubectl get pods -w
# nginx-deployment-old-abc123   1/1     Running       0          30m
# nginx-deployment-old-abc124   1/1     Running       0          30m
# nginx-deployment-old-abc125   1/1     Running       0          30m
# nginx-deployment-new-def456   0/1     Pending       0          0s
# nginx-deployment-new-def456   0/1     ContainerCreating   0     0s
# nginx-deployment-new-def456   1/1     Running       0          10s
# nginx-deployment-old-abc123   1/1     Terminating   0          30m
```

## ⚙️ Configurando Estratégias de Atualização

### Rolling Update (Padrão)

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%    # Máximo de pods indisponíveis
      maxSurge: 25%          # Máximo de pods extras
```

### Configurações Comuns

```yaml
# Atualização conservadora (zero downtime)
rollingUpdate:
  maxUnavailable: 0
  maxSurge: 1

# Atualização rápida
rollingUpdate:
  maxUnavailable: 50%
  maxSurge: 50%

# Atualização uma por vez
rollingUpdate:
  maxUnavailable: 1
  maxSurge: 1
```

### Recreate Strategy

```yaml
spec:
  strategy:
    type: Recreate
```

**Quando usar:**
- Aplicações que não suportam múltiplas versões
- Recursos compartilhados (volumes, portas)
- Aplicações stateful simples

## 🔍 Monitorando Atualizações

### Comandos de Monitoramento

```bash
# Status da atualização
kubectl rollout status deployment/nginx-deployment

# Histórico de revisões
kubectl rollout history deployment/nginx-deployment

# Detalhes de uma revisão específica
kubectl rollout history deployment/nginx-deployment --revision=2

# Pausar atualização
kubectl rollout pause deployment/nginx-deployment

# Retomar atualização
kubectl rollout resume deployment/nginx-deployment
```

### Verificando Progresso

```bash
# Pods em tempo real
kubectl get pods -l app=nginx -w

# ReplicaSets
kubectl get rs -l app=nginx

# Eventos
kubectl get events --sort-by=.metadata.creationTimestamp

# Logs durante atualização
kubectl logs -f deployment/nginx-deployment
```

## 🔙 Rollback de Deployments

### Rollback Rápido

```bash
# Voltar para versão anterior
kubectl rollout undo deployment/nginx-deployment

# Verificar rollback
kubectl rollout status deployment/nginx-deployment
```

### Rollback para Versão Específica

```bash
# Ver histórico
kubectl rollout history deployment/nginx-deployment

# Rollback para revisão específica
kubectl rollout undo deployment/nginx-deployment --to-revision=1

# Confirmar rollback
kubectl get deployment nginx-deployment -o wide
```

## 🛠️ Exemplos Práticos

### Exemplo 1: Atualização de Imagem

```bash
# Estado inicial
kubectl get deployment nginx-deployment
# nginx-deployment   3/3     3            3           5m

# Atualizar nginx 1.21 → 1.22
kubectl set image deployment/nginx-deployment nginx=nginx:1.22

# Monitorar
kubectl rollout status deployment/nginx-deployment
# deployment "nginx-deployment" successfully rolled out

# Verificar nova imagem
kubectl describe deployment nginx-deployment | grep Image
# Image: nginx:1.22
```

### Exemplo 2: Adicionando Variáveis de Ambiente

```bash
# Adicionar variável ENV
kubectl patch deployment nginx-deployment -p '{
  "spec": {
    "template": {
      "spec": {
        "containers": [{
          "name": "nginx",
          "env": [
            {"name": "ENVIRONMENT", "value": "production"},
            {"name": "LOG_LEVEL", "value": "info"}
          ]
        }]
      }
    }
  }
}'

# Verificar atualização
kubectl rollout status deployment/nginx-deployment
```

### Exemplo 3: Atualizando Recursos

```bash
# Atualizar requests e limits
kubectl patch deployment nginx-deployment -p '{
  "spec": {
    "template": {
      "spec": {
        "containers": [{
          "name": "nginx",
          "resources": {
            "requests": {
              "memory": "128Mi",
              "cpu": "200m"
            },
            "limits": {
              "memory": "256Mi",
              "cpu": "400m"
            }
          }
        }]
      }
    }
  }
}'
```

### Exemplo 4: Atualização via Arquivo YAML

```yaml
# deployment-v2.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
    version: v2
spec:
  replicas: 4  # Aumentar réplicas
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
        version: v2
    spec:
      containers:
      - name: nginx
        image: nginx:1.22  # Nova versão
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "128Mi"
            cpu: "200m"
          limits:
            memory: "256Mi"
            cpu: "400m"
        env:
        - name: ENVIRONMENT
          value: "production"
```

```bash
# Aplicar atualização
kubectl apply -f deployment-v2.yaml

# Monitorar
kubectl rollout status deployment/nginx-deployment
```

## 🚨 Troubleshooting de Atualizações

### Problemas Comuns

#### 1. Atualização Travada

```bash
# Verificar status
kubectl rollout status deployment/nginx-deployment --timeout=60s

# Se travado, verificar pods
kubectl get pods -l app=nginx
kubectl describe pod <pod-name>

# Possível rollback
kubectl rollout undo deployment/nginx-deployment
```

#### 2. Imagem Não Encontrada

```bash
# Verificar eventos
kubectl get events --sort-by=.metadata.creationTimestamp

# Logs do pod
kubectl logs <pod-name>

# Corrigir imagem
kubectl set image deployment/nginx-deployment nginx=nginx:1.21
```

#### 3. Recursos Insuficientes

```bash
# Verificar recursos do cluster
kubectl top nodes
kubectl describe nodes

# Ajustar recursos no deployment
kubectl patch deployment nginx-deployment -p '{
  "spec": {
    "template": {
      "spec": {
        "containers": [{
          "name": "nginx",
          "resources": {
            "requests": {"memory": "64Mi", "cpu": "100m"}
          }
        }]
      }
    }
  }
}'
```

## 🎯 Boas Práticas

### Antes da Atualização

1. **Backup**: Salve configuração atual
```bash
kubectl get deployment nginx-deployment -o yaml > backup-deployment.yaml
```

2. **Teste**: Valide em ambiente de desenvolvimento
3. **Recursos**: Verifique disponibilidade no cluster
4. **Dependências**: Considere ordem de atualização

### Durante a Atualização

1. **Monitore**: Use `kubectl rollout status`
2. **Logs**: Acompanhe logs dos novos pods
3. **Health Checks**: Configure probes adequados
4. **Gradual**: Use maxUnavailable conservador

### Após a Atualização

1. **Verificação**: Teste funcionalidade da aplicação
2. **Monitoramento**: Observe métricas e logs
3. **Limpeza**: Remova ReplicaSets antigos se necessário
4. **Documentação**: Registre mudanças realizadas

### Configurações Recomendadas

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  template:
    spec:
      containers:
      - name: app
        image: app:v2
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
```

## 📋 Comandos de Referência

### Atualização
```bash
# Imagem
kubectl set image deployment/<name> <container>=<image>

# Edição interativa
kubectl edit deployment <name>

# Patch
kubectl patch deployment <name> -p '<json-patch>'

# Arquivo
kubectl apply -f deployment.yaml
```

### Monitoramento
```bash
# Status
kubectl rollout status deployment/<name>

# Histórico
kubectl rollout history deployment/<name>

# Pausar/Retomar
kubectl rollout pause deployment/<name>
kubectl rollout resume deployment/<name>
```

### Rollback
```bash
# Versão anterior
kubectl rollout undo deployment/<name>

# Versão específica
kubectl rollout undo deployment/<name> --to-revision=<n>
```

### Debug
```bash
# Pods
kubectl get pods -l app=<label>

# Eventos
kubectl get events --sort-by=.metadata.creationTimestamp

# Logs
kubectl logs -f deployment/<name>
```

## 🔗 Próximos Passos

Após dominar atualizações de deployment:
- [Blue-Green Deployments](blue-green-deployments.md)
- [Canary Deployments](canary-deployments.md)
- [GitOps](gitops-deployments.md)
- [Helm Charts](helm-deployments.md)

## 📚 Referências

- [Rolling Updates](https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/)
- [Deployment Strategies](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#strategy)
- [Rollback](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-back-a-deployment)
