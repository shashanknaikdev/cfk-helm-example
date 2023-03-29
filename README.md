# cfk-helm-acls
Example of using helm charts and a k8s job to create ACLs for dependent components such as Schema Registry, C3, ksqlDB, Connect, etc. This approach can be incorporated into a gitOps flow as well

### Pre-Steps:
* Set the tutorial directory under the directory you downloaded this Github repo:
```
export TUTORIAL_HOME=<Github repo directory>
```

* Deploy Confluent for Kubernetes
This workflow scenario assumes you are using the namespace confluent.

Set up the Helm Chart:
```
helm repo add confluentinc https://packages.confluent.io/helm
```

Install Confluent For Kubernetes using Helm:
```
helm upgrade --install operator confluentinc/confluent-for-kubernetes -n confluent
```

Check that the Confluent For Kubernetes pod comes up and is running:
```
kubectl get pods
```

* Provide a Certificate Authority

Confluent For Kubernetes provides auto-generated certificates for Confluent Platform components to use for TLS network encryption. You'll need to generate and provide a Certificate Authority (CA).

Generate a CA pair to use:
```
openssl genrsa -out $TUTORIAL_HOME/ca-key.pem 2048

openssl req -new -key $TUTORIAL_HOME/ca-key.pem -x509 \
  -days 1000 \
  -out $TUTORIAL_HOME/ca.pem \
  -subj "/C=US/ST=CA/L=MountainView/O=Confluent/OU=Operator/CN=TestCA"
```

Create a Kubernetes secret for the certificate authority:
```
kubectl create secret tls ca-pair-sslcerts \
  --cert=$TUTORIAL_HOME/ca.pem \
  --key=$TUTORIAL_HOME/ca-key.pem -n confluent
```

### Instructions to start-up cluster:
* kubectl create secret tls ca-pair-sslcerts \
  --cert=$TUTORIAL_HOME/ca.pem \  
  --key=$TUTORIAL_HOME/ca-key.pem -n confluent
* helm install confluent-core helm-cfk-core -n confluent
* helm install confluent-acl helm-cfk-acls -n confluent
* helm install confluent-addons helm-cfk-addons -n confluent

### Instructions to tear-down cluster:

* helm delete confluent-core -n confluent
* helm delete confluent-acl -n confluent
* helm delete confluent-addons -n confluent


### Client config for testing
```
kubectl exec -it kafka-0 -- sh  

cat <<EOF>/tmp/client.properties 
bootstrap.servers=kafka.confluent.svc.cluster.local:9071 
security.protocol=SSL 
ssl.truststore.location=/mnt/sslcerts/truststore.jks 
ssl.truststore.password=mystorepassword 
ssl.keystore.location=/mnt/sslcerts/keystore.jks 
ssl.keystore.password=mystorepassword
EOF

kafka-topics --list --bootstrap-server kafka:9071 --command-config /tmp/client.properties
```
