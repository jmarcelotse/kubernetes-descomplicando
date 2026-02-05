# Criando Deployment através de Manifesto

Guia completo para criar e gerenciar Deployments usando arquivos YAML (manifestos) no Kubernetes.

## 🎯 O que é um Manifesto?

Um **manifesto** é um arquivo YAML que descreve declarativamente o estado desejado de um recurso no Kubernetes. Para Deployments, o manifesto define:

- Quantas réplicas queremos
- Qual imagem usar
- Configurações de recursos
- Labels e seletores
- Estratégias de atualização

## 📋 Estrutura Básica do Manifesto

### Manifesto Mínimo

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: meu-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: meu-app
  template:
    metadata:
      labels:
        app: meu-app
    spec:
      containers:
      - name: container-principal
        image: nginx:1.21
        ports:
        - containerPort: 80
```

### Manifesto Completo

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-application
  namespace: default
  labels:
    app: web-app
    version: v1.0
    tier: frontend
  annotations:
    deployment.kubernetes.io/revision: "1"
    description: "Aplicação web principal"
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  selector:
    matchLabels:
      app: web-app
      tier: frontend
  template:
    metadata:
      labels:
        app: web-app
        tier: frontend
        version: v1.0
    spec:
      containers:
      - name: web-server
        image: nginx:1.21
        ports:
        - containerPort: 80
          name: http
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
        env:
        - name: ENV
          value: "production"
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
        volumeMounts:
        - name: config-volume
          mountPath: /etc/nginx/conf.d
      volumes:
      - name: config-volume
        configMap:
          name: nginx-config
```

## 🏗️ Anatomia do Manifesto

### 1. Cabeçalho (Header)
```yaml
apiVersion: apps/v1    # Versão da API
kind: Deployment       # Tipo do recurso
```

### 2. Metadados
```yaml
metadata:
  name: meu-deployment        # Nome único
  namespace: default          # Namespace (opcional)
  labels:                     # Labels para organização
    app: minha-app
    tier: frontend
  annotations:                # Metadados adicionais
    description: "Minha aplicação"
```

### 3. Especificação (Spec)
```yaml
spec:
  replicas: 3                 # Número de pods desejados
  selector:                   # Como encontrar pods gerenciados
    matchLabels:
      app: minha-app
  template:                   # Template do pod
    metadata:
      labels:
        app: minha-app
    spec:
      containers:             # Definição dos containers
      - name: container-name
        image: nginx:1.21
```

## 📝 Criando Manifestos Passo a Passo

### Passo 1: Arquivo Básico

Crie `basic-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-basic
  labels:
    app: nginx
spec:
  replicas: 2
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

### Passo 2: Aplicar o Manifesto

```bash
# Aplicar o deployment
kubectl apply -f basic-deployment.yaml

# Verificar criação
kubectl get deployments
kubectl get pods -l app=nginx
```

### Passo 3: Adicionar Recursos

Edite o arquivo para incluir recursos:

```yaml
spec:
  template:
    spec:
      containers:
      - name: nginx
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

### Passo 4: Aplicar Mudanças

```bash
# Aplicar atualizações
kubectl apply -f basic-deployment.yaml

# Verificar mudanças
kubectl describe deployment nginx-basic
```

## 🔄 Estratégias de Deployment

### Rolling Update (Padrão)

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%    # Máximo de pods indisponíveis
      maxSurge: 25%          # Máximo de pods extras
```

### Recreate

```yaml
spec:
  strategy:
    type: Recreate
```

## 🏥 Health Checks

### Liveness Probe

```yaml
spec:
  template:
    spec:
      containers:
      - name: app
        image: nginx:1.21
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
```

### Readiness Probe

```yaml
        readinessProbe:
          httpGet:
            path: /ready
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          successThreshold: 1
          failureThreshold: 3
```

## 🔧 Configurações Avançadas

### Variáveis de Ambiente

```yaml
spec:
  template:
    spec:
      containers:
      - name: app
        image: myapp:1.0
        env:
        - name: DATABASE_URL
          value: "postgresql://db:5432/myapp"
        - name: SECRET_KEY
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: secret-key
```

### Volumes

```yaml
spec:
  template:
    spec:
      containers:
      - name: app
        image: nginx:1.21
        volumeMounts:
        - name: config-volume
          mountPath: /etc/nginx/conf.d
        - name: data-volume
          mountPath: /var/www/html
      volumes:
      - name: config-volume
        configMap:
          name: nginx-config
      - name: data-volume
        persistentVolumeClaim:
          claimName: web-data-pvc
```

### Node Selector

```yaml
spec:
  template:
    spec:
      nodeSelector:
        disktype: ssd
        zone: us-west-1a
```

### Affinity e Anti-Affinity

```yaml
spec:
  template:
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - web-app
            topologyKey: kubernetes.io/hostname
