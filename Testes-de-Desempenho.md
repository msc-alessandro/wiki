# Testes de Desempenho

## Medidas de Desempenho

![Medidas_de_Desempenho](uploads/6d82010c4d920cf1c5221c0e6ea73f64/Medidas_de_Desempenho.png)

## Subtipos de Testes de Desempenho

![Subtipos](uploads/f976ced675a806b7b56e8083ddf207a3/Subtipos.png)


## Escolhas Para os Testes do Cluster

### Subtipos

1.  Teste de Carga
2.  Teste de Estresse
3.  Teste de Capacidade


### Variáveis Independentes

1.  Carga máxima.
2.  Usuários.
3.  Rampa.
4.  Tempo de Resposta Tolerável.
5.  Número de Agentes. 


### Variáveis Dependentes

1. Usuários Ativos.
2. Tempo Médio de Resposta.
3. Porcentagem de Erros.
4. Throughput.


## Operacionalização

### JMeter CLI + GUI

O JMeter oferece a possibilidade de utilizar a ferramenta como CLI ou GUI para a realização dos testes de desempenho, podendo depois exportar os dados em diversos formatos (XML, CSV).



### JMeter Operator + InfluxDB + Grafana

O grupo [Kubernauts](https://github.com/kubernauts) disponiblizou abertamente um operador para realizar testes de Carga no Kubernetes utilizando o JMeter. Essa configuração utiliza o InfluxDB para criar uma série temporal dos dados que são gerados por Pods do JMeter nos nós do Kubernetes, a série temporal dos dados então é lida por um Pod do Grafana para apresentar os dados em uma interface mais amigável.


![test-architecture](uploads/8265d421282aa74682842ad0889afcdf/test-architecture.jpg)

![grafana1](uploads/39c98572515556240067a8a8d0e6843b/grafana1.png)

![grafana2](uploads/b7dee97082c03e550520e2004fd60926/grafana2.png)










