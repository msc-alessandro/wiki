## Raspberry PI 3 B e B+

*  **Compatibilidade com Pacote Pré-compilado:** Compatível - Testado até a versão 1.14.1
*  **Compatibilidade com Binários Compilados Manualmente:** Compatível - Testado até a versão 1.14.1

## Raspberry PI 2

* **Compatibilidade com Pacote Pré-compilado:** Compatível - Testado até a versão 1.14.1
* **Compatibilidade com Binários Compilados Manualmente:** Compatível - Testado até a versão 1.14.1

## Raspberry PI Zero

* **Compatibilidade com Pacote Pré-compilado:**  Incompatível - Kubernetes deixou de dar suporte aos Raspberries baseados arquitetura ARMV6l na versão 1.5.6. 


## Raspberry PI 1 A+ (v1.1) e B+ (v1.1)

* **Compatibilidade com Pacote Pré-compilado:**  Incompatível - Kubernetes deixou de dar suporte aos Raspberries baseados arquitetura ARMV6l na versão 1.5.6.
* **Compatibilidade com Binários Compilados Manualmente:** Compatível - Testado até a versão 1.10


Para compilar para esses sistemas.

```
sudo apt-get install gcc-arm-linux-gnueabi 
```

É necessário alterar o arquivo de compilação do Kubernetes ` hack/lib/golang.sh`  e adicionar flags para tornar possível o cross-compiling para ARMV6 e ARMHF

Mudar a seção:
```
export GOOS=${platform%/}
export GOARCH=${platform##/}
```

Para:
```
export GOOS=${platform%/}
export GOARCH=${platform##/}
export GOARM=5 
```

E depois alterar o *cross-compiler* para:

```
case "${platform}" in
"linux/arm")
export CGO_ENABLED=1
export CC=arm-linux-gnueabi-gcc
```

Para compilar
```
make all WHAT=cmd/kube-proxy KUBE_VERBOSE=5 KUBE_BUILD_PLATFORMS=linux/arm
make all WHAT=cmd/kubelet KUBE_VERBOSE=5 KUBE_BUILD_PLATFORMS=linux/arm
make all WHAT=cmd/kubectl KUBE_VERBOSE=5 KUBE_BUILD_PLATFORMS=linux/arm
```


## Raspberry PI 1 A+ (v1.0)

Não encontrei esse Raspberry para venda e nem alguem que possua essa primeira versão, mas não acredito que consiga rodar o sistema aqui pois a capacidade de hardware é muito baixa.





