# 1. Preparando o ambiente
Não serei tão detalhista nessa parte, portanto é esperado que você já tenha o `docker` instalado, caso ainda não tenha, recomendo a leitura da documentação oficial do Docker disponível em: https://docs.docker.com/get-started/get-docker/ 

Para a atividade de hoje iremos o Ubuntu 24.04 "Noble Numbat" como base, para isso basta executar o comando abaixo:

```bash
docker pull ubuntu:24.04
```

O comando acima irá baixar a imagem oficial do Ubuntu no [Docker Hub](https://hub.docker.com/_/ubuntu), por padrão o Docker sempre irá baixar a última versão disponível, para especificarmos a versão, basta adicionarmos uma tag seguindo a sintaxe `:<versão>`

O resultado será esse:
```bash
v@hp-256r-g9:~/Projetos/Faculdade/RemoteAccessFirewallSetup$ docker pull ubuntu:24.04
24.04: Pulling from library/ubuntu
b40150c1c271: Pull complete 
92842f25412d: Download complete 
Digest: sha256:c4a8d5503dfb2a3eb8ab5f807da5bc69a85730fb49b5cfca2330194ebcc41c7b
Status: Downloaded newer image for ubuntu:24.04
docker.io/library/ubuntu:24.04
```
Caso queira verificar a imagem, basta executar o comando `docker image`
```bash
v@hp-256r-g9:~/Projetos/Faculdade/RemoteAccessFirewallSetup$ docker images
                                                                                                                    i Info →   U  In Use
IMAGE           ID             DISK USAGE   CONTENT SIZE   EXTRA
ubuntu:24.04    c4a8d5503dfb        119MB         31.7MB
```

Com tudo pronto, podemos prosseguir para a execução do contêiner.

## 1.2 Executando o contêiner 
Existem diversas formas para executar um contêiner, abordarei a maneira mais granular disponível. Primeiramente, criaremos o contêiner com o comando `docker create`

```bash
docker create -it --name meu-container --cap-add=NET_ADMIN -p 2222:22 ubuntu:24.04 bash
```

O parâmetro `it` significa *interactive teletypewriter*, entenda como terminal interativo, `--name` como sugere define o nome do contêiner, é importante reservar um nome significativo pois o terminal consegue preencher automaticamente o nome do contêiner pressionando `tab`.


```bash
docker run --it --name ubuntu:24.04

apt install 
