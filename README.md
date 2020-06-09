# kafka-demo
Repository to setup kafka and http kafka bridge


## Create a kafka cluster 

Create a new project after login to Openshift

```
oc new-project kafka
```

Apply the Strimzi install files, including ClusterRoles, ClusterRoleBindings and some Custom Resource Definitions (CRDs).

```
oc apply -f 'https://strimzi.io/install/latest?namespace=kafka'
```

Deploy the Kafka cluster with 1 kafka node and 1 zoo keeper node

```
oc apply -f kafka.yaml
```

Create a producer to send messages to a topic names 'my-topic'

```
oc run kafka-producer -ti --image=strimzi/kafka:0.18.0-kafka-2.5.0 --rm=true --restart=Never -- bin/kafka-console-producer.sh --broker-list my-cluster-kafka-bootstrap:9092 --topic my-topic
```

Create a consumer to recieve messages (in another console) from the topic 'my-topic'

```
oc run kafka-consumer -ti --image=strimzi/kafka:0.18.0-kafka-2.5.0 --rm=true --restart=Never -- bin/kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic my-topic --from-beginning
```

You can see messages you type in producer console in the consumer console. 
This is the basic Kafka producer-consumer deployment.
Use 'Control-C' to exit from the producer-consumer console to terminate the pod. 


## Create a Kafka Bridge

Deploy the kafka bridge

```
oc apply -f kafka-bridge.yaml
```

Deploy the route for kafka bridge

```
oc apply -f route.yaml
```

To get the route url use ,
```
oc get routes
```

Check if bridge is healthy
```
curl -v GET http://<route-url>/healthy
```

If the bridge is reachable through the route, it will return an HTTP response with status code 200 OK but an empty body. Following an example of the output.

```
> GET /healthy HTTP/1.1
> Host: my-bridge.io:80
> User-Agent: curl/7.61.1
> Accept: */*
> 
< HTTP/1.1 200 OK
< content-length: 0
```

Send data to the bridge using http post

```
curl -X POST \
  http://my-bridge.io/topics/my-topic \
  -H 'content-type: application/vnd.kafka.json.v2+json' \
  -d '{
    "records": [
        {
            "key": "key-1",
            "value": "value-1"
        },
        {
            "key": "key-2",
            "value": "value-2"
        }
    ]
}'
```

You can run sender.py to send dummy IoT data to check working of Kafka-HTTP bridge. 

```
python sender.py
```


