
## Dependências de instalação


Primeiro é necessário que você instale o Raspibian através deste [link](https://www.raspberrypi.org/downloads/raspbian/).

Depois é necessário configurar o *hostname* do Raspberry através do comando:

```
sudo raspi-config

```

A partir do Kubernetes na versão 1.7 é necessário que o host esteja com o Swap desabilitado, para isto é necessário executar os seguintes comandos:

```
sudo dphys-swapfile swapoff
sudo dphys-swapfile uninstall
sudo update-rc.d dphys-swapfile remove
```

### Docker

Para instalar o Docker, não use o script de instalação do Docker, a versão do Docker atual **(18.09.0)** causa falha de segmentação no Raspibian e por isto é necessário instalar uma versão anterior.

```
sudo apt-get install docker-ce=<VERSION_STRING>
// A versão testada foi a 18.06.3-ce
```

Adicione o usuário **pi** no grupo Docker para poder executar os comandos sem sudo:

```
sudo groupadd docker
```

```
sudo usermod -aG docker $USER
```


### Habilitar o CGROUPS

Abra o arquivo:

```
sudo vim /boot/cmdline.txt
```

Adicione os seguintes comandos ao final do arquivo:

```
cgroup_enable=cpuset cgroup_enable=memory
```

É necessário reiniciar o Raspberry:

```
sudo reboot
```

### Instalação do Kubernetes

Instale os pacotes de dependencia:

```
sudo apt update && sudo apt install -y apt-transport-https curl
```

Adicione a chave GPG do Google:

```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```

Crie o arquivo:

```
sudo vim /etc/apt/sources.list.d/kubernetes.list
```

Adicione o repositório do Kubernetes:

```
#/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
```

Realize o update de todos os repositórios:

```
sudo apt update

```

Instale os pacotes do Kubernetes:

```
sudo apt install kubelet kubeadm kubectl

```