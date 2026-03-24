# Serviços

Este documento cobre a instalação e configuração dos serviços que rodam no servidor. Cada serviço é instalado de forma isolada e segura, com a configuração mínima necessária para funcionar em produção.

---

## Índice

- [Visão geral](#visão-geral)
- [Nginx](#nginx)
- [Próximos serviços](#próximos-serviços)

---

## Visão geral

O servidor foi projetado para hospedar múltiplos serviços de forma organizada. O Nginx atua como ponto central de entrada: todo tráfego HTTP/HTTPS passa por ele antes de chegar a qualquer serviço interno.

```
internet → Nginx (80/443)
               ├── sites estáticos
               ├── backend Node.js  (proxy → :3000)
               ├── backend Python   (proxy → :8000)
               └── outros serviços  (proxy → :XXXX)
```

Serviços de banco de dados e outros processos internos nunca ficam expostos diretamente. São acessíveis apenas através do Nginx ou dentro da própria rede do servidor.

---

## Nginx

### O que é

O Nginx é um servidor web e proxy reverso de alto desempenho. É o primeiro software a receber qualquer requisição vinda da internet e o responsável por decidir para onde ela vai.

Ele atua de duas formas no servidor:

- **Servidor web:** serve arquivos estáticos (HTML, CSS, JS, imagens) diretamente, sem passar por nenhum backend
- **Proxy reverso:** repassa requisições para backends internos (Node.js, Python, etc.) rodando em portas locais, sem expô-los diretamente à internet

### Por que Nginx e não Apache?

Ambos cumprem a mesma função. O Nginx foi escolhido por ser mais leve em consumo de memória, ter configuração mais legível e ser o padrão mais adotado em infraestruturas modernas.

### Instalação

```bash
sudo apt install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
```

O `enable` garante que o Nginx inicia automaticamente a cada boot do servidor.

### Verificação

```bash
sudo systemctl status nginx
# Active: active (running)
```

Acessando `http://172.16.11.100` no navegador, a página padrão do Nginx foi exibida com sucesso, confirmando que o serviço está respondendo corretamente na rede local.

### Estrutura de arquivos

| Caminho | Descrição |
|---|---|
| `/etc/nginx/nginx.conf` | Configuração principal |
| `/etc/nginx/sites-available/` | Configurações de cada site (inativas) |
| `/etc/nginx/sites-enabled/` | Links simbólicos para os sites ativos |
| `/var/www/` | Diretório raiz padrão dos arquivos dos sites |
| `/var/log/nginx/` | Logs de acesso e erro |

A configuração de cada site é feita em um arquivo separado dentro de `sites-available/` e ativada com um link simbólico em `sites-enabled/`. Isso permite habilitar e desabilitar sites sem apagar configurações.

---

## Próximos serviços

Os serviços abaixo serão instalados e documentados conforme o servidor for evoluindo.

| Serviço | Função | Status |
|---|---|---|
| **Let's Encrypt** | Certificados SSL gratuitos para HTTPS | Pendente |
| **Docker** | Orquestração de containers para isolar aplicações | Pendente |
| **PostgreSQL** | Banco de dados relacional | Pendente |
| **MySQL** | Banco de dados relacional alternativo | Pendente |
| **Node.js** | Runtime para backends JavaScript | Pendente |
| **Python** | Runtime para backends Python | Pendente |
| **DuckDNS** | DNS dinâmico para endereço fixo na internet | Pendente |