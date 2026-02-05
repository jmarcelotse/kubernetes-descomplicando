# kubectl attach e kubectl exec

Comandos essenciais para interagir com containers em execução no Kubernetes.

## kubectl exec

Executa comandos dentro de containers em execução.

### Sintaxe Básica
```bash
kubectl exec [OPTIONS] POD_NAME -- COMMAND [args...]
kubectl exec [OPTIONS] POD_NAME -c CONTAINER_NAME -- COMMAND [args...]
```

### Exemplos Práticos

#### Executar comando simples
```bash
# Listar arquivos no container
kubectl exec test-pod -- ls -la

# Ver processos em execução
kubectl exec test-pod -- ps aux

# Verificar versão do nginx
kubectl exec test-pod -- nginx -v
```

#### Sessão interativa
```bash
# Abrir shell bash interativo
kubectl exec -it test-pod -- bash

# Abrir shell sh se bash não disponível
kubectl exec -it test-pod -- sh

# Executar comando específico interativamente
kubectl exec -it test-pod -- /bin/bash
```

#### Pods com múltiplos containers
```bash
# Especificar container específico
kubectl exec -it multi-pod -c nginx-container -- bash
kubectl exec -it multi-pod -c app-container -- sh
```

### Flags Importantes
- `-i, --stdin`: Manter STDIN aberto
- `-t, --tty`: Alocar pseudo-TTY
- `-c, --container`: Especificar container (obrigatório para pods multi-container)

## kubectl attach

Conecta-se ao processo principal (PID 1) de um container em execução.

### Sintaxe Básica
```bash
kubectl attach [OPTIONS] POD_NAME
kubectl attach [OPTIONS] POD_NAME -c CONTAINER_NAME
```

### Exemplos Práticos

#### Conectar ao processo principal
```bash
# Anexar ao processo principal do pod
kubectl attach test-pod

# Anexar com TTY interativo
kubectl attach -it test-pod

# Anexar a container específico
kubectl attach -it multi-pod -c nginx-container
```

### Flags Importantes
- `-i, --stdin`: Passar STDIN para o container
- `-t, --tty`: Alocar pseudo-TTY
- `-c, --container`: Especificar container

## Diferenças Principais

| Aspecto | kubectl exec | kubectl attach |
|---------|--------------|----------------|
| **Função** | Executa novos comandos | Conecta ao processo principal |
| **Processo** | Cria novo processo | Anexa ao PID 1 existente |
| **Uso comum** | Debug, manutenção | Monitorar logs em tempo real |
| **Interatividade** | Shell interativo | Saída do processo principal |

## Casos de Uso

### kubectl exec
- **Debug de aplicações**: Investigar problemas dentro do container
- **Manutenção**: Executar comandos de limpeza ou configuração
- **Desenvolvimento**: Testar comandos antes de incluir no Dockerfile
- **Troubleshooting**: Verificar configurações e arquivos

### kubectl attach
- **Monitoramento**: Ver logs em tempo real do processo principal
- **Aplicações interativas**: Conectar a aplicações que esperam input
- **Debug de inicialização**: Acompanhar processo de startup

## Exemplos Avançados

### Executar múltiplos comandos
```bash
# Executar sequência de comandos
kubectl exec test-pod -- sh -c "cd /var/log && ls -la && tail nginx/access.log"

# Executar script
kubectl exec test-pod -- sh -c "
  echo 'Verificando sistema...'
  df -h
  free -m
  ps aux | head -10
"
```

### Copiar arquivos (alternativa ao kubectl cp)
```bash
# Criar arquivo no container
kubectl exec test-pod -- sh -c "echo 'Hello World' > /tmp/test.txt"

# Ler arquivo do container
kubectl exec test-pod -- cat /tmp/test.txt
```

### Monitorar logs em tempo real
```bash
# Anexar para ver logs do processo principal
kubectl attach -it test-pod

# Alternativa com exec para tail de logs
kubectl exec -it test-pod -- tail -f /var/log/nginx/access.log
```

## Diagrama de Funcionamento

```
┌─────────────────────────────────────────────────────────────┐
│                        kubectl exec                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Cliente kubectl ──────────────► API Server                │
│                                      │                      │
│                                      ▼                      │
│                                  kubelet                    │
│                                      │                      │
│                                      ▼                      │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                Container                            │    │
│  │  ┌─────────────┐    ┌─────────────┐                │    │
│  │  │   PID 1     │    │  Novo       │ ◄── exec       │    │
│  │  │ (processo   │    │  Processo   │                │    │
│  │  │ principal)  │    │ (comando)   │                │    │
│  │  └─────────────┘    └─────────────┘                │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                       kubectl attach                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Cliente kubectl ──────────────► API Server                │
│                                      │                      │
│                                      ▼                      │
│                                  kubelet                    │
│                                      │                      │
│                                      ▼                      │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                Container                            │    │
│  │  ┌─────────────┐                                   │    │
│  │  │   PID 1     │ ◄────────────── attach            │    │
│  │  │ (processo   │                                   │    │
│  │  │ principal)  │                                   │    │
│  │  └─────────────┘                                   │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## Boas Práticas

### Segurança
- Use `kubectl exec` apenas quando necessário
- Evite executar como root quando possível
- Monitore sessões exec em produção
- Use RBAC para controlar acesso

### Performance
- Prefira `kubectl logs` para visualizar logs
- Use `kubectl attach` para monitoramento em tempo real
- Evite sessões exec de longa duração

### Troubleshooting
```bash
# Verificar se pod está rodando
kubectl get pods

# Verificar logs antes de usar exec
kubectl logs test-pod

# Verificar eventos do pod
kubectl describe pod test-pod

# Testar conectividade
kubectl exec test-pod -- ping google.com
```

## Comandos Úteis para Debug

```bash
# Informações do sistema
kubectl exec test-pod -- uname -a
kubectl exec test-pod -- cat /etc/os-release

# Rede
kubectl exec test-pod -- netstat -tulpn
kubectl exec test-pod -- ss -tulpn

# Processos
kubectl exec test-pod -- ps aux
kubectl exec test-pod -- top -n 1

# Disco
kubectl exec test-pod -- df -h
kubectl exec test-pod -- du -sh /var/log

# Variáveis de ambiente
kubectl exec test-pod -- env
```

## Limitações

### kubectl exec
- Requer container em execução
- Comando deve existir no container
- Limitado pelas permissões do container

### kubectl attach
- Só funciona com processo principal ativo
- Nem todos os processos aceitam input
- Pode não funcionar com aplicações que redirecionam output

## Conclusão

`kubectl exec` e `kubectl attach` são ferramentas fundamentais para debug e manutenção de aplicações Kubernetes. Use `exec` para executar comandos e `attach` para monitorar processos principais.
