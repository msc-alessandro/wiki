## Dependências de instalação

O Chake e o chef utilizam Ruby como linguagem de script e portanto dependem que o Ruby esteja instalado na máquina de onde os comandos serão executados.

```
Ruby version >= 0
```


## Dependências de execução

O Chake em si é uma gem (Pacote do Ruby) que utiliza o Rake para a execução dos comandos ( *tasks* ). Portanto ambas as gems devem ser instaladas.

```
Linux

gem install chake

gem install rake
```
Para a execução local, foram preparadas VMs e portanto será necessário baixar e instalar o [Vagrant](https://www.vagrantup.com/downloads.html).

## Dependencias de instalação nos nós alvo.

As máquinas alvo da instalção devem possuir `sudo` instalado além de ferramentas como `wget` e `curl`. Para permitir a instalação no windows é necessário que a máquina possua o gerenciador de pacotes `Chocolatey`. A lista de dependencias dos nós alvo é a seguinte:

```
Windows:
	- Chocolatey
	- sudo
	- wget
	- curl

Linux:
	- sudo
	- wget
	- curl
```

## Árvore de arquivos

```
├── bootstrap_files
│   └── chefdk_3.6.57-1_amd64.deb
├── config
│   ├── local
│   └── roles
├── config.rb
├── cookbooks
│   ├── basics
│   ├── docker
│   ├── firewalld
│   └── gitlab-ci-runner
├── nodes.d
├── nodes.yaml
├── Rakefile
├── README.md
├── scripts
└── Vagrantfile
```

|    **Arquivo**    |                                            **Descrição**                                                |
| ----------------- | ------------------------------------------------------------------------------------------------------- |
| nodes.yaml        | Lista dos nós que estão sendo gerenciados e uma lista das receitas que devem ser executadas em cada nó. |
| config.rb         | Configurações que serão utilizadas pelo chef-solo.                                                      |
| config/ambiente   | Configurações de ip e ssh que dever ser utilizadas pelo chake para realizar ssh nos nós.                |
| config/roles      | Roles que serão utilizados pelo chef-solo.                                                              |
| cookbooks         | Diretório com as receitas que serão executadas nos nós.                                                 |
| Vagrantfile       | Arquivo de confiuração do Vagrant. Pode ser usado para rodar e VMs/Containers localmente.               |
| scripts           | Pasta com scripts para utilização no processo de deploy.                                                |

## Adicionando novos ambientes para deploy

Para adicionar um novo ambiente, crie uma nova pasta dentro do diretório config com o nome do ambiente. Neste diretório, devem existir três arquivos, são eles:

```
ips.yaml: Contém os ips dos nós que vão receber as configurações
ssh_config: Contém as configurações para ssh dos nós.
runners.yaml: Contém as configurações dos runners do gitlab que serão aplicadas nos nós.
```

## Adicionando nós a um ambiente

Adicione um novo nó no arquivo nodes.yaml.

```
gitlab-runner:
  run_list:
    - role[gitlab-runner]
```

Adicione o ip do novo nó no arquivo ips.yaml dentro da pasta do ambiente que este novo nó se encontra.

```
gitlab-runner: 10.0.2.10
```

Adicione as configurações de SSH no arquivo `ssh_config` dentro da pasta do ambiente em que o nó se encontra.

```
Host gitlab-runner
  HostName 127.0.0.1
  User vagrant
  Port 2222
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile /home/alessandrocb/Projetos/chef/.vagrant/machines/gitlab-runner/virtualbox/private_key
  IdentitiesOnly yes
  LogLevel FATAL
```

## Convergindo os ambientes

O comando para aplicar as receitas nos nós é o `converge`. Para convergir as máquinas o comando é `rake converge:<nó> CHAKE_ENV=<ambiente>`, ao rodar este comando todas as receitas ou roles aplicados ao nó que estão presentes no `nodes.yaml` são aplicados a uma máquina. O usuário pode selecionar qual máquina quer convergir passando a opção de nó no comando, como por exemplo `rake converge:gitlab-runner` ou `rake converge:desktop`, e ao rodar o comando sem a opção de nó, todas as máquinas serão convergidas, ignorando as configurações locais que possuem o prefixo `local://`. A variável `CHAKE_ENV` dita em qual ambiente o converge irá ocorrer, caso essa varável seja ignorada o `chake`  utilizará as configurações presentes em `config/local/`, caso um ambiente seja especificado, como `CHAKE_ENV=ed`, o converge irá ocorrer com as especificações dos arquivos em `config/ed/`.

