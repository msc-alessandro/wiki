# Gitlab Runners


## Requisitos

O Gitlab-Runner é escrito em [Go](https://golang.org/) e é executado como binário, sem a necessidade de requisitos para execução além da sua instalação, sendo assim, ele pode ser executado no Linux, Windows e OSX. Para utilizar o *runner* com o Docker, é necessária a instalação do Docker com a versão superior a `v1.5.0`.

## Executores

O Gitlab-Runner implementa um conjunto de executores que podem ser utilizados para construir e executar *builds* em diferentes cenários de execução. Os *runners* disponíveis são:

### Shell

O executor mais simples, ele permite a execução de builds no mesmo ambiente em que o runner está instalado e por consequência, é compatível com qualquer ambiente em que um Gitlab-Runner pode ser instalado.

### Docker

Utiliza o [Docker Engine](https://www.docker.com/products/docker-engine) para cada build, utilizando containers separados e isolados, com configurações predefinidas pelo `.gitlab-ci.yml` e de acordo com o `config.toml`.

### Virtualbox

Utiliza o Virtualbox para criar ambiente limpos a cada build. Nesta configuração é necessário que o Virtualbox exponha um servidor de ssh para que seja possível conectar as máquinas virtuais e que as máquinas possuam um *shell* compatível com o `Bash`.

### SSH

Permite que o Gitlab-CI faça SSH em máquinas remotas e execute as builds. Para isso, é necessário a que as configurações de SSH sejam definidas no arquivo `config.toml`.

```
[[runners]]
  executor = "ssh"
  [runners.ssh]
    host = "example.com"
    port = "22"
    user = "root"
    password = "password"
    identity_file = "/path/to/identity/file"

```


## Configuração dos runners

As configurações dos runners ficam no arquivo `config/<ambiente>/runners.yaml`, esse arquivo tem a seguinte configuração:

```
runners:
  - description: test-runner-1
    options:
      url: 'https://gitlab.com'
      registration_token: 'tZzvXHYxz9LqDRRN4nMv'
      tag_list: [ 'test1', 'tag' ]
      executor: 'docker'
      docker_image: 'node:8'
      retries: 1
  - description: test-runner-2
    options:
      url: 'https://gitlab.com'
      registration_token: 'tZzvXHYxz9LqDRRN4nMv'
      tag_list: [ 'test2', 'tag' ]
      executor: 'docker'
      docker_image: 'node:8'
      retries: 1

```

## Executando e criando runners localmente
Este repositório possui arquivos que permitem a criação de *runners* localmente usando máquinas virtuais para testar a configutação das receitas. O arquivo `Vagrantfile` possui configurações para instanciar uma máquina virtual, para isto é necessário apenas executar o comando `vagrant up` dentro da pasta do projeto. Caso queira convergir as ferramentas na sua máquina, sem fazer questão de isolamento da ferramenta, basta criar uma configuração como a seguinte no arquivo `nodes.yaml`:

```
local://desktop:
  run_list:
    - role[workstation]
local://laptop:
  run_list:
    - role[workstation]
```

Logo após, será necessário apenas executar o comando para convergir os ambiente.

