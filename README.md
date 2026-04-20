# RemoteAccessFirewallSetup

Documentação prática sobre como configurar acesso remoto via **SSH** e controle de tráfego de rede com **UFW** em um ambiente Linux, do zero. O ambiente é baseado em contêineres Docker com Ubuntu 24.04, simulando uma máquina sem nenhuma dessas ferramentas pré-instaladas.

## Como funciona

O projeto é composto por dois contêineres:

- **Servidor** — executa o `openssh-server` e o `ufw`, simulando uma máquina remota acessível via SSH e protegida por firewall.
- **Cliente** — executa o `openssh-client`, simulando a máquina que inicia a conexão.

```
RemoteAccessFirewallSetup/
├── README.md
├── RemoteAccessFirewallSetup.md
├── server/
│   └── Dockerfile.server
└── client/
    └── Dockerfile.client
```

## Início rápido

**Build das imagens:**
```bash
docker build -t servidor-ssh -f server/Dockerfile.server .
docker build -t cliente-ssh -f client/Dockerfile.client .
```

**Subindo os contêineres:**
```bash
docker run -d --name servidor --cap-add NET_ADMIN servidor-ssh
docker run -it --name cliente cliente-ssh
```

**Conectando do cliente ao servidor:**
```bash
# Dentro do contêiner cliente:
ssh root@<IP_DO_SERVIDOR>
# Senha: 123
```

> O IP do servidor pode ser obtido com `docker inspect servidor | grep IPAddress`.

## Documentação completa

Para o passo a passo detalhado, incluindo explicações sobre cada comando, configuração do SSH, ativação do UFW e regras de firewall, consulte:

📄 [RemoteAccessFirewallSetup.md](./RemoteAccessFirewallSetup.md)
