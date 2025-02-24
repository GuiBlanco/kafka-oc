# Implantação do Kafka no OpenShift usando KRaft (sem Zookeeper)

Este repositório contém manifestos Kubernetes para implantar o Apache Kafka no OpenShift usando imagens Bitnami e o modo KRaft (sem Zookeeper), configurado para um único pod com suporte a SSL e rotas expostas.

## Componentes

- Kafka (1 broker)
- Armazenamento persistente para Kafka
- Serviço headless para Kafka
- Configuração SSL para comunicação segura
- Routes OpenShift para acesso externo

## Pré-requisitos

- Acesso ao cluster OpenShift
- Ferramenta de linha de comando `oc` instalada
- Permissões apropriadas para criar recursos em seu namespace
- Secret 'kafka-certificates' contendo:
  - keystore.jks
  - truststore.jks
  - Senhas configuradas como "secret"

## Instruções de Implantação

1. Crie um novo projeto (se necessário):
```bash
oc new-project kafka-project
```

2. Implante todos os recursos:
```bash
oc apply -f kafka-configmap.yaml,kafka-statefulset.yaml,kafka-service.yaml,kafka-pvc.yaml,kafka-route.yaml
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

3. Verifique as rotas expostas:
```bash
oc get routes
```

4. Teste o cluster Kafka com SSL:
```bash
# Crie um tópico de teste usando SSL
oc exec kafka-0 -- bin/kafka-topics.sh --create --topic test-topic --bootstrap-server localhost:9094 \
  --command-config /opt/bitnami/kafka/config/certs/client-ssl.properties \
  --partitions 1 --replication-factor 1

# Liste os tópicos usando SSL
oc exec kafka-0 -- bin/kafka-topics.sh --list --bootstrap-server localhost:9094 \
  --command-config /opt/bitnami/kafka/config/certs/client-ssl.properties
```

## Detalhes de Configuração

- Versão do Kafka: 3.6.1
- Portas do Kafka:
  - 9092 (PLAINTEXT - interno)
  - 9093 (CONTROLLER)
  - 9094 (SSL)
- Armazenamento: 10Gi para o broker
- SSL ativado na porta 9094
- Routes configuradas:
  - kafka-broker: SSL (passthrough)
  - kafka-broker-internal: PLAINTEXT
  - kafka-controller: CONTROLLER

## Configuração SSL

Esta implantação usa SSL para comunicação segura:
1. Certificados montados do secret 'kafka-certificates'
2. Keystore e Truststore configurados com senha "secret"
3. SSL ativado na porta 9094
4. Route configurado com SSL passthrough para acesso externo seguro

### Exemplo de client-ssl.properties
```properties
security.protocol=SSL
ssl.truststore.location=/opt/bitnami/kafka/config/certs/truststore.jks
ssl.truststore.password=secret
ssl.keystore.location=/opt/bitnami/kafka/config/certs/keystore.jks
ssl.keystore.password=secret
```

## Rotas Expostas

1. kafka-broker: Acesso SSL externo
   - Porta: 9094
   - TLS: Passthrough
2. kafka-broker-internal: Acesso PLAINTEXT interno
   - Porta: 9092
3. kafka-controller: Acesso ao controller
   - Porta: 9093

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
- Erros de configuração SSL
- Problemas com certificados

### Verificação de Conectividade SSL
Para testar a conectividade SSL:
```bash
openssl s_client -connect <kafka-broker-route-hostname>:443 -tls1_2
```

## Limpeza

Para remover todos os recursos:
```bash
oc delete -f kafka-statefulset.yaml,kafka-service.yaml,kafka-configmap.yaml,kafka-pvc.yaml,kafka-route.yaml
```

Observação: O PersistentVolumeClaim precisará ser excluído separadamente se você quiser remover todos os dados.
