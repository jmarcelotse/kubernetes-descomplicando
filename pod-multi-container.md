# Criando Pod Multi-Container com Manifesto

Pods multi-container permitem executar múltiplos containers que compartilham recursos como rede, volumes e ciclo de vida.

## Conceitos Fundamentais

### O que é um Pod Multi-Container?
Um pod que contém dois ou mais containers que:
- Compartilham o mesmo endereço IP
- Compartilham volumes
- São criados e destruídos juntos
- Comunicam-se via localhost

### Padrões Comuns
1. **Sidecar**: Container auxiliar que complementa o principal
2. **Ambassador**: Proxy para comunicação externa
3. **Adapter**: Padroniza saída de dados

## Manifesto Multi-Container

### Estrutura Básica
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
  labels:
    app: web-with-sidecar
spec:
  containers:
  # Container principal
  - name: nginx-container
    image: nginx:latest
    ports:
    - containerPort: 80
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx
    
  # Container sidecar
  - name: log-collector
    image: busybox
    command: ['sh', '-c']
    args:
    - |
      while true; do
        echo "$(date): Coletando logs..." >> /var/log/collector.log
        if [ -f /var/log/nginx/access.log ]; then
          tail -n 5 /var/log/nginx/access.log >> /var/log/collector.log
        fi
        sleep 30
      done
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx
    - name: collector-logs
      mountPath: /var/log
      
  volumes:
  - name: shared-logs
    emptyDir: {}
  - name: collector-logs
    emptyDir: {}
```

## Componentes do Manifesto

### Containers
```yaml
containers:
- name: nginx-container          # Nome único do container
  image: nginx:latest           # Imagem Docker
  ports:                        # Portas expostas
  - containerPort: 80
  volumeMounts:                 # Volumes montados
  - name: shared-logs
    mountPath: /var/log/nginx
```

### Volumes Compartilhados
```yaml
volumes:
- name: shared-logs             # Volume compartilhado
  emptyDir: {}                  # Tipo emptyDir (temporário)
- name: collector-logs
  emptyDir: {}
```

### Comandos Personalizados
```yaml
command: ['sh', '-c']           # Comando de entrada
args:                           # Argumentos do comando
- |
  while true; do
    echo "Executando tarefa..."
    sleep 30
  done
```

## Diagrama de Arquitetura

```
┌─────────────────────────────────────────────────────────────┐
│                    Pod Multi-Container                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────┐    ┌─────────────────────────┐     │
│  │  nginx-container    │    │   log-collector         │     │
│  │                     │    │                         │     │
│  │  ┌─────────────┐    │    │  ┌─────────────────┐    │     │
│  │  │   nginx     │    │    │  │   busybox       │    │     │
│  │  │   :80       │    │    │  │   (script)      │    │     │
│  │  └─────────────┘    │    │  └─────────────────┘    │     │
│  │         │           │    │          │              │     │
│  │         ▼           │    │          ▼              │     │
│  │  /var/log/nginx ────┼────┼──► /var/log/nginx       │     │
│  │                     │    │                         │     │
│  └─────────────────────┘    │  /var/log/collector.log │     │
│                             └─────────────────────────┘     │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                 Volumes                             │   │
│  │  ┌─────────────┐    ┌─────────────────────────┐     │   │
│  │  │shared-logs  │    │   collector-logs        │     │   │
│  │  │(emptyDir)   │    │   (emptyDir)            │     │   │
│  │  └─────────────┘    └─────────────────────────┘     │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  Rede: 10.244.x.x (IP compartilhado)                       │
└─────────────────────────────────────────────────────────────┘
```

## Criando o Pod

### Aplicar o Manifesto
```bash
# Criar o pod
kubectl apply -f k8s/pod/multi-container-pod.yaml

# Verificar status
kubectl get pods

# Ver detalhes
kubectl describe pod multi-container-pod
```

### Verificar Containers
```bash
# Listar containers do pod
kubectl get pod multi-container-pod -o jsonpath='{.spec.containers[*].name}'

# Status de cada container
kubectl get pod multi-container-pod -o jsonpath='{.status.containerStatuses[*].name}'
```

## Interagindo com Containers

### Executar Comandos
```bash
# Acessar container nginx
kubectl exec -it multi-container-pod -c nginx-container -- bash

# Acessar container log-collector
kubectl exec -it multi-container-pod -c log-collector -- sh

# Executar comando específico
kubectl exec multi-container-pod -c nginx-container -- nginx -v
```

### Visualizar Logs
```bash
# Logs do container nginx
kubectl logs multi-container-pod -c nginx-container

# Logs do log-collector
kubectl logs multi-container-pod -c log-collector

