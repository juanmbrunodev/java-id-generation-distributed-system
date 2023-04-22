# Kafka Kubernetes and Docker Cluster

This folder contains the necessary docker and kubernetes manifest files to spin a cluster locally in
aims of running the system from a developer machine.

That said, the K8s manifest files can be used (if changed when needed for productionizing) to run the
cluster in other environments like cloud, etc.

## Docker Compose Local Run

To run the Kafka (and Zookeeper) cluster locally, the following command can be run, provided Docker
is installed locally and running:

```
docker-compose up [-d] 
```

Where, [-d] can run the Docker process that spins up the cluster in "daemon" or background mode.

To check that the cluster is running the following command can be run:

```
docker-compose ps
```

The desired output informing the cluster is indeed running should look like the following:

```
  Name               Command            State                     Ports                   
------------------------------------------------------------------------------------------
kafka       /etc/confluent/docker/run   Up      0.0.0.0:9092->9092/tcp                    
zookeeper   /etc/confluent/docker/run   Up      0.0.0.0:2181->2181/tcp, 2888/tcp, 3888/tcp
...

```

**To connect to the docker compose cluster from the host machine (i.e. dev machine) the following can be run:**

```
kafka-topics --bootstrap-server localhost:29092,localhost:39092 --list
```

Two ports are specified as this docker-compose Kafka cluster is a multi-node one.


_Kafka has to be installed locally in the host machine for this command to work_


## Running the Kafka Cluster in Kubernetes

Because docker-compose is ephimeral, when the command `docker-compose down` is run to destroy the local cluster, so do
the topics and all information stored on them. An alternative to avoid this is to use a kubernetes cluster, which this section
explains how to create.

_This next part of the documentation assumes there is a Kubernetes engine running locally (minikube
is a recommendation, and also assumed to be installed)._

First, start minikube by running:

```
minikube start
```

Then, in the /manifests directory, the following commands can be run to create all the necessary K8s entities:

1- Create the namespace:

```
kubectl apply -f namespace.yaml
```

This can be verified by running:

```
kubectl get namespaces
```

The _kafka_ namespace should now be present.

2- Deploying Zookeeper, by running the following command:

```
kubectl apply -f zookeeper.yaml
```

To check the deployment and service were created, we can run the following command:

```
kubectl get services -n kafka
```

This will output a value for the CLUSTER-IP column, which has to be replaced in the _kafka.yaml_ file, line
37.

3- Deploying the Kafka Broker, by running the following command:

```
kubectl apply -f kafka.yaml
```

4- We can verify both services, and corresponding pods, are running, by executing the following command, which
lists the pods for the _kafka_ namespace we created in step 1:

```
kubectl get pods -n kafka
```

5- Notice the line in kafka.yaml where we provide a value for KAFKA_ADVERTISED_LISTENERS. To ensure that Zookeeper 
and Kafka can communicate by using this hostname (kafka-broker), we need to add the following entry to the _/etc/hosts_ 
file on our local machine:

```
127.0.0.1 kafka-broker
```

Now the kafka broker service can be exposed by forwarding its port. With the command above to get the pods, we can use the
id of the pod and replace in below command:

```
kubectl port-forward [pod_id] 9092 -n kafka
```

If all goes well, we should see something like the following:

```
Forwarding from 127.0.0.1:9092 -> 9092
Forwarding from [::1]:9092 -> 9092
```

Once port is forwarded to kafka broker, you can list the topics in the Kafka cluster using the kafka-topics command 
as follows:

```
kafka-topics --bootstrap-server localhost:9092 --list
```

(Kafka has to be installed locally for whatever OS you are running, in Mac the following should suffice:
`brew install kafka`)

The cluster on Minikube (Kubernetes local) is now ready to be used. For example, a topic can be created running the following
command:

```
kafka-topics --bootstrap-server localhost:9092 --create --topic topic_test
```

**WARNING**: Both docker-compose and minikube kubernetes cluster are, persistent wise, ephemeral. This means when the clusters
are destroyed, the topics and contents are also erased. To address this in Kubernetes, a stateful set can be used, but this outside
the scope of this demo project.



