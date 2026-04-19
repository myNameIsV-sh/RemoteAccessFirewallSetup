# Índice

1. [Preparando o ambiente](#preparando-o-ambiente)  
   1.1. [Primeiros Passos](#primeiros-passos)  
   1.2. [Criando um contêiner básico](#criando-um-contêiner-básico)  
   1.3. [Permissões de Rede](#permissões-de-rede)

   
## 1. Preparando o ambiente
Não serei tão detalhista nessa parte, portanto é esperado que você já tenha o `docker` instalado, caso ainda não tenha, recomendo a leitura da documentação oficial do Docker disponível em: https://docs.docker.com/get-started/get-docker/ 

### 1.1 Primeiros passos 
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

### 1.2 Criando um contêiner básico 
Existem diversas formas para executar um contêiner, abordarei a maneira mais granular disponível. Primeiramente, criaremos o contêiner com o comando `docker create`, é importante destacar que, uma vez criado, não é possível ceder novas permissões para o contêiner, essa informação será importante lá na frente quando formos lidar com firewall. 

A maneira mais básica para criar um contêiner é essa

```bash
docker create -it --name meu-container ubuntu:24.04 /bin/bash
```

O parâmetro `it` significa *interactive teletypewriter*, entenda como terminal interativo, `--name` como sugere define o nome do contêiner, é importante reservar um nome significativo pois o terminal consegue preencher automaticamente o nome do contêiner pressionando `tab`. Especificamos a imagem que queremos executar e também específicamos o interpretador de comandos para o contêiner, nesse ccaso estamos utilizando o `bash`.

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

## 1.4 Instalando pacotes
Dentro contêiner, podemos instalar os pacotes desejados, precisamos instalar o `ufw` e o `ssh`, para isso iremos atualizar a nossa lista de pacotes e em seguida iremos instalar os pacotes desejados. Como estamos utilizando o Ubuntu, iremos utilizar o APT (Advanced Package Tool).

```text
apt update -y && apt install ufw ssh -y
```

> Nota: Utilize o parâmetro `-y` com atenção, pois ele irá ignorar todas as confirmações e vai prosseguir com quais alterações.

O `ssh` é um meta-pacote, ou seja, ele engloba os pacotes `openssh-client` e `openssh-server`. Escolha de acordo com a sua necessiade. Para verificarmos se os pacotes foram instalados, basta executar `ufw version` e `ssh -V` .

```text
root@cac44711b105:/# ufw version
ufw 0.36.2
Copyright 2008-2023 Canonical Ltd.

root@cac44711b105:/# ssh -V
OpenSSH_9.6p1 Ubuntu-3ubuntu13.15, OpenSSL 3.0.13 30 Jan 2024
```

## 1.5 Iniciando os serviços
Para prosseguirmos com as ativações do serviços, é necessário ter algum gerenciador de processos instalando no nosso contêiner, no entanto, essa prática é completamente desencorajada para ambientes de produção e vai contra o princípio de responsabilidade única dos processos que o Docker propõe; um contêiner executando sistema operacional como o Ubuntu é um processo isolado, quando instalamos o SystemD, é preciso que ele seja o primeiro processo número 1 em execução no sistema, é nesse momento que surge um impasse. Já no caso do UFW lidamos com outro problema, o Docker é reponsável por se comunicar com o firewall do sistema hóspede, utlizar o UFW dentro de um contêiner requer permissões especiais para manipulação de tabelas de endereços IP e portas do sistema, dessa forma o contêiner precisaria ter um acesso especial à esses recursos, tornando a sua execução mais insegura. Como estamos em um ambiente didático e controlado, iremos simular a execução de ambos os programas.

Se tentarmos executar o comando `systemctl status`, seremos recebidos com a seguinte mensagem de erro:
```bash
root@cac44711b105:/# systemctl status
System has not been booted with systemd as init system (PID 1). Can't operate.
Failed to connect to bus: Host is down
```

Primeiro, vamos instalar o SystemD

```text
apt install systemd systemctl 
```

Dessa forma podemos executar os comandos necessários para a execução dos serviços que instalamos, vamos verificar os status e vamos ativá-los, as respostas do comando `status` certamente estarão bagunçadas, mas é possível enxergar as informações que precisamos, atente-se para a última linha, é ela que nos diz o status do nosso serviço.

```text
root@cac44711b105:/# systemctl status ufw
/usr/bin/systemctl:1541: SyntaxWarning: invalid escape sequence '\w'
  expanded = re.sub("[$](\w+)", lambda m: get_env1(m), cmd.replace("\\\n",""))
/usr/bin/systemctl:1543: SyntaxWarning: invalid escape sequence '\w'
  new_text = re.sub("[$][{](\w+)[}]", lambda m: get_env2(m), expanded)
/usr/bin/systemctl:1628: SyntaxWarning: invalid escape sequence '\w'
  cmd3 = re.sub("[$](\w+)", lambda m: get_env1(m), cmd2)
/usr/bin/systemctl:1631: SyntaxWarning: invalid escape sequence '\w'
  newcmd += [ re.sub("[$][{](\w+)[}]", lambda m: get_env2(m), part) ]
ufw.service - Uncomplicated firewall
    Loaded: loaded (/usr/lib/systemd/system/ufw.service, enabled)
    Active: inactive (dead)
```

O mesmo vale para o ssh:

```text
root@cac44711b105:/# systemctl status ufw
/usr/bin/systemctl:1541: SyntaxWarning: invalid escape sequence '\w'
  expanded = re.sub("[$](\w+)", lambda m: get_env1(m), cmd.replace("\\\n",""))
/usr/bin/systemctl:1543: SyntaxWarning: invalid escape sequence '\w'
  new_text = re.sub("[$][{](\w+)[}]", lambda m: get_env2(m), expanded)
/usr/bin/systemctl:1628: SyntaxWarning: invalid escape sequence '\w'
  cmd3 = re.sub("[$](\w+)", lambda m: get_env1(m), cmd2)
/usr/bin/systemctl:1631: SyntaxWarning: invalid escape sequence '\w'
  newcmd += [ re.sub("[$][{](\w+)[}]", lambda m: get_env2(m), part) ]
ufw.service - Uncomplicated firewall
    Loaded: loaded (/usr/lib/systemd/system/ufw.service, enabled)
    Active: inactive (dead)
```

Precisamos ativar os serviços, para isso, basta executar `systemctl start <nome do serviço>` e `systemctl enable <nome do serviço>` para que o processo inicie junto com o sistema. Note que, quando iniciarmos os serviços, não haverá nenhuma mensagem de confirmação, mas se executarmos `systemctl status <nome do serviço>` novamente, poderemos ver que o serviço foi ativo. No exemplo abaixo, utilizei o ufw, basta repetir o mesmo procedimento para o ssh ou para qualquer outro serviço que desejar.

```text
root@cac44711b105:/# systemctl start ufw
/usr/bin/systemctl:1541: SyntaxWarning: invalid escape sequence '\w'
  expanded = re.sub("[$](\w+)", lambda m: get_env1(m), cmd.replace("\\\n",""))
/usr/bin/systemctl:1543: SyntaxWarning: invalid escape sequence '\w'
  new_text = re.sub("[$][{](\w+)[}]", lambda m: get_env2(m), expanded)
/usr/bin/systemctl:1628: SyntaxWarning: invalid escape sequence '\w'
  cmd3 = re.sub("[$](\w+)", lambda m: get_env1(m), cmd2)
/usr/bin/systemctl:1631: SyntaxWarning: invalid escape sequence '\w'
  newcmd += [ re.sub("[$][{](\w+)[}]", lambda m: get_env2(m), part) ]

root@cac44711b105:/# systemctl enable ufw
/usr/bin/systemctl:1541: SyntaxWarning: invalid escape sequence '\w'
  expanded = re.sub("[$](\w+)", lambda m: get_env1(m), cmd.replace("\\\n",""))
/usr/bin/systemctl:1543: SyntaxWarning: invalid escape sequence '\w'
  new_text = re.sub("[$][{](\w+)[}]", lambda m: get_env2(m), expanded)
/usr/bin/systemctl:1628: SyntaxWarning: invalid escape sequence '\w'

  cmd3 = re.sub("[$](\w+)", lambda m: get_env1(m), cmd2)
/usr/bin/systemctl:1631: SyntaxWarning: invalid escape sequence '\w'
  newcmd += [ re.sub("[$][{](\w+)[}]", lambda m: get_env2(m), part) ]

root@cac44711b105:/# systemctl status ufw
/usr/bin/systemctl:1541: SyntaxWarning: invalid escape sequence '\w'
  expanded = re.sub("[$](\w+)", lambda m: get_env1(m), cmd.replace("\\\n",""))
/usr/bin/systemctl:1543: SyntaxWarning: invalid escape sequence '\w'
  new_text = re.sub("[$][{](\w+)[}]", lambda m: get_env2(m), expanded)
/usr/bin/systemctl:1628: SyntaxWarning: invalid escape sequence '\w'
  cmd3 = re.sub("[$](\w+)", lambda m: get_env1(m), cmd2)
/usr/bin/systemctl:1631: SyntaxWarning: invalid escape sequence '\w'
  newcmd += [ re.sub("[$][{](\w+)[}]", lambda m: get_env2(m), part) ]
ufw.service - Uncomplicated firewall
    Loaded: loaded (/usr/lib/systemd/system/ufw.service, enabled)
    Active: active (running)
```

Existe uma maneira mais elegante de executarmos esses serviços, utilizando o parâmetro `--now` no comando `systemctl enable`, dispensamos a necessidade de utilizarmos `start` e `enable`.

Na próxima sessão, iremos realizar algumas configurações pontuais para o SSH e UFW.

## 1.6 Configurando serviços
Com os processos devidamente iniciliazados e habilitados, agora podermos configurar e executar os nossos serviços, primeiro, vamos configurar o SSH.

Como estamos em um ambiente virtual, por padrão o usuário sempre será o `root`, por conta disso, teremos alguns problemas para acessarmos um servidor ssh que esteja contêinerizado. Primeiro vamos definir uma senhas para o `root` utilizando o comando `passwd`. A sintaxe é simple:

```text
root@cac44711b105:/# passwd root 
New password: 
Retype new password: 
passwd: password updated successfully
```

Como estamos em um ambiente didático e controlado, inseri a senha "123", mas lembre-se sempre de utilizar uma senha forte.

Após ter dedicado uma senha para o root, agora precisamos permitir o Login como usuário root no servidor, para isso precisamos acessar o arquivo de configuração do SSH e remover a `#` da linha `#PermitRootLogin prohibit-password` e no lugar de `prohibit-password` substituimos por `yes`. O arquivo se encontra em `/etc/ssh/sshd_config`, caso queira utilizar um editor de textos como o `nano`, `vi` lembre-se de instalá-los, ou caso queira utilizar uma opção já existente, utilize o comando `sed`.

```text
root@cac44711b105:/# sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
```

Após isso, reinicie o ssh com o comando `systemctl restart ssh` para que as mudanças surtam efeito.










### 1.4 Permissões de Rede
