# Implantação do Kafka no OpenShift usando KRaft (sem Zookeeper)

Este repositório contém manifestos Kubernetes para implantar o Apache Kafka no OpenShift usando imagens Bitnami e o modo KRaft (sem Zookeeper), configurado para um único pod.

## Componentes

- Kafka (1 broker)
- Armazenamento persistente para Kafka
- Serviço headless para Kafka

## Pré-requisitos

- Acesso ao cluster OpenShift
- Ferramenta de linha de comando `oc` instalada
- Permissões apropriadas para criar recursos em seu namespace

## Instruções de Implantação

1. Crie um novo projeto (se necessário):
```bash
oc new-project kafka-project
```

2. Implante a configuração do Kafka:
```bash
oc apply -f kafka-configmap.yaml
```

3. Crie os serviços do Kafka:
```bash
oc apply -f kafka-service.yaml
```

4. Implante o broker do Kafka:
```bash
oc apply -f kafka-statefulset.yaml
```

## Verificação

1. Verifique se o pod está em execução:
```bash
oc get pods
```

2. Verifique os logs do broker Kafka:
```bash
oc logs kafka-0
```

3. Teste o cluster Kafka:
```bash
# Crie um tópico de teste
oc exec kafka-0 -- bin/kafka-topics.sh --create --topic test-topic --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1

# Liste os tópicos
oc exec kafka-0 -- bin/kafka-topics.sh --list --bootstrap-server localhost:9092
```

## Detalhes de Configuração

- Versão do Kafka: 3.6.1
- Portas do Kafka: 9092 (PLAINTEXT), 9093 (CONTROLLER)
- Armazenamento: 10Gi para o broker

## Considerações de Segurança

Esta implantação usa listeners PLAINTEXT para simplicidade. Para ambientes de produção, você deve:
1. Habilitar autenticação
2. Configurar criptografia TLS
3. Configurar mecanismos SASL apropriados
4. Implementar políticas de segurança adequadas

## Manutenção

### Atualização da Versão do Kafka
Para atualizar a versão do Kafka, modifique a tag da imagem no arquivo kafka-statefulset.yaml e aplique as alterações:
```bash
oc apply -f kafka-statefulset.yaml
```

### Monitoramento
Considere implementar monitoramento usando:
- Prometheus
- Grafana
- JMX Exporter

## Solução de Problemas

1. Se o pod não estiver iniciando:
```bash
oc describe pod kafka-0
```

2. Verifique os logs do broker:
```bash
oc logs kafka-0
```

3. Problemas comuns:
- Problemas de provisionamento de armazenamento
- Problemas de conectividade de rede
- Erros de configuração

## Limpeza

Para remover todos os recursos:
```bash
oc delete -f kafka-statefulset.yaml
oc delete -f kafka-service.yaml
oc delete -f kafka-configmap.yaml
```

Observação: O PersistentVolumeClaim precisará ser excluído separadamente se você quiser remover todos os dados.
