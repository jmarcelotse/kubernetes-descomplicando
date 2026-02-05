# Configurando Volume EmptyDir

EmptyDir é o tipo de volume mais simples no Kubernetes, criado quando um pod é atribuído a um node e existe enquanto o pod estiver rodando.

## Conceitos Fundamentais

### O que é EmptyDir?
- Volume temporário criado no node
- Compartilhado entre containers do mesmo pod
- Removido quando pod é deletado
- Armazenado no disco local do node

### Características
- **Temporário**: Dados perdidos quando pod morre
- **Compartilhado**: Acessível por todos containers do pod
- **Local**: Armazenado no filesystem do node
- **Rápido**: Acesso direto ao disco local

## Casos de Uso

### Compartilhamento de Dados
- Troca de arquivos entre containers
- Cache temporário
- Processamento de dados em pipeline
- Logs compartilhados

### Armazenamento Temporário
- Scratch space para computações
- Arquivos temporários de aplicações
- Buffer para processamento de dados

## Configuração Básica

### Pod com EmptyDir
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-pod
spec:
  containers:
  - name: writer
    image: busybox
    command: ['sh', '-c']
    args:
    - |
      while true; do
        echo "$(date): Escrevendo dados..." >> /shared/data.log
        sleep 10
      done
    volumeMounts:
    - name: shared-storage
      mountPath: /shared
      
  - name: reader
    image: busybox
    command: ['sh', '-c']
    args:
    - |
      while true; do
        if [ -f /shared/data.log ]; then
          echo "=== Últimas 5 linhas ==="
          tail -n 5 /shared/data.log
        fi
        sleep 15
      done
    volumeMounts:
    - name: shared-storage
      mountPath: /shared
      
  volumes:
  - name: shared-storage
    emptyDir: {}
```

### EmptyDir com Configurações
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-configured
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: cache-volume
      mountPath: /var/cache/nginx
    - name: memory-volume
      mountPath: /tmp/memory
      
  volumes:
  - name: cache-volume
    emptyDir:
      sizeLimit: 1Gi          # Limite de tamanho
  - name: memory-volume
    emptyDir:
      medium: Memory          # Armazenar na RAM
      sizeLimit: 512Mi
```

## Diagrama de Funcionamento

```
┌─────────────────────────────────────────────────────────────┐
│                        Node                                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                    Pod                              │   │
│  │                                                     │   │
│  │  ┌─────────────┐    ┌─────────────────────────┐     │   │
│  │  │Container A  │    │      Container B        │     │   │
│  │  │             │    │                         │     │   │
│  │  │/app/data ───┼────┼──► EmptyDir Volume ◄────┼───  │   │
│  │  │             │    │                         │ /tmp│   │
│  │  └─────────────┘    └─────────────────────────┘     │   │
│  │                                                     │   │
│  └─────────────────────────────────────────────────────┘   │
│                              │                             │
│                              ▼                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │            Node Filesystem                          │   │
│  │  /var/lib/kubelet/pods/<pod-uid>/volumes/           │   │
│  │  └── kubernetes.io~empty-dir/                       │   │
│  │      └── shared-storage/                            │   │
│  │          ├── data.log                               │   │
│  │          └── other-files                            │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

## Exemplo Prático: Web Server + Log Processor

### Manifesto Completo
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-with-logs
  labels:
    app: web-logs-demo
spec:
  containers:
  # Servidor web principal
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
    volumeMounts:
    - name: nginx-logs
      mountPath: /var/log/nginx
    - name: shared-content
      mountPath: /usr/share/nginx/html
      
  # Processador de logs
  - name: log-processor
    image: busybox
    command: ['sh', '-c']
    args:
    - |
      while true; do
        if [ -f /logs/access.log ]; then
          echo "=== $(date) ==="
          echo "Total requests: $(wc -l < /logs/access.log)"
          echo "Last 3 requests:"
          tail -n 3 /logs/access.log
          echo "========================"
        fi
        sleep 30
      done
    volumeMounts:
    - name: nginx-logs
      mountPath: /logs
      
  # Gerador de conteúdo
  - name: content-generator
    image: busybox
    command: ['sh', '-c']
    args:
    - |
      echo "<h1>Web Server com EmptyDir</h1>" > /content/index.html
      echo "<p>Gerado em: $(date)</p>" >> /content/index.html
      echo "<p>Pod: $HOSTNAME</p>" >> /content/index.html
      while true; do
        echo "<p>Atualizado: $(date)</p>" >> /content/status.html
        sleep 60
      done
    volumeMounts:
    - name: shared-content
      mountPath: /content
      
  volumes:
  - name: nginx-logs
    emptyDir: {}
  - name: shared-content
    emptyDir:
      sizeLimit: 100Mi
```

## Tipos de EmptyDir

### 1. Disco Local (Padrão)
```yaml
volumes:
- name: disk-storage
  emptyDir: {}                # Usa disco do node
```

### 2. Memória RAM
```yaml
volumes:
- name: memory-storage
  emptyDir:
    medium: Memory            # Usa RAM do node
    sizeLimit: 1Gi
```

### 3. Com Limite de Tamanho
```yaml
volumes:
- name: limited-storage
  emptyDir:
    sizeLimit: 2Gi            # Máximo 2GB
```

## Exemplo: Cache de Aplicação

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-cache
spec:
  containers:
  - name: web-app
    image: nginx
    volumeMounts:
    - name: app-cache
      mountPath: /var/cache/app
    - name: temp-files
      mountPath: /tmp
    env:
    - name: CACHE_DIR
      value: "/var/cache/app"
      
  - name: cache-warmer
    image: busybox
    command: ['sh', '-c']
    args:
    - |
      while true; do
        echo "Warming cache at $(date)" > /cache/warm.txt
        echo "Cache status: Active" > /cache/status.txt
        sleep 300  # A cada 5 minutos
      done
    volumeMounts:
    - name: app-cache
      mountPath: /cache
      
  volumes:
  - name: app-cache
    emptyDir:
      sizeLimit: 500Mi
  - name: temp-files
    emptyDir:
      medium: Memory
      sizeLimit: 100Mi
