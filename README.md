# strimzi-kafka-k8s
Template files for kafka connect(ors) on eks/ubuntu-vm 

# Using Kafka-cli

```
https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/quickstart/kafka-cli
```


      # Env Variables

      AZURE_LOCATION=East
      AZURE_SUBSCRIPTION=MYSUBCRIPTION-AZR
      OLDPWD=/root/strimzi-kafka-connect-eventhubs
      KAFKACAT_CONFIG=/root/strimzi-kafka-connect-eventhubs/kafkacat.conf
      KAFKA_OPTS=-Djava.security.auth.login.config=jaas.conf
      PATH=/root/.krew/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/root/strimzi-kafka-connect-eventhubs/kafkacat.conf:/root/.config/
      AZURE_RESOURCE_GROUP=EHRG
      KAFKA_INSTALL_HOME=/usr/local/bin/kafka_2.11-2.3.0/
      EVENT_HUB_NAME=myeventhub
      EVENT_HUBS_NAMESPACE=test



      # To Feed/produce Kafka event to hub.


      $KAFKA_INSTALL_HOME/bin/kafka-console-producer.sh --topic 433 --broker-list $EVENT_HUBS_NAMESPACE.servicebus.windows.net:9093 --producer.config client_common.properties


      Enter Input as below:

      {"schema":{"type":"string","optional":false},"payload":"eventhubtest"}


      # To consume events from kafka hub

      $KAFKA_INSTALL_HOME/bin/kafka-console-consumer.sh --topic 433 --bootstrap-server $EVENT_HUBS_NAMESPACE.servicebus.windows.net:9093 --consumer.config client_common.properties



