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
*  Sem um loadbalancer, como o [MetalLB](https://metallb.universe.tf/) ou [NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/deploy/)


> For a single-threaded program, a cpu usage of 0.1 means that if you could freeze the machine at a random moment in time, and look at what each core is doing, there is a 1 in 10 chance that your single thread is running at that instant. The number of cores on the machine does not affect the meaning of 0.1. For a container with multiple threads, the container's usage is the sum of its thread's usage (per previous definition.) There is no guarantee about which core you run on, and you might run on a different core at different points in your container's lifetime. A cpu limit of 0.1 means that your usage is not allowed to exceed 0.1 for a significant period of time. A cpu request of 0.1 means that the system will try to ensure that you are able to have a cpu usage of at least 0.1, if your thread is not blocking often. 



*  Nessa configuração o roteador estava configurado para DMZ no endpoint em que o NGINX respondia.
*  A bridge do Docker estava desabilitada.
*  Entre cada teste o Deployment foi deletado e recriado para isolar a execução, também foram coletados os logs do NGINX separadamente para cada execução.


#### Tamanho de Envio


```
Size in bytes: 850
Sent bytes:114
Headers size in bytes: 238
Body size in bytes: 612
```

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


* RPS: 10
* Max Response Time (ms): 100
* Threads: 1

#### Summary
![test_0_summary](uploads/3f60c5be005926ecb173061691e2370a/test_0_summary.png)

#### Active Threads

![test_0_threads](uploads/54197c2ad2ae9727b5c97dc7b88b83ba/test_0_threads.png)

#### Response Time

![test_0_response](uploads/05815f230c5300c98ee755c00e203b7d/test_0_response.png)
#### Transactions

![test_0_transactions](uploads/2d21579972fc3a8eeb022f8f9e6cfb59/test_0_transactions.png)

#### Logs

https://pastebin.com/A8agKcHP

### Teste 1

* RPS: 10
* Max Response Time (ms): 1000
* Threads: 10

#### Summary

![test_1_summary](uploads/70dd29935b5ebffc1389120ebeb0df26/test_1_summary.png)

#### Active Threads

![test_1_threads](uploads/e052faafe82068f42a844597d783d305/test_1_threads.png)
#### Response Time

![test_1_response](uploads/cc3e782750d709bcc1befaf28658342f/test_1_response.png)
#### Transactions

![test_1_transactions](uploads/35bcbd34e613376bc976fb2b69803531/test_1_transactions.png)

#### Logs

https://pastebin.com/kgFHs7uu

### Teste 2

* RPS: 10
* Max Response Time (ms): 10000
* Threads: 100

#### Summary

![test_2_summary](uploads/d95c28c9243f22aebeb09fbd6c855507/test_2_summary.png)

#### Active Threads

![test_2_threads](uploads/e7b7e636f16eec1399385aecb10e7ab8/test_2_threads.png)
#### Response Time
![test_2_response](uploads/999a7b1104085012f303e794f82b7cb6/test_2_response.png)

#### Transactions
![test_2_transactions](uploads/bcc30ee57ac81a7843e07be55aa14414/test_2_transactions.png)

#### Logs
https://pastebin.com/pESj8v06

### Teste 3

* RPS: 100
* Max Response Time (ms): 100
* Threads: 10


#### Summary
![test_3_summary](uploads/6d1b7f9c08a9e0cc22d45f3af2b8068e/test_3_summary.png)

#### Active Threads
![test_3_threads](uploads/936fcdb6d352f82f6ec2d17c97185ac5/test_3_threads.png)

#### Response Time

![test_3_response](uploads/fce7be1d5934b69817b9d8299697d346/test_3_response.png)
#### Transactions
![test_3_transactions](uploads/d56e0d5ee6dc684ac6df16e924715391/test_3_transactions.png)

#### Logs

https://pastebin.com/T01CjWGn

### Teste 4

* RPS: 100
* Max Response Time (ms): 1000
* Threads: 100

#### Summary
![test_4_summary](uploads/3ba5bf466a809f0feab4363215510056/test_4_summary.png)

#### Active Threads
![test_4_threads](uploads/1e3211d94de43f6fb8ca769561be49bb/test_4_threads.png)
#### Response Time

![test_4_response](uploads/fe3ca09e3882a5e5262f2c235291b2eb/test_4_response.png)
#### Transactions
![test_4_transactions](uploads/5ac62c0f890689189e70a46c2eb461c5/test_4_transactions.png)

#### Logs

https://pastebin.com/fn2grx2K

### Teste 5

* RPS: 100
* Max Response Time (ms): 10000
* Threads: 1000


#### Summary

![test_5_summary](uploads/76f8654d3bd5435163893a3ca078c880/test_5_summary.png)
#### Active Threads

![test_5_threads](uploads/6d91ffd67867d861c93844ab67cfb05c/test_5_threads.png)

#### Response Time

![test_5_response](uploads/b5676eb446561191e6ed86febadb3b1a/test_5_response.png)
#### Transactions

![test_5_transactions](uploads/7c9b6e19efa690d38a677ec650c8ab9a/test_5_transactions.png)

#### Logs

https://pastebin.com/awGDhZrJ

### Teste 6

* RPS: 1000
* Max Response Time (ms): 100
* Threads: 100

#### Summary

![test_6_summary](uploads/d888293d5fefaa86e6348b31f57f4ea6/test_6_summary.png)
#### Active Threads
![test_6_threads](uploads/e6bb3629bb5ede1a4102ce8683dd509f/test_6_threads.png)

#### Response Time

![test_6_response](uploads/19630564718d6ccd79358f71dee39aeb/test_6_response.png)
#### Transactions
![test_6_transactions](uploads/fbeddae0827eef8b1724b5adfa4321e9/test_6_transactions.png)

#### Logs

https://pastebin.com/5ftkHhVf

### Teste 7

* RPS: 1000
* Max Response Time (ms): 1000
* Threads: 1000

#### Summary

![test_7_summary](uploads/604a7f296a711031da253612fb31a8eb/test_7_summary.png)
#### Active Threads

![test_7_threads](uploads/8e51ab91a8fc811842fa8033b3167e3b/test_7_threads.png)

#### Response Time

![test_7_response](uploads/355e2d6e38b02bc3685f5c6350b4588f/test_7_response.png)
#### Transactions
![test_7_transactions](uploads/8342429aa6ce443a00576dda4db55813/test_7_transactions.png)

#### Logs

https://pastebin.com/tP8pfna1

### Teste 8

* RPS: 1000
* Max Response Time (ms): 10000
* Threads: 10000

#### Summary

![test_8_summary](uploads/cfc8c6c86d1d18b285b85d49a97d2ba3/test_8_summary.png)
#### Active Threads

![test_8_threads](uploads/7550aefb45ae31098def98e025c83e59/test_8_threads.png)
#### Response Time
![test_8_response](uploads/8265968dd1e220182c872b8c1dabb0d5/test_8_response.png)

#### Transactions
![test_8_transactions](uploads/e9b947c8484a1da1f3c0a3f2eb55c86f/test_8_transactions.png)


#### Logs

https://pastebin.com/zKzdMdmy