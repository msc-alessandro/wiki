## Definição

Execução dos testes de desempenho usando o protocola AMQP juntamente com o RabbitMQ em um cluster do Kubernetes.

## Ferramentas

*  RabbitMQ
*  Java
*  RabbitMQ Perf Test
*  Python 3

## Configuração do Cluster

```
kind: Service
apiVersion: v1
metadata:
  namespace: test-rabbitmq
  name: rabbitmq
  labels:
    app: rabbitmq
    type: LoadBalancer
spec:
  type: NodePort
  externalIPs:
    - 192.168.25.2
  ports:
   - name: http
     protocol: TCP
     port: 15672
     targetPort: 15672
   - name: amqp
     protocol: TCP
     port: 5672
     targetPort: 5672
   - name: mqtt
     protocol: TCP
     port: 1883
     targetPort: 1883
  selector:
    app: rabbitmq
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: rabbitmq-config
  namespace: test-rabbitmq
data:
  enabled_plugins: |
      [rabbitmq_management,rabbitmq_peer_discovery_k8s, rabbitmq_mqtt].
  rabbitmq.conf: |
      ## Cluster formation. See https://www.rabbitmq.com/cluster-formation.html to learn more.
      cluster_formation.peer_discovery_backend  = rabbit_peer_discovery_k8s
      cluster_formation.k8s.host = kubernetes.default.svc.cluster.local
      ## Should RabbitMQ node name be computed from the pod's hostname or IP address?
      ## IP addresses are not stable, so using [stable] hostnames is recommended when possible.
      ## Set to "hostname" to use pod hostnames.
      ## When this value is changed, so should the variable used to set the RABBITMQ_NODENAME
      ## environment variable.
      cluster_formation.k8s.address_type = hostname
      ## How often should node cleanup checks run?
      cluster_formation.node_cleanup.interval = 30
      ## Set to false if automatic removal of unknown/absent nodes
      ## is desired. This can be dangerous, see
      ##  * https://www.rabbitmq.com/cluster-formation.html#node-health-checks-and-cleanup
      ##  * https://groups.google.com/forum/#!msg/rabbitmq-users/wuOfzEywHXo/k8z_HWIkBgAJ
      cluster_formation.node_cleanup.only_log_warning = true
      cluster_partition_handling = autoheal
      ## See https://www.rabbitmq.com/ha.html#master-migration-data-locality
      queue_master_locator=min-masters
      ## This is just an example.
      ## This enables remote access for the default user with well known credentials.
      ## Consider deleting the default user and creating a separate user with a set of generated
      ## credentials instead.
      ## Learn more at https://www.rabbitmq.com/access-control.html#loopback-users
      loopback_users.guest = false
      mqtt.listeners.tcp.default = 1883
      mqtt.default_user = rabbit
      mqtt.default_pass = s3kRe7
      mqtt.vhost            = /
      mqtt.exchange         = amq.topic
      # 24 hours by default
      mqtt.subscription_ttl = 86400000
      mqtt.prefetch         = 10
---
apiVersion: apps/v1
# See the Prerequisites section of https://www.rabbitmq.com/cluster-formation.html#peer-discovery-k8s.
kind: StatefulSet
metadata:
  name: rabbitmq
  namespace: test-rabbitmq
spec:
  selector:
    matchLabels:
      app: rabbitmq
  serviceName: rabbitmq
  # Three nodes is the recommended minimum. Some features may require a majority of nodes
  # to be available.
  replicas: 1
  template:
    metadata:
      labels:
        app: rabbitmq
    spec:
      serviceAccountName: rabbitmq
      terminationGracePeriodSeconds: 10
      nodeSelector:
        # Use Linux nodes in a mixed OS kubernetes cluster.
        # Learn more at https://kubernetes.io/docs/reference/kubernetes-api/labels-annotations-taints/#kubernetes-io-os
        kubernetes.io/hostname: bahamut
      containers:
      - name: rabbitmq-k8s
        image: rabbitmq:3.8
        volumeMounts:
          - name: config-volume
            mountPath: /etc/rabbitmq
        # Learn more about what ports various protocols use
        # at https://www.rabbitmq.com/networking.html#ports
        ports:
          - name: http
            protocol: TCP
            containerPort: 15672
          - name: amqp
            protocol: TCP
            containerPort: 5672
        livenessProbe:
          exec:
            # This is just an example. There is no "one true health check" but rather
            # several rabbitmq-diagnostics commands that can be combined to form increasingly comprehensive
            # and intrusive health checks.
            # Learn more at https://www.rabbitmq.com/monitoring.html#health-checks.
            #
            # Stage 2 check:
            command: ["rabbitmq-diagnostics", "status"]
          initialDelaySeconds: 60
          # See https://www.rabbitmq.com/monitoring.html for monitoring frequency recommendations.
          periodSeconds: 60
          timeoutSeconds: 15
        readinessProbe:
          exec:
            # This is just an example. There is no "one true health check" but rather
            # several rabbitmq-diagnostics commands that can be combined to form increasingly comprehensive
            # and intrusive health checks.
            # Learn more at https://www.rabbitmq.com/monitoring.html#health-checks.
            #
            # Stage 2 check:
            command: ["rabbitmq-diagnostics", "status"]
            # To use a stage 4 check:
            # command: ["rabbitmq-diagnostics", "check_port_connectivity"]
          initialDelaySeconds: 20
          periodSeconds: 60
          timeoutSeconds: 10
        imagePullPolicy: Always
        env:
          - name: MY_POD_NAME
            valueFrom:
              fieldRef:
                apiVersion: v1
                fieldPath: metadata.name
          - name: MY_POD_NAMESPACE
            valueFrom:
              fieldRef:
                fieldPath: metadata.namespace
          - name: RABBITMQ_USE_LONGNAME
            value: "true"
          # See a note on cluster_formation.k8s.address_type in the config file section
          - name: K8S_SERVICE_NAME
            value: rabbitmq
          - name: RABBITMQ_NODENAME
            value: rabbit@$(MY_POD_NAME).$(K8S_SERVICE_NAME).$(MY_POD_NAMESPACE).svc.cluster.local
          - name: K8S_HOSTNAME_SUFFIX
            value: .$(K8S_SERVICE_NAME).$(MY_POD_NAMESPACE).svc.cluster.local
          - name: RABBITMQ_ERLANG_COOKIE
            value: "mycookie"
      volumes:
        - name: config-volume
          configMap:
            name: rabbitmq-config
            items:
            - key: rabbitmq.conf
              path: rabbitmq.conf
            - key: enabled_plugins
              path: enabled_plugins
```

