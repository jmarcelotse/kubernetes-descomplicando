# Conhecendo o YAML e o kubectl com dry-run

O **YAML** (YAML Ain't Markup Language) é o formato padrão para definir recursos no Kubernetes. O **dry-run** permite testar comandos sem aplicá-los ao cluster.

## O que é YAML?

YAML é um formato de serialização de dados legível por humanos, usado para configurações e definições de recursos no Kubernetes.

### Características do YAML
- **Indentação sensível** - usa espaços (não tabs)
- **Estrutura hierárquica** - baseada em indentação
- **Tipos de dados** - strings, números, booleanos, listas, objetos
- **Comentários** - linhas iniciadas com `#`

### Sintaxe Básica
```yaml
# Comentário
chave: valor
numero: 42
booleano: true
lista:
  - item1
  - item2
  - item3
objeto:
  propriedade1: valor1
  propriedade2: valor2
```

## Estrutura de um Recurso Kubernetes

Todo recurso Kubernetes segue uma estrutura YAML padrão:

```yaml
apiVersion: v1           # Versão da API
kind: Pod               # Tipo do recurso
metadata:               # Metadados
  name: meu-pod
  namespace: default
  labels:
    app: nginx
spec:                   # Especificação
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

### Campos Obrigatórios
- **apiVersion** - versão da API Kubernetes
- **kind** - tipo do recurso (Pod, Service, Deployment, etc.)
- **metadata** - informações sobre o recurso
- **spec** - especificação desejada do recurso

## O que é dry-run?

O **dry-run** permite simular a execução de comandos kubectl sem realmente aplicá-los ao cluster.

### Tipos de dry-run

#### client (--dry-run=client)
- Validação apenas no cliente
- Não envia requisição ao servidor
- Mais rápido, validação básica

#### server (--dry-run=server)
- Validação no servidor Kubernetes
- Envia requisição mas não persiste
- Validação completa (permissões, recursos, etc.)

## Gerando YAML com kubectl

### Comando Básico
```bash
kubectl create <recurso> <nome> [opções] --dry-run=client -o yaml
```

### Exemplos Práticos

#### Pod
```bash
kubectl run nginx --image=nginx --port=80 --dry-run=client -o yaml
```

#### Deployment
```bash
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml
```

#### Service
```bash
kubectl create service clusterip nginx --tcp=80:80 --dry-run=client -o yaml
```

#### ConfigMap
```bash
kubectl create configmap app-config --from-literal=key1=value1 --dry-run=client -o yaml
```

## Fluxo de Trabalho com dry-run

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        DRY-RUN WORKFLOW                                │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  1. GENERATE        2. REVIEW         3. SAVE          4. APPLY         │
│  ┌─────────────┐    ┌─────────────┐   ┌─────────────┐  ┌─────────────┐  │
│  │   kubectl   │───▶│    YAML     │──▶│    FILE     │─▶│   kubectl   │  │
│  │  dry-run    │    │   OUTPUT    │   │   .yaml     │  │   apply     │  │
│  └─────────────┘    └─────────────┘   └─────────────┘  └─────────────┘  │
│                                                                         │
│  5. VERIFY                                                              │
│  ┌─────────────┐                                                        │
│  │   kubectl   │                                                        │
│  │    get      │                                                        │
│  └─────────────┘                                                        │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## Exemplos Práticos

### 1. Criando um Pod

#### Gerar YAML
```bash
kubectl run webapp --image=nginx:1.20 --port=80 --dry-run=client -o yaml
```

#### Resultado
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: webapp
  name: webapp
spec:
  containers:
  - image: nginx:1.20
    name: webapp
    ports:
    - containerPort: 80
  restartPolicy: Always
```

#### Salvar em arquivo
```bash
kubectl run webapp --image=nginx:1.20 --port=80 --dry-run=client -o yaml > webapp-pod.yaml
```

### 2. Criando um Deployment

#### Gerar YAML
```bash
kubectl create deployment api --image=node:16 --replicas=3 --dry-run=client -o yaml
```

#### Resultado
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: api
  name: api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
    spec:
      containers:
      - image: node:16
        name: node
```

### 3. Criando um Service

#### Gerar YAML
```bash
kubectl create service nodeport webapp --tcp=80:80 --dry-run=client -o yaml
```

#### Resultado
```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: webapp
  name: webapp
spec:
  ports:
  - name: "80-80"
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: webapp
  type: NodePort
```

## Validação com dry-run

### Validação Client-side
```bash
# Validação básica de sintaxe
kubectl apply -f pod.yaml --dry-run=client
```

### Validação Server-side
```bash
# Validação completa no servidor
kubectl apply -f pod.yaml --dry-run=server
```

### Comparação de Validações

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      VALIDATION COMPARISON                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  CLIENT-SIDE                    SERVER-SIDE                             │
│  ┌─────────────────────────┐    ┌─────────────────────────────────────┐  │
│  │ • Sintaxe YAML          │    │ • Sintaxe YAML                     │  │
│  │ • Campos obrigatórios   │    │ • Campos obrigatórios               │  │
│  │ • Tipos de dados        │    │ • Tipos de dados                    │  │
│  │                         │    │ • Permissões RBAC                   │  │
│  │ ✓ Rápido               │    │ • Quotas de recursos                │  │
│  │ ✓ Offline              │    │ • Validação de esquema              │  │
│  │ ✗ Validação limitada   │    │ • Conflitos de nomes                │  │
│  │                         │    │                                     │  │
│  │                         │    │ ✓ Validação completa                │  │
│  │                         │    │ ✗ Requer conexão                    │  │
│  └─────────────────────────┘    └─────────────────────────────────────┘  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

## Editando YAML Gerado

### Exemplo: Personalizando um Pod
```bash
# Gerar base
kubectl run app --image=nginx --dry-run=client -o yaml > app.yaml

# Editar arquivo para adicionar:
# - Variables de ambiente
# - Volume mounts
# - Resource limits
# - Labels adicionais
```

### YAML Personalizado
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
  labels:
    app: webapp
    version: v1.0
    environment: production
spec:
  containers:
  - name: app
    image: nginx:1.20
    ports:
    - containerPort: 80
    env:
    - name: ENV
      value: "production"
    - name: DEBUG
      value: "false"
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
    volumeMounts:
    - name: config
      mountPath: /etc/config
  volumes:
  - name: config
    configMap:
      name: app-config
  restartPolicy: Always
```

## Boas Práticas

### 1. Sempre use dry-run primeiro
```bash
# Teste antes de aplicar
kubectl apply -f deployment.yaml --dry-run=server
kubectl apply -f deployment.yaml
```

### 2. Salve YAMLs gerados
```bash
# Mantenha histórico das configurações
kubectl create deployment app --image=nginx --dry-run=client -o yaml > deployments/app.yaml
```

### 3. Valide sintaxe
```bash
# Use ferramentas de validação
yamllint pod.yaml
kubectl apply -f pod.yaml --dry-run=client --validate=true
```

### 4. Organize arquivos
```
projeto/
├── pods/
│   ├── webapp.yaml
│   └── database.yaml
├── services/
│   ├── webapp-svc.yaml
│   └── database-svc.yaml
└── deployments/
    ├── webapp-deploy.yaml
    └── database-deploy.yaml
```

## Comandos Úteis

### Conversão de Formatos
```bash
# YAML para JSON
kubectl get pod webapp -o json

# JSON para YAML
kubectl get pod webapp -o yaml
```

### Explicação de Campos
```bash
# Documentação de campos
kubectl explain pod.spec
kubectl explain deployment.spec.template
kubectl explain service.spec
```

### Diff antes de aplicar
```bash
# Ver diferenças
kubectl diff -f deployment.yaml
```

## Troubleshooting YAML

### Erros Comuns

#### 1. Indentação incorreta
```yaml
# ❌ Errado
spec:
containers:
- name: app

# ✅ Correto
spec:
  containers:
  - name: app
```

#### 2. Tabs em vez de espaços
```bash
# Verificar tabs
cat -A arquivo.yaml
```

#### 3. Campos obrigatórios ausentes
```yaml
# ❌ Faltando apiVersion
kind: Pod
metadata:
  name: app

# ✅ Completo
apiVersion: v1
kind: Pod
metadata:
  name: app
```

### Validação de YAML
```bash
# Validar sintaxe
python -c "import yaml; yaml.safe_load(open('pod.yaml'))"

# Validar no Kubernetes
kubectl apply -f pod.yaml --dry-run=server --validate=true
```

## Próximos Passos

- Aprender sobre Deployments e ReplicaSets
- Trabalhar com Services e Ingress
- Configurar ConfigMaps e Secrets
- Implementar Health Checks e Probes
- Explorar Helm para templates YAML
