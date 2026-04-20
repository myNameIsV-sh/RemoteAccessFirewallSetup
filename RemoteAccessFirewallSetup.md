# Índice

1. [Preparando o ambiente](#preparando-o-ambiente)  
   1.1. [Primeiros Passos](#primeiros-passos)  
   1.2. [Criando um contêiner básico](#criando-um-contêiner-básico)  
   1.3. [Permissões de Rede](#permissões-de-rede)

   
## 1. Preparando o ambiente
Não serei tão detalhista nessa parte, portanto é esperado que você já tenha o `docker` instalado. Caso ainda não tenha, recomendo a leitura da documentação oficial do Docker disponível em: https://docs.docker.com/get-started/get-docker/ 

### 1.1 Primeiros passos 
Para a atividade de hoje, iremos utilizar o Ubuntu 24.04 "Noble Numbat" como base. Para isso, basta executar o comando abaixo:

```text
docker pull ubuntu:24.04
```

O comando acima irá baixar a imagem oficial do Ubuntu no [Docker Hub](https://hub.docker.com/_/ubuntu). Por padrão, o Docker sempre irá baixar a última versão disponível; para especificarmos a versão, basta adicionarmos uma tag seguindo a sintaxe `:<versão>`.

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

Caso queira verificar a imagem, basta executar o comando `docker images`:
```text
v@hp-256r-g9:~/Projetos/Faculdade/RemoteAccessFirewallSetup$ docker images
                                                                                                                    i Info →   U  In Use
IMAGE           ID             DISK USAGE   CONTENT SIZE   EXTRA
ubuntu:24.04    c4a8d5503dfb        119MB         31.7MB
```

Com tudo pronto, podemos prosseguir para a execução do contêiner.

### 1.2 Criando um contêiner básico 
Existem diversas formas para executar um contêiner; abordarei a maneira mais granular disponível. Primeiramente, criaremos o contêiner com o comando `docker create`. É importante destacar que, uma vez criado, não é possível conceder novas permissões para o contêiner — essa informação será importante mais à frente, quando formos lidar com firewall.

A maneira mais básica para criar um contêiner é essa:

```bash
docker create -it --name meu-container ubuntu:24.04 /bin/bash
```

O parâmetro `it` significa *interactive teletypewriter*, entenda como terminal interativo. O `--name`, como sugere, define o nome do contêiner — é importante reservar um nome significativo, pois o terminal consegue preenchê-lo automaticamente ao pressionar `tab`. Especificamos também a imagem que queremos executar e o interpretador de comandos do contêiner, nesse caso o `bash`.

> Nota: Este é um exemplo de criação básica. Como iremos lidar com rede, precisaremos destruir este contêiner e recriá-lo com permissões de rede específicas, que veremos mais adiante.

Após a execução do comando, o terminal irá retornar o `CONTAINER ID` do contêiner, indicando que ele foi criado com sucesso.

```text
v@hp-256r-g9:~/Projetos/Faculdade/RemoteAccessFirewallSetup$ docker create -it --name meu-container ubuntu:24.04 /bin/bash
cac44711b105ac2f4e7f57fb5c25ba7650a77967c2a887a200ff5a7e74e689ed
```

Para listarmos os nossos contêineres, basta executar o comando `docker ps -a`, onde o parâmetro `-a` significa *all*:
```text
v@hp-256r-g9:~/Projetos/Faculdade/RemoteAccessFirewallSetup$ docker ps -a
CONTAINER ID   IMAGE          COMMAND       CREATED         STATUS    PORTS     NAMES
cac44711b105   ubuntu:24.04   "/bin/bash"   8 seconds ago   Created             meu-container
```

Para iniciar o contêiner, basta utilizar o comando `start` fornecendo o nome ou o identificador como parâmetro:
```bash
docker start meu-container
```

E para acessá-lo:
```bash
docker exec -it meu-container /bin/bash
```

E *voilà*, temos um contêiner pronto para os nossos testes. Repare que agora estamos dentro do contêiner e podemos executar quaisquer comandos. Vou executar o `cat /etc/os-release` para visualizarmos algumas informações sobre a distribuição em uso.

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
Dentro do contêiner, podemos instalar os pacotes desejados. Precisamos instalar o `ufw` e o `ssh`; para isso, vamos atualizar a lista de pacotes e em seguida instalar o que precisamos. Como estamos utilizando o Ubuntu, usaremos o APT (Advanced Package Tool).

```text
apt update -y && apt install ufw ssh -y
```

> Nota: Utilize o parâmetro `-y` com atenção, pois ele ignorará todas as confirmações e prosseguirá com as alterações automaticamente.

O `ssh` é um meta-pacote, ou seja, ele engloba os pacotes `openssh-client` e `openssh-server`. Escolha de acordo com a sua necessidade. Para verificarmos se os pacotes foram instalados corretamente, basta executar `ufw version` e `ssh -V`.

```text
root@cac44711b105:/# ufw version
ufw 0.36.2
Copyright 2008-2023 Canonical Ltd.

root@cac44711b105:/# ssh -V
OpenSSH_9.6p1 Ubuntu-3ubuntu13.15, OpenSSL 3.0.13 30 Jan 2024
```

## 1.5 Iniciando os serviços
Para ativarmos os serviços, é necessário ter algum gerenciador de processos instalado no contêiner. No entanto, essa prática é completamente desencorajada para ambientes de produção e vai contra o princípio de responsabilidade única que o Docker propõe: um contêiner executando um sistema operacional como o Ubuntu é um processo isolado, e quando instalamos o SystemD, ele precisa ser o primeiro processo em execução no sistema (PID 1) — é aí que surge um impasse. Já no caso do UFW, lidamos com outro problema: o Docker é responsável por se comunicar com o firewall do sistema hospedeiro, e utilizar o UFW dentro de um contêiner requer permissões especiais para manipular tabelas de endereços IP e portas, tornando a execução do contêiner mais insegura. Como estamos em um ambiente didático e controlado, iremos simular a execução de ambos os programas.

Se tentarmos executar o comando `systemctl status`, seremos recebidos com a seguinte mensagem de erro:
```bash
root@cac44711b105:/# systemctl status
System has not been booted with systemd as init system (PID 1). Can't operate.
Failed to connect to bus: Host is down
```

Primeiro, vamos instalar o SystemD:

```text
apt install systemd systemctl 
```

Com isso, podemos executar os comandos necessários para iniciar os serviços instalados. As respostas do comando `status` certamente virão com alguns avisos, mas é possível extrair as informações que precisamos — atente-se à última linha, pois é ela que indica o status do serviço.

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

Para ativar os serviços, basta executar `systemctl start <nome do serviço>` e `systemctl enable <nome do serviço>` para que o processo inicie junto com o sistema. Note que, ao iniciar os serviços, não haverá nenhuma mensagem de confirmação, mas ao executar `systemctl status <nome do serviço>` novamente, é possível verificar que o serviço está ativo. No exemplo abaixo utilizei o ufw; basta repetir o mesmo procedimento para o ssh ou qualquer outro serviço.

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

Existe uma maneira mais elegante de fazer isso: utilizando o parâmetro `--now` no comando `systemctl enable`, dispensamos a necessidade de chamar `start` e `enable` separadamente.

Na próxima seção, realizaremos algumas configurações pontuais para o SSH e o UFW.

## 1.6 Configurando serviços
Com os processos devidamente inicializados e habilitados, podemos agora configurar e executar os nossos serviços. Nesta seção, iremos configurar um servidor e um cliente SSH.

Como estamos em um ambiente virtual, por padrão o usuário sempre será o `root`, o que nos traz alguns problemas para acessar um servidor SSH contêinerizado. Primeiro, vamos definir uma senha para o `root` utilizando o comando `passwd`:

```text
root@cac44711b105:/# passwd root 
New password: 
Retype new password: 
passwd: password updated successfully
```

> Nota: como estamos em um ambiente didático e controlado, utilizei a senha "123", mas lembre-se de usar uma senha forte — de preferência uma frase longa.

Após definir a senha do root, precisamos permitir o login como usuário root no servidor. Para isso, é necessário acessar o arquivo de configuração do SSH, remover o `#` da linha `#PermitRootLogin prohibit-password` e substituir `prohibit-password` por `yes`. O arquivo se encontra em `/etc/ssh/sshd_config`. Caso queira usar um editor de texto como o `nano` ou `vi`, lembre-se de instalá-lo; ou, se preferir uma alternativa já disponível no sistema, utilize o `sed`:

```text
root@cac44711b105:/# sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
```

Após isso, reinicie o SSH com o comando `systemctl restart ssh` para que as mudanças entrem em vigor.

Do lado do cliente, utilizaremos o `openssh-client`. Para realizar a conexão, precisamos indicar o usuário que desejamos assumir e o servidor ao qual iremos nos conectar. Como estamos em máquinas locais, o comando `hostname -I` já nos fornece o endereço necessário:

```text
root@cac44711b105:/# hostname -I
172.17.0.2 
```

No contêiner do cliente, utilizamos o seguinte comando:

```text
root@b66df2f475b4:/# ssh root@172.17.0.2
```

No mesmo instante, você se deparará com uma mensagem parecida com essa:

```text
The authenticity of host '172.17.0.2 (172.17.0.2)' can't be established.
ED25519 key fingerprint is SHA256:abcd1234...
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

Ao confirmar com "yes", uma chave do servidor será salva no arquivo `~/.ssh/known_hosts` e, em seguida, será solicitada a senha do root do servidor:

```text
root@172.17.0.2's password:
```

Inserimos "123" e *voilà* — perceba que deixamos de estar no contêiner `b66df2f475b4` e passamos para o `cac44711b105`. Dessa forma, temos controle total sobre o servidor.

Para finalizar, vamos ativar o firewall no lado do servidor. Antes disso, precisamos alternar para o `iptables-legacy`, o que nos permitirá utilizar o firewall dentro do contêiner. Para isso, basta executar o comando `update-alternatives`:

```text
update-alternatives --set iptables /usr/sbin/iptables-legacy && update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
```

Após a execução, será exibida uma confirmação:

```text
update-alternatives: using /usr/sbin/iptables-legacy to provide /usr/sbin/iptables (iptables) in manual mode
update-alternatives: using /usr/sbin/ip6tables-legacy to provide /usr/sbin/ip6tables (ip6tables) in manual mode
```

O UFW, por padrão, bloqueia todas as conexões de entrada e libera todas as saídas, ou seja, a porta 22 (porta padrão do SSH) será bloqueada. No contêiner do servidor, basta executar:

```text
root@cac44711b105:/# ufw enable
```

A partir desse momento, todas as conexões de entrada são bloqueadas. O cliente não consegue mais se comunicar e, ao tentar realizar uma nova conexão, será recebido com um `Connection timed out`. Se quiséssemos ser ainda mais específicos, poderíamos bloquear diretamente o endereço IP do cliente — bastaria digitar `ufw deny from 172.17.0.3`.

# Considerações Finais
