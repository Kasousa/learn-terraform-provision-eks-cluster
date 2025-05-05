# EKS - STRIMZI- KAFKA - Data Dog

# 1 - Subir a infraestrutura do EKS
- terraform init 
- terraform plan
- terraform apply -auto-approve

# 2 - Validar a subida da infraestrutura
- aws eks update-kubeconfig --region sa-east-1 --name kquesso-cluster-XXXXX
- kubectl get nodes

# 3 - Subir o Kafka utilizando o Strimzi Cluster Operator (https://strimzi.io/quickstarts/)
- kubectl create namespace kafka
- kubectl create -f 'https://strimzi.io/install/latest?namespace=kafka' -n kafka
- kubectl get pods -n kafka -w
- kubectl apply -f kafka-single-node.yaml -n kafka 
- kubectl get pods -n kafka -w

# 4 - Configurando consumer e producer
- kubectl -n kafka run kafka-producer -ti --image=quay.io/strimzi/kafka:0.46.0-kafka-4.0.0 --rm=true --restart=Never -- bin/kafka-console-producer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic helloKafka
- kubectl -n kafka run kafka-consumer -ti --image=quay.io/strimzi/kafka:0.46.0-kafka-4.0.0 --rm=true --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic helloKafka --from-beginning

# 5 - Configurando a instrumentação via datadog
- helm repo add datadog https://helm.datadoghq.com
- helm install datadog-operator datadog/datadog-operator
- kubectl create secret generic datadog-secret --from-literal api-key=46184dde2681f7a6c3cfb471d6d16137
- kubectl apply -f datadog-agent.yaml

# 6 - Subindo app java para expor o kafka publicamente
- kubectl apply kafkaproducer.yaml