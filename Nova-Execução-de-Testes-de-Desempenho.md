## Definição

Execução dos testes de desempenho utilizando o JMeter como ferramenta de Teste e um cluster do Kubernetes com as configurações já listadas anteriormente. 


## Ferramentas

*  JMeter
*  JMeter Plugin Manager
*  JMeter Custom Thread Groups
*  Throughput Shapping Timer


## Configuração do Cluster


```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: default
  labels:
    app: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      restartPolicy: Always
      containers:
      - name: nginx
        image: nginx
        imagePullPolicy: Always
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 80
      nodeSelector:
        kubernetes.io/hostname: bahamut

---

apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    name: nginx
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
      name: http
  selector:
    name: nginx

```

### Considerações

*  1 **pod** com 1 container, não foi escalado horizontalmente.
*  500 milicpu  para o pod.
*  Utilizando a bridge do Docker (~-20% performance).
*  Sem um loadbalancer, como o [MetalLB](https://metallb.universe.tf/) ou [NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/deploy/)


> For a single-threaded program, a cpu usage of 0.1 means that if you could freeze the machine at a random moment in time, and look at what each core is doing, there is a 1 in 10 chance that your single thread is running at that instant. The number of cores on the machine does not affect the meaning of 0.1. For a container with multiple threads, the container's usage is the sum of its thread's usage (per previous definition.) There is no guarantee about which core you run on, and you might run on a different core at different points in your container's lifetime. A cpu limit of 0.1 means that your usage is not allowed to exceed 0.1 for a significant period of time. A cpu request of 0.1 means that the system will try to ensure that you are able to have a cpu usage of at least 0.1, if your thread is not blocking often. 


## Número de Threads

Para calcular o número de threads utilizou-se a seguinte expressão:


**RPS** * **max response time** / **1000** = **Thread Number**

**RPS** = Responses Per Second


Os números utilizados foram:

|Test Number| RPS | Max Response Time (ms) | Threads|
| ------ | ------ | ------ | ------ |
|0| 10 | 100 | 1 |
|1| 10 | 1000 | 10 |
|2| 10 | 10000 | 100 |
|3| 100 | 100 | 10|
|4| 100 | 1000 | 100|
|5| 100 | 10000 | 1000 |
|6| 1000 | 100 | 100|
|7| 1000 | 1000 | 1000|
|8| 1000 | 10000 | 10000|


* Total Execution Time: 200 sec
* Start Up Time: 10 sec
* Shutdown Time: 10 sec
* Hold Load for: 180 sec

### Thread Group

![threads_group](uploads/ce7381961a8f711099509676b70fc382/threads_group.png)


### RPS 10 Curve

![rps_10](uploads/86fa9ca3a4029af7aaf61e99cd7fdd4a/rps_10.png)

### RPS 100 Curve

![rps_100](uploads/d787d62120d58864db853f71f3e105ab/rps_100.png)

### RPS 1000 Curve

![rps_1000](uploads/522abea0cbabc64f87c4957681bc550f/rps_1000.png)

## Execução

### Teste 0

#### Summary

![test_0_summary](uploads/e2d6d6162cff0b59b961b5b2c87ac2e7/test_0_summary.png)

#### Active Threads

![test_0_active_threads](uploads/ae55ead254fe3610f4947d684c565ab8/test_0_active_threads.png)


#### Response Time

![test_0_response_time](uploads/71f2278068e9b4ad1fb89dfdfd64637a/test_0_response_time.png)

#### Transactions

![test_0_transactions](uploads/fb958f17dcac2cbf563d65e904fec645/test_0_transactions.png)

### Teste 1

#### Summary

![test_1_summary](uploads/f3d03685c5a955d2a051d43b5b224d80/test_1_summary.png)

#### Active Threads

![test_1_active_threads](uploads/56fa1fb1feae5fb1719c1188b5283fe1/test_1_active_threads.png)

#### Response Time

![test_1_responso_time](uploads/91719cee0bd07e33b1d76eddc6a26a85/test_1_responso_time.png)

#### Transactions

![test_1_transactions](uploads/9c55fd136235b471e77e7ba202db24bc/test_1_transactions.png)

### Teste 2

#### Summary

![test_2_summary](uploads/6a2c74b33ac51f2e28e9a2f84f61226d/test_2_summary.png)

#### Active Threads

![test_2_active_threads](uploads/e88f0e3b09491d22ce16e90dbee5ee48/test_2_active_threads.png)

#### Response Time

![test_2_response_time](uploads/dee346861d26b57a704f1f9672e599e2/test_2_response_time.png)

#### Transactions

![test_2_transactions](uploads/9d239e57b13b37bd0e6b183c32d770e8/test_2_transactions.png)

### Teste 3

#### Summary

![test_3_summary](uploads/dad6a6681dbd4235e158a02222699597/test_3_summary.png)

#### Active Threads

![test_3_active_threads](uploads/1105049d429680080e3711989912a00c/test_3_active_threads.png)

#### Response Time

![test_3_response_time](uploads/b38e16f58772feb6f0e58b29d3007f3c/test_3_response_time.png)

#### Transactions

![test_3_transactions](uploads/2e5240a719f1361af86f18d6df58b9e7/test_3_transactions.png)

### Teste 4

#### Summary

![test_4_summary](uploads/cee313f8865309959b1566c285f8475e/test_4_summary.png)

#### Active Threads

![test_4_active_threads](uploads/7093163075bbf6421a0e7dc54435eb52/test_4_active_threads.png)

#### Response Time

![test_4_response_time](uploads/17b789f39231bca3c799c51b93b6705d/test_4_response_time.png)

#### Transactions

![test_4_transactions](uploads/44ca92f9f08a8ca8ac4079b7f9004da0/test_4_transactions.png)

### Teste 5


#### Summary

![test_5_summary](uploads/60c386d8546c93ed329995ee625e9441/test_5_summary.png)

#### Active Threads

![test_5_active_threads](uploads/ca567d6cdf748c7d9f7fdface68ed68e/test_5_active_threads.png)

#### Response Time

![test_5_response_times](uploads/de9c5e742ef01aaaa262fa59283e9a10/test_5_response_times.png)

#### Transactions

![test_5_transactions](uploads/965ccdaa9c956b934e445653be4424a8/test_5_transactions.png)

### Teste 6

#### Summary

![test_6_summary](uploads/cc47da1d78f64146e34b1fcf639c025a/test_6_summary.png)

#### Active Threads

![test_6_active_threads](uploads/98c456f92556685ec967838183de287f/test_6_active_threads.png)

#### Response Time

![test_6_response_time](uploads/bfc29cbe279bfc8da6b1369cd5e0b1b6/test_6_response_time.png)

#### Transactions

![test_6_transactions](uploads/1bfd8cc2a42d834c0e09efa5796c8a09/test_6_transactions.png)

### Teste 7

#### Summary

![test_7_summary](uploads/932f86b852a6ba06637836b0e3645456/test_7_summary.png)

#### Active Threads

![test_7_active_threads](uploads/c42a76e6263d7805556f057f8792fb5d/test_7_active_threads.png)


#### Response Time

![test_7_response_time](uploads/2b7da599f99e41e196d3a814d42c82b0/test_7_response_time.png)

#### Transactions

![test_7_transactions](uploads/161706244e490f6269388394cc9442be/test_7_transactions.png)


### Teste 8

#### Summary

![test_8_summary](uploads/8ba1031aa46d916bc567fee3ef5aabc3/test_8_summary.png)

#### Active Threads

![test_8_active_threads](uploads/0cb25c79fc2817c5bb41cb9548c8e178/test_8_active_threads.png)

#### Response Time

![test_8_response_time](uploads/079e8c7fa70d54457f7256ab832800e8/test_8_response_time.png)

#### Transactions

![test_8_transactions](uploads/c8271a22290b89786982f88b92f2d9b3/test_8_transactions.png)