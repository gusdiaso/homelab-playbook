# Configuração base do servidor

Este documento cobre a configuração inicial do servidor, as etapas fundamentais que garantem um ambiente seguro e acessível, independente dos serviços que serão instalados posteriormente.

Todo o setup foi pensado com segurança desde o início: nenhuma etapa foi feita de forma conveniente às custas de exposição desnecessária.

---

## Índice

- [Ambiente](#ambiente)
- [Etapa 1 — Atualização do sistema](#etapa-1--atualização-do-sistema)
- [Etapa 2 — Usuário seguro e bloqueio do root](#etapa-2--usuário-seguro-e-bloqueio-do-root)
- [Etapa 3 — IP fixo na rede local](#etapa-3--ip-fixo-na-rede-local)
- [Etapa 4 — SSH seguro](#etapa-4--ssh-seguro)
- [Etapa 5 — Firewall com UFW](#etapa-5--firewall-com-ufw)
- [Etapa 6 — Fail2Ban](#etapa-6--fail2ban)
- [Resumo de segurança](#resumo-de-segurança)

---

## Ambiente

| Componente | Valor |
|---|---|
| Sistema operacional | Ubuntu Server 24.04 LTS |
| Interface de rede | `enp3s0` |
| IP do servidor | `172.16.11.100` |
| IP do roteador | `172.16.11.1` |
| Usuário administrativo | `gustavo` |
| Hostname | `jarvis` |

---

## Etapa 1 — Atualização do sistema

### Por que fazer isso?

Antes de qualquer configuração, o sistema precisa estar atualizado. Pacotes desatualizados podem conter vulnerabilidades conhecidas e iniciar um servidor com software antigo significa começar com brechas de segurança que já têm exploit público.

### O que foi feito

```bash
apt update && apt upgrade -y
```

---

## Etapa 2 — Usuário seguro e bloqueio do root

### Por que fazer isso?

O Ubuntu Server recém-instalado usa o `root` como principal forma de acesso. O `root` é o superusuário do Linux, com permissão para fazer qualquer coisa no sistema sem restrições.

Isso representa um risco sério: se alguém descobrir a senha do root, tem controle total do servidor. A solução é criar um usuário comum com poderes administrativos controlados e bloquear o acesso direto ao root.

### O que foi feito

**Criação do usuário:**

```bash
adduser gustavo
```

O comando `adduser` é interativo e solicita senha e dados opcionais. A senha deve ser forte: mínimo 12 caracteres, com letras maiúsculas, minúsculas, números e símbolos.

**Adição ao grupo sudo:**

```bash
usermod -aG sudo gustavo
```

O grupo `sudo` permite executar comandos administrativos com `sudo comando`, exigindo a própria senha a cada uso. Toda ação administrativa é consciente e autenticada.

Verificação:

```bash
groups gustavo
# gustavo : gustavo sudo
```

**Teste antes de travar o root:**

Antes de qualquer mudança irreversível, o novo usuário foi testado em um segundo terminal:

```bash
su - gustavo
sudo whoami
# root
```

Somente após confirmar o acesso, o root foi bloqueado.

**Bloqueio da senha do root:**

```bash
sudo passwd -l root
```

A flag `-l` (*lock*) impede login direto como root, tanto localmente quanto via SSH. O usuário root continua existindo internamente pois o sistema depende dele, mas não pode ser acessado por ninguém de fora.

Verificação:

```bash
sudo passwd -S root
# root L ... (L = Locked)
```

**sudo configurado para pedir senha sempre:**

```bash
sudo visudo
```

Adicionado abaixo de `Defaults env_reset`:

```
Defaults timestamp_timeout=0
```

Por padrão, o sudo lembra a autenticação por 15 minutos. Com `timestamp_timeout=0`, a senha é exigida em toda execução, protegendo sessões abertas e desacompanhadas.

---

## Etapa 3 — IP fixo na rede local

### Por que fazer isso?

Por padrão, o roteador distribui IPs automaticamente via DHCP e o servidor pode receber um endereço diferente a cada reinicialização. Se o IP muda, o SSH para de funcionar, os serviços deixam de responder e o redirecionamento de portas do roteador aponta para o lugar errado.

Com IP fixo, o servidor sempre estará no mesmo endereço dentro da rede local, independente de quantas vezes reiniciar.

> O IP do roteador (`172.16.11.1`) nunca muda, pois ele se atribui esse endereço fixo. O que varia sem configuração é o IP dos outros aparelhos da rede.

### O que foi feito

Identificação da interface de rede e gateway:

```bash
ip a
# enp3s0: inet 172.16.11.X/24

ip route | grep default
# default via 172.16.11.1 dev enp3s0
```

Edição do arquivo de configuração de rede:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Conteúdo aplicado:

```yaml
network:
  version: 2
  ethernets:
    enp3s0:
      dhcp4: false
      addresses:
        - 172.16.11.100/24
      routes:
        - to: default
          via: 172.16.11.1
      nameservers:
        addresses: [1.1.1.1, 8.8.8.8]
```

| Opção | Descrição |
|---|---|
| `dhcp4: false` | Desliga a atribuição automática de IP |
| `addresses` | Define o IP fixo do servidor |
| `routes` | Define o roteador como saída para a internet |
| `nameservers` | Servidores DNS: Cloudflare (`1.1.1.1`) e Google (`8.8.8.8`) |

Aplicação e verificação:

```bash
sudo netplan apply

ip a | grep 172
# 172.16.11.100/24

ping -c 4 1.1.1.1
# 4 packets transmitted, 4 received
```

---

## Etapa 4 — SSH seguro

### Por que fazer isso?

SSH (*Secure Shell*) é o protocolo que permite controlar o servidor remotamente pelo terminal, sem teclado e monitor conectados fisicamente. É a forma padrão de administrar servidores Linux.

A configuração padrão tem vulnerabilidades: roda na porta 22 (varrida constantemente por bots), permite tentativas ilimitadas de senha e aceita login do root. Todas essas configurações foram endurecidas.

### O que foi feito

**Instalação:**

```bash
sudo apt install openssh-server -y
sudo systemctl enable ssh
sudo systemctl start ssh
```

O `enable` garante que o SSH inicia automaticamente a cada boot.

**Configuração endurecida em `/etc/ssh/sshd_config`:**

```
Port 2222
PermitRootLogin no
PasswordAuthentication yes
MaxAuthTries 3
LoginGraceTime 20
X11Forwarding no
AllowUsers gustavo
```

| Parâmetro | Valor | Motivo |
|---|---|---|
| `Port` | `2222` | Sai da porta 22, varrida constantemente por bots |
| `PermitRootLogin` | `no` | Root não consegue logar mesmo que tente |
| `PasswordAuthentication` | `yes` | Mantido por ora, será substituído por chave SSH futuramente |
| `MaxAuthTries` | `3` | Após 3 senhas erradas a conexão é encerrada |
| `LoginGraceTime` | `20` | 20 segundos para autenticar, depois desconecta |
| `X11Forwarding` | `no` | Desativa interface gráfica remota, desnecessária em servidor headless |
| `AllowUsers` | `gustavo` | Apenas este usuário pode conectar via SSH |

**Reinicialização e verificação:**

```bash
sudo systemctl restart ssh

sudo ss -tlnp | grep ssh
# LISTEN 0 128 0.0.0.0:2222
```

**Conexão do Windows:**

```bash
ssh -p 2222 gustavo@172.16.11.100
```

Na primeira conexão o sistema apresenta a fingerprint do servidor e pede confirmação. Isso é esperado e garante que você está conectando no servidor correto.

---

## Etapa 5 — Firewall com UFW

### Por que fazer isso?

Por padrão, o Ubuntu não bloqueia conexões entrantes e qualquer porta aberta por qualquer serviço fica acessível. O UFW (*Uncomplicated Firewall*) permite definir exatamente quais portas aceitam conexões, bloqueando todo o resto.

A política adotada é a mais restritiva possível: bloquear tudo por padrão e liberar apenas o estritamente necessário.

### O que foi feito

```bash
sudo apt install ufw -y

sudo ufw default deny incoming   # bloqueia tudo que entra por padrão
sudo ufw default allow outgoing  # permite tudo que sai

sudo ufw allow 2222/tcp          # SSH
sudo ufw allow 80/tcp            # HTTP
sudo ufw allow 443/tcp           # HTTPS

sudo ufw enable
```

Regra de limite aplicada manualmente na porta do SSH:

```bash
sudo ufw limit 2222/tcp
```

O `LIMIT` rejeita conexões de IPs que tentarem se conectar mais de 6 vezes em 30 segundos, como proteção complementar ao Fail2Ban contra ataques de força bruta.

Estado final:

```
Status: active

To                         Action      From
--                         ------      ----
2222/tcp                   ALLOW       172.16.11.0/24
22                         DENY        Anywhere
80/tcp                     ALLOW       Anywhere
443/tcp                    ALLOW       Anywhere
2222/tcp                   LIMIT       Anywhere
```

---

## Etapa 6 — Fail2Ban

### Por que fazer isso?

Servidores expostos à internet recebem tentativas de login constantes. Bots automatizados testam combinações de usuário e senha ininterruptamente. O Fail2Ban monitora os logs do SSH e bane automaticamente IPs que excedem o número de tentativas permitidas, reduzindo drasticamente a superfície de ataque.

### O que foi feito

```bash
sudo apt install fail2ban -y
```

Arquivo de configuração criado em `/etc/fail2ban/jail.local`:

```ini
[DEFAULT]
bantime  = 1h
findtime = 10m
maxretry = 3

[sshd]
enabled = true
port    = 2222
```

| Opção | Valor | Significa |
|---|---|---|
| `bantime` | `1h` | IP banido fica bloqueado por 1 hora |
| `findtime` | `10m` | Janela de tempo que conta as tentativas |
| `maxretry` | `3` | 3 erros em 10 minutos resultam em banimento |
| `port` | `2222` | Monitora a porta correta do SSH |

```bash
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

Verificação:

```bash
sudo fail2ban-client status sshd
# Currently failed: 0 / Total banned: 0
```

---

## Resumo de segurança

| Medida | Como foi implementada |
|---|---|
| Sem acesso root direto | `passwd -l root` |
| sudo sempre pede senha | `timestamp_timeout=0` no visudo |
| SSH fora da porta padrão | `Port 2222` no sshd_config |
| Root bloqueado no SSH | `PermitRootLogin no` |
| Tentativas de login limitadas | `MaxAuthTries 3` + UFW LIMIT + Fail2Ban |
| Firewall restritivo | UFW com `deny incoming` por padrão |
| Apenas usuário autorizado via SSH | `AllowUsers gustavo` |