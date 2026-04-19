# 1. Preparando o ambiente
Não serei tão detalhista nessa parte, portanto é esperado que você já tenha o `docker` instalado, caso ainda não tenha, recomendo a leitura da documentação oficial do Docker disponível em: https://docs.docker.com/get-started/get-docker/ 

Para a atividade de hoje iremos o Ubuntu 24.04 "Noble Numbat" como base, para isso basta executar o comando abaixo:

```text
docker pull ubuntu:24.04
```

O comando acima irá baixar a imagem oficial do Ubuntu no [Docker Hub](https://hub.docker.com/_/ubuntu), por padrão o Docker sempre irá baixar a última versão disponível, para especificarmos a versão, basta adicionarmos uma tag seguindo a sintaxe `:<versão>`

O resultado será esse:
```text
v@hp-256r-g9:~/Projetos/Faculdade/RemoteAccessFirewallSetup$ docker pull ubuntu:24.04
24.04: Pulling from library/ubuntu
b40150c1c271: Pull complete 
92842f25412d: Download complete 
Digest: sha256:c4a8d5503dfb2a3eb8ab5f807da5bc69a85730fb49b5cfca2330194ebcc41c7b
Status: Downloaded newer image for ubuntu:24.04
docker.io/library/ubuntu:24.04
```

Caso queira verificar a imagem, basta executar o comando `docker images`
```text
v@hp-256r-g9:~/Projetos/Faculdade/RemoteAccessFirewallSetup$ docker images
                                                                                                                    i Info →   U  In Use
IMAGE           ID             DISK USAGE   CONTENT SIZE   EXTRA
ubuntu:24.04    c4a8d5503dfb        119MB         31.7MB
```

Com tudo pronto, podemos prosseguir para a execução do contêiner.

## 1.2 Executando o contêiner 
Existem diversas formas para executar um contêiner, abordarei a maneira mais granular disponível. Primeiramente, criaremos o contêiner com o comando `docker create`, é importante destacar que, uma vez criado, não é possível ceder novas permissões para o contêiner, essa informação será importante lá na frente quando formos lidar com firewall. 

A maneira mais básica para criar um contêiner é essa

```bash
docker create -it --name meu-container ubuntu:24.04 /bin/bash
```

O parâmetro `it` significa *interactive teletypewriter*, entenda como terminal interativo, `--name` como sugere define o nome do contêiner, é importante reservar um nome significativo pois o terminal consegue preencher automaticamente o nome do contêiner pressionando `tab`. Especificamos a imagem que queremos executar e também específicamos o interpretador de comandos para o contêiner, nesse ccaso estamos utilizando o `bash`.

> Atenção
> Nota: Este é um exemplo de criação básica. Como iremos lidar com rede, precisaremos destruir este contêiner e recriá-lo com permissões de rede específicas que veremos mais adiante.

Após a execução do comando, o seu terminal irá retornar o `CONTAINER ID` do contêiner, indicado que o contêiner foi criado com sucesso.

```text
v@hp-256r-g9:~/Projetos/Faculdade/RemoteAccessFirewallSetup$ docker create -it --name meu-container ubuntu:24.04 /bin/bash
cac44711b105ac2f4e7f57fb5c25ba7650a77967c2a887a200ff5a7e74e689ed
```

Para listarmos os nossos contêineres, basta executar o comando `docker ps -a`, o parâmetro `-a` significa *all*.
```text
v@hp-256r-g9:~/Projetos/Faculdade/RemoteAccessFirewallSetup$ docker ps -a
CONTAINER ID   IMAGE          COMMAND       CREATED         STATUS    PORTS     NAMES
cac44711b105   ubuntu:24.04   "/bin/bash"   8 seconds ago   Created             meu-container
```

Para iniciarmos o nosso contêiner basta utilizarmos o comando `start` fornecendo o nome ou identificador do contêiner como parâmetros:
```bash
docker start meu-container
```

E para acessá-lo basta executar:
```bash
docker exec -it meu-container /bin/bash
```

E *voilà*, temos um contêiner pronto para os nossos testes. Repare que agora estarmos dentro do contêiner e estamos livres executarmos quaisquer comandos. Irei executar o comando `cat /etc/os-release` para visualizarmos algumas informações referentes a distribuição que estamos utilizando.

```text
root@cac44711b105:/# cat /etc/os-release 
PRETTY_NAME="Ubuntu 24.04.4 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.4 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=noble
LOGO=ubuntu-logo
root@cac44711b105:/# 
```