```

## 📊 Exemplos Práticos

### Exemplo 1: Aplicação Web Simples

Arquivo: `web-app.yaml`

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
  template:
    metadata:
      labels:
        app: web-app
        tier: frontend
    spec:
      containers:
      - name: nginx
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

### Exemplo 2: API Backend

Arquivo: `api-backend.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-backend
  labels:
    app: api-backend
    tier: backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: api-backend
  template:
    metadata:
      labels:
        app: api-backend
        tier: backend
    spec:
      containers:
      - name: api
        image: node:16-alpine
        command: ["node", "server.js"]
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          value: "production"
        - name: PORT
          value: "3000"
        resources:
          requests:
            memory: "128Mi"
            cpu: "500m"
          limits:
            memory: "256Mi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
```

## 🛠️ Comandos para Manifestos

### Aplicação e Gerenciamento

```bash
# Aplicar manifesto
kubectl apply -f deployment.yaml

# Aplicar múltiplos arquivos
kubectl apply -f ./manifests/

# Aplicar com validação
kubectl apply -f deployment.yaml --validate=true

# Dry run (simular sem aplicar)
kubectl apply -f deployment.yaml --dry-run=client

# Ver diferenças antes de aplicar
kubectl diff -f deployment.yaml
```

### Validação e Debug

```bash
# Validar sintaxe YAML
kubectl apply -f deployment.yaml --dry-run=client --validate=true

# Explicar campos do manifesto
kubectl explain deployment.spec.template.spec.containers

# Ver manifesto aplicado
kubectl get deployment nginx-basic -o yaml
```

### Atualizações

```bash
# Aplicar mudanças
kubectl apply -f deployment.yaml

# Forçar recriação
kubectl replace -f deployment.yaml --force

# Deletar e recriar
kubectl delete -f deployment.yaml
kubectl apply -f deployment.yaml
```

## 🔍 Troubleshooting de Manifestos

### Erros Comuns

#### 1. Seletor não corresponde aos labels
```yaml
# ❌ Erro
spec:
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: webapp  # Diferente do seletor
```

```yaml
# ✅ Correto
spec:
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app  # Igual ao seletor
```

#### 2. Recursos insuficientes
```bash
# Verificar eventos
kubectl describe deployment meu-app

# Ver recursos disponíveis
kubectl top nodes
kubectl describe nodes
```

#### 3. Imagem não encontrada
```bash
# Verificar logs dos pods
kubectl logs -l app=meu-app

# Verificar eventos
kubectl get events --sort-by=.metadata.creationTimestamp
```

## 📋 Template de Manifesto

Use este template como base:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: <NOME-DO-APP>
  labels:
    app: <NOME-DO-APP>
    version: <VERSAO>
    tier: <TIER>
spec:
  replicas: <NUMERO-REPLICAS>
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  selector:
    matchLabels:
      app: <NOME-DO-APP>
  template:
    metadata:
      labels:
        app: <NOME-DO-APP>
        version: <VERSAO>
        tier: <TIER>
    spec:
      containers:
      - name: <NOME-CONTAINER>
        image: <IMAGEM>:<TAG>
        ports:
        - containerPort: <PORTA>
        resources:
          requests:
            memory: "<MEMORIA-REQUEST>"
            cpu: "<CPU-REQUEST>"
          limits:
            memory: "<MEMORIA-LIMIT>"
            cpu: "<CPU-LIMIT>"
        livenessProbe:
          httpGet:
            path: <HEALTH-PATH>
            port: <PORTA>
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: <READY-PATH>
            port: <PORTA>
          initialDelaySeconds: 5
          periodSeconds: 5
```

## 🎯 Boas Práticas

### Organização de Arquivos
- Um manifesto por arquivo
- Nomes descritivos: `frontend-deployment.yaml`
- Organize por ambiente: `prod/`, `dev/`, `staging/`

### Versionamento
- Use tags específicas de imagem
- Versione seus manifestos no Git
- Documente mudanças importantes

### Segurança
- Não inclua secrets nos manifestos
- Use ServiceAccounts específicos
- Configure security contexts

### Recursos
- Sempre defina requests e limits
- Use valores baseados em monitoramento
- Configure health checks adequados

### Labels
- Use labels consistentes
- Inclua informações úteis: app, version, tier
- Facilite filtragem e seleção

## 🔗 Próximos Passos

Após dominar manifestos de Deployment:
- [Services](criando-service-manifesto.md) - Expor aplicações
- [ConfigMaps](criando-configmap-manifesto.md) - Configurações
- [Secrets](criando-secret-manifesto.md) - Dados sensíveis
- [Kustomize](usando-kustomize.md) - Gerenciamento de manifestos

## 📚 Referências

- [Documentação Oficial - Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [API Reference - Deployment](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.28/#deployment-v1-apps)
- [Kubectl Reference](https://kubernetes.io/docs/reference/kubectl/)