## Estado Inicial do Cluster

![cluster-inicial](uploads/6d57993f2f039b4d91ec43b068406c48/cluster-inicial.png)

### Teste 1

#### Configuração do Teste 1
```
[
    {
        'name': 'publish_consume_low',
        'type': 'simple',
        'uri': 'amqp://guest:guest@10.105.244.74:5672',
        'params': [{
            'time-limit': 180,
            'producer-count': 10,
            'consumer-count': 10
        }]
    }
]
```


#### Resultados

![rabbitmq-test1](uploads/da912857e7b77d47534a0a23d8cd27ef/rabbitmq-test1.png)


### Teste 2

#### Configuração do Teste 2
```
[
    {
        'name': 'publish_consume_medium',
        'type': 'simple',
        'uri': 'amqp://guest:guest@10.105.244.74:5672',
        'params': [{
            'time-limit': 180,
            'producer-count': 100,
            'consumer-count': 100
        }]

    }
]
```

#### Resultados

![rabbitmq-test2](uploads/b1a65833e4549e27f37d9b7d7f293deb/rabbitmq-test2.png)

### Teste 3

#### Configuração do Teste 3

```
[
    {
        'name':      'message-sizes-large',
        'type':      'varying',
        'uri': 'amqp://guest:guest@10.105.244.74:5672',
        'params':    [{
            'time-limit': 30
        }],
        'variables': [{
            'name':   'min-msg-size',
            'values': [
                5000,
                10000,
                50000,
                100000,
                500000,
                1000000
            ]
        }]
    },
    {
        'name':      'rate-vs-latency',
        'type':      'rate-vs-latency',
        'uri': 'amqp://guest:guest@10.105.244.74:5672',
        'params':    [{
            'time-limit': 30
        }]
    }
]
```

#### Resultados

![rabbitmq-test3](uploads/a1954901e6842a7671bbee63e2a6a9a2/rabbitmq-test3.png)