```

## Comandos Úteis

### Criar e Testar Pod
```bash
# Aplicar manifesto
kubectl apply -f emptydir-pod.yaml

# Verificar pod
kubectl get pods

# Ver logs do container writer
kubectl logs emptydir-pod -c writer

# Ver logs do container reader
kubectl logs emptydir-pod -c reader
```

### Acessar Volume
```bash
# Executar comando no container
kubectl exec -it emptydir-pod -c writer -- ls -la /shared

# Ver conteúdo do arquivo
kubectl exec emptydir-pod -c reader -- cat /shared/data.log

# Criar arquivo manualmente
kubectl exec emptydir-pod -c writer -- sh -c "echo 'Teste manual' >> /shared/test.txt"
```

### Monitorar Uso
```bash
# Ver uso de disco do pod
kubectl exec emptydir-pod -c writer -- df -h /shared

# Ver arquivos no volume
kubectl exec emptydir-pod -c writer -- find /shared -type f -exec ls -lh {} \;
```

## Comparação com Outros Volumes

| Tipo | Persistência | Compartilhamento | Performance | Uso |
|------|-------------|------------------|-------------|-----|
| **EmptyDir** | Temporário | Entre containers do pod | Alta | Cache, temp files |
| **HostPath** | Persistente | Node específico | Alta | Logs do sistema |
| **PVC** | Persistente | Entre pods | Média | Dados da aplicação |
| **ConfigMap** | Imutável | Entre pods | Alta | Configurações |

## Limitações e Considerações

### Limitações
- **Temporário**: Dados perdidos quando pod é deletado
- **Local**: Limitado ao espaço do node
- **Não compartilhado**: Apenas entre containers do mesmo pod

### Considerações de Performance
```yaml
# Para alta performance (RAM)
emptyDir:
  medium: Memory
  sizeLimit: 1Gi

# Para economia de recursos (disco)
emptyDir:
  sizeLimit: 500Mi
```

### Monitoramento de Espaço
```bash
# Script para monitorar uso
kubectl exec pod-name -c container -- sh -c '
  while true; do
    echo "=== $(date) ==="
    df -h /mounted/path
    du -sh /mounted/path/*
    sleep 60
  done
'
```

## Troubleshooting

### Pod Pending por Espaço
```bash
# Verificar eventos
kubectl describe pod pod-name

# Verificar espaço no node
kubectl describe node node-name

# Ver uso de volumes
kubectl exec pod-name -- df -h
```

### Volume Não Montado
```bash
# Verificar configuração
kubectl get pod pod-name -o yaml | grep -A 10 volumes

# Ver eventos de montagem
kubectl describe pod pod-name | grep -A 5 Events
```

### Problemas de Permissão
```yaml
# Configurar securityContext
spec:
  securityContext:
    fsGroup: 2000
  containers:
  - name: app
    securityContext:
      runAsUser: 1000
      runAsGroup: 2000
```

## Exemplo Avançado: Pipeline de Dados

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: data-pipeline
spec:
  containers:
  # Gerador de dados
  - name: data-generator
    image: busybox
    command: ['sh', '-c']
    args:
    - |
      for i in $(seq 1 100); do
        echo "data-$i,$(date),value-$RANDOM" >> /pipeline/raw-data.csv
        sleep 5
      done
    volumeMounts:
    - name: pipeline-data
      mountPath: /pipeline
      
  # Processador
  - name: data-processor
    image: busybox
    command: ['sh', '-c']
    args:
    - |
      while true; do
        if [ -f /pipeline/raw-data.csv ]; then
          echo "Processing data..."
          sort /pipeline/raw-data.csv > /pipeline/processed-data.csv
          wc -l /pipeline/processed-data.csv > /pipeline/stats.txt
        fi
        sleep 10
      done
    volumeMounts:
    - name: pipeline-data
      mountPath: /pipeline
      
  # Monitor
  - name: monitor
    image: busybox
    command: ['sh', '-c']
    args:
    - |
      while true; do
        echo "=== Pipeline Status ==="
        ls -la /pipeline/
        if [ -f /pipeline/stats.txt ]; then
          cat /pipeline/stats.txt
        fi
        sleep 30
      done
    volumeMounts:
    - name: pipeline-data
      mountPath: /pipeline
      
  volumes:
  - name: pipeline-data
    emptyDir:
      sizeLimit: 1Gi
```

## Boas Práticas

### Definir Limites
```yaml
emptyDir:
  sizeLimit: 1Gi              # Sempre definir limite
```

### Usar RAM para Performance
```yaml
emptyDir:
  medium: Memory              # Para dados temporários críticos
  sizeLimit: 512Mi
```

### Monitorar Uso
```bash
# Incluir monitoramento no container
while true; do
  df -h /mounted/path | tail -1
  sleep 60
done
```

### Limpeza Automática
```yaml
# Container de limpeza
- name: cleaner
  image: busybox
  command: ['sh', '-c']
  args:
  - |
    while true; do
      find /shared -type f -mtime +1 -delete
      sleep 3600  # A cada hora
    done
```

## Conclusão

EmptyDir é ideal para:
- **Compartilhamento temporário** entre containers
- **Cache de aplicações** com alta performance
- **Processamento de dados** em pipeline
- **Armazenamento temporário** de arquivos

**Lembre-se**: EmptyDir é temporário - use PersistentVolumes para dados que precisam sobreviver ao ciclo de vida do pod.
