
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


### CGroups e Systemd

É necessário editar o arquivo do systemd do Kubernetes para adicionar o *cgroup-driver* usado pelo Docker, para isto execute:


```
docker info | grep -i cgroup
# Cgroup Driver: cgroupfs
```

Depois abra o arquivo do SystemD:

```
sudo vim /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```

E troque a linha:

```
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS
```

Por:

```
ExecStart=/usr/bin/kubelet --cgroup-driver=cgroupfs $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS
```

Depois recarregue o daemon:

```
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

### Configuração do Master

Instale o Kubernetes no master seguindo as mesmas configurações anteriores e execute:

```
sudo kubeadm init
```

### Atenção

Caso o Kubernetes reclame de algo durante a instalação você pode desabilitar o *error checking* através do seguinte parâmetro:

```
sudo kubeadm init --ignore-preflight-errors='SystemVerification'
```

#### Trava na inicialização

Caso o kubernetes trave com o seguinte erro durante a inicialização:

```
[kubelet-check] It seems like the kubelet isn't running or healthy.
[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get http://localhost:10248/healthz: dial tcp 127.0.0.1:10248: connect: connection refused.
```

Será necessário mudar os seguintes arquivos:

```
--- /usr/lib/systemd/system/kubelet.service ---

[Unit] 

Description=Kubernetes Kubelet Server 
Documentation=https://github.com/GoogleCloudPlatform/kubernetes

[Service] 

WorkingDirectory=/var/lib/kubelet
EnvironmentFile=-/etc/kubernetes/config
EnvironmentFile=-/etc/kubernetes/kubelet
ExecStart=/usr/bin/kubelet \

    $KUBE_LOGTOSTDERR \

    $KUBE_LOG_LEVEL \

    $KUBELET_API_SERVER \

    $KUBELET_ADDRESS \

    $KUBELET_PORT \

    $KUBELET_HOSTNAME \

    $KUBE_ALLOW_PRIV \

    $KUBELET_ARGS

Restart=always
StartLimitInterval=0
RestartSec=10

[Install]
WantedBy=multi-user.target

```

```
--- /etc/kubernetes/config ---

KUBE_LOGTOSTDERR="--logtostderr=true"
KUBE_LOG_LEVEL="--v=0"
```

```
--- /etc/kubernetes/kubelet ---

KUBELET_ARGS="--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --cgroup-driver=cgroupfs --network-plugin=cni --pod-infra-container-image=k8s.gcr.io/pause:3.1"
```

Depois é preciso dar reload no daemon do Linux e desabilitar os serviços do Kubernetes

```
systemctl daemon-reload
systemctl disable kube-apiserver
systemctl disable kube-controller-manager
systemctl disable kube-scheduler
systemctl disable kube-proxy

```

### Mais atenção ainda

Se por algum motivo (só Deus sabe qual, mas aconteceu uma vez) o Kubernetes não conseguir iniciar nem com as configurações acima, recomendo que remova a instalação do Kubernetes que fez para fazer na mão seguindo os passos a seguir:

Instale os pacotes

```
sudo apt curl docker-ce ebtables ethtool wget unzip golang-cfssl
```

É necessário permitir conexões em bridge com IPV4
 
```
sysctl net.bridge.bridge-nf-call-iptables=1
```

Se você ja tinha o Docker é necessário limpar o iptables:

```
iptables -F
iptables -t nat -F
```

Depois

```
systemctl enable docker && systemctl restart docker
```


Instale o CNI

```
export CNI_VERSION="v0.6.0"
mkdir -p /opt/cni/bin
curl -L "https://github.com/containernetworking/plugins/releases/download/${CNI_VERSION}/cni-plugins-arm64-${CNI_VERSION}.tgz" | tar -C /opt/cni/bin -xz
```


Instale o CRI

```
export CRICTL_VERSION="v1.11.1"
mkdir -p /opt/bin
curl -L "https://github.com/kubernetes-incubator/cri-tools/releases/download/${CRICTL_VERSION}/crictl-${CRICTL_VERSION}-linux-arm64.tar.gz" | tar -C /opt/bin -xz
```

Instale o Kubernetes
```
export RELEASE="$(curl -sSL https://dl.k8s.io/release/stable.txt)"

mkdir -p /opt/bin
cd /opt/bin
curl -L --remote-name-all https://storage.googleapis.com/kubernetes-release/release/${RELEASE}/bin/linux/amd64/{kubeadm,kubelet,kubectl}
chmod +x {kubeadm,kubelet,kubectl}

curl -sSL "https://raw.githubusercontent.com/kubernetes/kubernetes/${RELEASE}/build/debs/kubelet.service" | sed "s:/usr/bin:/opt/bin:g" > /etc/systemd/system/kubelet.service
mkdir -p /etc/systemd/system/kubelet.service.d
curl -sSL "https://raw.githubusercontent.com/kubernetes/kubernetes/${RELEASE}/build/debs/10-kubeadm.conf" | sed "s:/usr/bin:/opt/bin:g" > /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
systemctl enable kubelet && systemctl start kubelet
```

Adicione o `/opt/bin` no `$PATH`

```
export PATH="/opt/bin:$PATH"
```

## Finalização

Inicie o cluster

```
kubeadm init --pod-network-cidr=10.244.0.0/16
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Depois da instalação do cluster o terminal vai te instruir como conectar os workers e o master node, para isso é preciso executar o `kubeadm join`  com um comando parecido com o comando abaixo:

```
kubeadm join 192.168.25.5:6443 --token t2zy9t.nvyz2q57h5aob7xn \
    --discovery-token-ca-cert-hash sha256:4e0abbee9ddb3f8b13c45ac20c0d72d16d0792c8761e5c286e778793f11e5125 
```