# Seguir logs em tempo real
kubectl logs -f multi-container-pod -c log-collector
```

## Testando a Comunicação

### Gerar Tráfego para Nginx
```bash
# Port-forward para acessar nginx
kubectl port-forward multi-container-pod 8080:80

# Em outro terminal, gerar requests
curl http://localhost:8080
curl http://localhost:8080/index.html
```

### Verificar Logs Coletados
```bash
# Ver logs coletados pelo sidecar
kubectl exec multi-container-pod -c log-collector -- cat /var/log/collector.log

# Ver logs do nginx diretamente
kubectl exec multi-container-pod -c nginx-container -- cat /var/log/nginx/access.log
```

## Padrões de Design

### 1. Sidecar Pattern
```yaml
# Container principal + auxiliar
containers:
- name: app
  image: myapp:latest
- name: logging-agent
  image: fluentd:latest
```

### 2. Ambassador Pattern
```yaml
# Container principal + proxy
containers:
- name: app
  image: myapp:latest
- name: ambassador
  image: nginx:latest  # Proxy reverso
```

### 3. Adapter Pattern
```yaml
# Container principal + adaptador
containers:
- name: app
  image: legacy-app:latest
- name: adapter
  image: prometheus-adapter:latest
```

## Exemplo Avançado: Web + Database Proxy

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-with-proxy
spec:
  containers:
  - name: web-app
    image: nginx:latest
    ports:
    - containerPort: 80
    
  - name: db-proxy
    image: haproxy:latest
    ports:
    - containerPort: 3306
    volumeMounts:
    - name: proxy-config
      mountPath: /usr/local/etc/haproxy
      
  - name: monitoring
    image: prom/node-exporter:latest
    ports:
    - containerPort: 9100
    
  volumes:
  - name: proxy-config
    configMap:
      name: haproxy-config
```

## Comunicação Entre Containers

### Via Localhost
```bash
# Container A pode acessar container B via localhost
curl http://localhost:3306  # Do web-app para db-proxy
curl http://localhost:9100  # Do web-app para monitoring
```

### Via Volumes Compartilhados
```bash
# Compartilhar arquivos entre containers
/shared-data/
├── app-data/     # Escrito pelo container principal
└── processed/    # Processado pelo sidecar
```

## Monitoramento e Debug

### Verificar Status dos Containers
```bash
# Status detalhado
kubectl describe pod multi-container-pod

# Eventos do pod
kubectl get events --field-selector involvedObject.name=multi-container-pod

# Recursos utilizados
kubectl top pod multi-container-pod --containers
```

### Debug de Problemas
```bash
# Verificar se todos containers estão rodando
kubectl get pod multi-container-pod -o jsonpath='{.status.containerStatuses[*].ready}'

# Ver motivo de falha
kubectl describe pod multi-container-pod | grep -A 10 "Container Statuses"

# Logs de inicialização
kubectl logs multi-container-pod -c nginx-container --previous
```

## Boas Práticas

### Design
- **Responsabilidade única**: Cada container deve ter uma função específica
- **Acoplamento fraco**: Containers devem ser independentes quando possível
- **Comunicação eficiente**: Use localhost para comunicação interna

### Recursos
```yaml
containers:
- name: app
  resources:
    requests:
      memory: "64Mi"
      cpu: "250m"
    limits:
      memory: "128Mi"
      cpu: "500m"
```

### Segurança
```yaml
containers:
- name: app
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    readOnlyRootFilesystem: true
```

## Casos de Uso Comuns

### 1. Aplicação Web + Log Collector
- **Principal**: Servidor web (nginx, apache)
- **Sidecar**: Coletor de logs (fluentd, filebeat)

### 2. Aplicação + Proxy de Banco
- **Principal**: Aplicação
- **Sidecar**: Proxy de conexão (pgbouncer, mysql-proxy)

### 3. Aplicação + Monitoramento
- **Principal**: Aplicação
- **Sidecar**: Agente de métricas (prometheus exporter)

### 4. Aplicação + Cache
- **Principal**: API
- **Sidecar**: Cache local (redis, memcached)

## Limitações

- **Complexidade**: Mais difícil de debugar
- **Recursos**: Maior consumo de CPU/memória
- **Dependências**: Falha de um container afeta o pod inteiro
- **Escalabilidade**: Todo o pod escala junto

## Limpeza

```bash
# Deletar o pod
kubectl delete pod multi-container-pod

# Verificar remoção
kubectl get pods
```

## Conclusão

Pods multi-container são poderosos para implementar padrões como sidecar, ambassador e adapter. Eles permitem separação de responsabilidades mantendo comunicação eficiente entre componentes relacionados.

Use quando precisar de:
- Componentes fortemente acoplados
- Compartilhamento de dados em tempo real
- Processamento auxiliar (logs, métricas, proxy)
- Padrões de design específicos