# Using K8S with eventhub as kafka cluster

    # Install strimzi cluster operator
    //add helm chart repo for Strimzi
    helm repo add strimzi https://strimzi.io/charts/
    //install it! (I have used strimzi-kafka as the release name)
    helm install strimzi-kafka strimzi/strimzi-kafka-operator
    
    # strimzi cluster operator

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      annotations:
        deployment.kubernetes.io/revision: "1"
        meta.helm.sh/release-name: strimzi-kafka
        meta.helm.sh/release-namespace: default
      creationTimestamp: null
      generation: 1
      labels:
        app: strimzi
        app.kubernetes.io/managed-by: Helm
        chart: strimzi-kafka-operator-0.20.1
        component: deployment
        heritage: Helm
        release: strimzi-kafka
      name: strimzi-cluster-operator
      selfLink: /apis/apps/v1/namespaces/default/deployments/strimzi-cluster-operator
    spec:
      progressDeadlineSeconds: 600
      replicas: 1
      revisionHistoryLimit: 10
      selector:
        matchLabels:
          name: strimzi-cluster-operator
          strimzi.io/kind: cluster-operator
      strategy:
        type: Recreate
      template:
        metadata:
          creationTimestamp: null
          labels:
            name: strimzi-cluster-operator
            strimzi.io/kind: cluster-operator
        spec:
          containers:
          - args:
            - /opt/strimzi/bin/cluster_operator_run.sh
            env:
            - name: STRIMZI_NAMESPACE
              valueFrom:
                fieldRef:
                  apiVersion: v1
                  fieldPath: metadata.namespace
            - name: STRIMZI_FULL_RECONCILIATION_INTERVAL_MS
              value: "120000"
            - name: STRIMZI_OPERATION_TIMEOUT_MS
              value: "300000"
            - name: STRIMZI_DEFAULT_TLS_SIDECAR_ENTITY_OPERATOR_IMAGE
              value: strimzi/kafka:0.20.1-kafka-2.6.0
            - name: STRIMZI_DEFAULT_KAFKA_EXPORTER_IMAGE
              value: strimzi/kafka:0.20.1-kafka-2.6.0
            - name: STRIMZI_DEFAULT_CRUISE_CONTROL_IMAGE
              value: strimzi/kafka:0.20.1-kafka-2.6.0
            - name: STRIMZI_DEFAULT_TLS_SIDECAR_CRUISE_CONTROL_IMAGE
              value: strimzi/kafka:0.20.1-kafka-2.6.0
            - name: STRIMZI_KAFKA_IMAGES
              value: |
                2.5.0=strimzi/kafka:0.20.1-kafka-2.5.0
                2.5.1=strimzi/kafka:0.20.1-kafka-2.5.1
                2.6.0=strimzi/kafka:0.20.1-kafka-2.6.0
            - name: STRIMZI_KAFKA_CONNECT_IMAGES
              value: |
                2.5.0=strimzi/kafka:0.20.1-kafka-2.5.0
                2.5.1=strimzi/kafka:0.20.1-kafka-2.5.1
                2.6.0=strimzi/kafka:0.20.1-kafka-2.6.0
            - name: STRIMZI_KAFKA_CONNECT_S2I_IMAGES
              value: |
                2.5.0=strimzi/kafka:0.20.1-kafka-2.5.0
                2.5.1=strimzi/kafka:0.20.1-kafka-2.5.1
                2.6.0=strimzi/kafka:0.20.1-kafka-2.6.0
            - name: STRIMZI_KAFKA_MIRROR_MAKER_IMAGES
              value: |
                2.5.0=strimzi/kafka:0.20.1-kafka-2.5.0
                2.5.1=strimzi/kafka:0.20.1-kafka-2.5.1
                2.6.0=strimzi/kafka:0.20.1-kafka-2.6.0
            - name: STRIMZI_KAFKA_MIRROR_MAKER_2_IMAGES
              value: |
                2.5.0=strimzi/kafka:0.20.1-kafka-2.5.0
                2.5.1=strimzi/kafka:0.20.1-kafka-2.5.1
                2.6.0=strimzi/kafka:0.20.1-kafka-2.6.0
            - name: STRIMZI_DEFAULT_TOPIC_OPERATOR_IMAGE
              value: strimzi/operator:0.20.1
            - name: STRIMZI_DEFAULT_USER_OPERATOR_IMAGE
              value: strimzi/operator:0.20.1
            - name: STRIMZI_DEFAULT_KAFKA_INIT_IMAGE
              value: strimzi/operator:0.20.1
            - name: STRIMZI_DEFAULT_KAFKA_BRIDGE_IMAGE
              value: strimzi/kafka-bridge:0.19.0
            - name: STRIMZI_DEFAULT_JMXTRANS_IMAGE
              value: strimzi/jmxtrans:0.20.1
            image: strimzi/operator:0.20.1
            imagePullPolicy: IfNotPresent
            livenessProbe:
              failureThreshold: 3
              httpGet:
                path: /healthy
                port: http
                scheme: HTTP
              initialDelaySeconds: 10
              periodSeconds: 30
              successThreshold: 1
              timeoutSeconds: 1
            name: strimzi-cluster-operator
            ports:
            - containerPort: 8080
              name: http
              protocol: TCP
            readinessProbe:
              failureThreshold: 3
              httpGet:
                path: /ready
                port: http
                scheme: HTTP
              initialDelaySeconds: 10
              periodSeconds: 30
              successThreshold: 1
              timeoutSeconds: 1
            resources:
              limits:
                cpu: "1"
                memory: 384Mi
              requests:
                cpu: 200m
                memory: 384Mi
            terminationMessagePath: /dev/termination-log
            terminationMessagePolicy: File
            volumeMounts:
            - mountPath: /opt/strimzi/custom-config/
              name: co-config-volume
          dnsPolicy: ClusterFirst
          restartPolicy: Always
          schedulerName: default-scheduler
          securityContext: {}
          serviceAccount: strimzi-cluster-operator
          serviceAccountName: strimzi-cluster-operator
          terminationGracePeriodSeconds: 30
          volumes:
          - configMap:
              defaultMode: 420
              name: strimzi-cluster-operator
            name: co-config-volume
    status: {}


    # Kafkaconnect CRD


    apiVersion: kafka.strimzi.io/v1beta1
    kind: KafkaConnect
    metadata:
      annotations:
        strimzi.io/use-connector-resources: "true"
      generation: 1
      name: my-sink-cluster
      selfLink: /apis/kafka.strimzi.io/v1beta1/namespaces/default/kafkaconnects/my-sink-cluster
    spec:
      authentication:
        passwordSecret:
          password: eventhubspassword
          secretName: eventhubssecret
        type: plain
        username: $ConnectionString
      bootstrapServers: test.servicebus.windows.net:9093
      config:
        config.storage.topic: sink-strimzi-connect-cluster-configs
        group.id: sink-cluster
        offset.storage.topic: sink-strimzi-connect-cluster-offsets
        status.storage.topic: sink-strimzi-connect-cluster-status
      logging:
        name: connect-logging-configmap
        type: external
      replicas: 1
      tls:
        trustedCertificates: []
      version: 2.5.0



    # Kafka connector CRD for corresponding KafkaConnect defined above.


    apiVersion: kafka.strimzi.io/v1alpha1
    kind: KafkaConnector
    metadata:
      annotations:
      generation: 1
      labels:
        strimzi.io/cluster: my-sink-cluster
      name: sink-connector
      selfLink: /apis/kafka.strimzi.io/v1alpha1/namespaces/default/kafkaconnectors/sink-connector
    spec:
      class: org.apache.kafka.connect.file.FileStreamSinkConnector
      config:
        file: /tmp/sink-worker.log
        topics: 433
      tasksMax: 1
    
 # References
 https://github.com/abhirockzz/strimzi-kafka-connect-eventhubs
 
 https://mirror.its.dal.ca/apache/kafka/2.3.0/kafka_2.11-2.3.0.tgz
