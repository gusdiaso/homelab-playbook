# HomeLab

Servidor doméstico configurado do zero com Ubuntu Server, construído com segurança como prioridade e usado como laboratório de aprendizado contínuo.

---

## Sobre o projeto

Este repositório documenta a configuração completa de um servidor self-hosted rodando em uma máquina física em casa. O objetivo é duplo: ter infraestrutura própria para hospedar projetos reais, e usar o ambiente para aprender na prática. Cada tecnologia estudada tem um lugar real para rodar, não apenas tutoriais em máquina virtual.

O servidor hospeda sites estáticos, aplicações web, backends e bancos de dados, todos acessíveis pela internet com HTTPS e domínio próprio. Toda a configuração foi feita com segurança em mente desde o início: root bloqueado, firewall restritivo, portas mínimas expostas e autenticação endurecida.

---

## Capacidades

- Hospedar sites estáticos e aplicações web acessíveis pela internet
- Rodar backends em Node.js, Python e outras stacks
- Servir bancos de dados de forma isolada e segura
- Expor APIs com HTTPS e domínio próprio
- Orquestrar serviços com Docker e Docker Compose
- Ser administrado remotamente via SSH de qualquer lugar

---

## Stack

| Categoria | Tecnologia |
|---|---|
| Sistema operacional | Ubuntu Server 24.04 LTS |
| Servidor web | Nginx |
| Containers | Docker + Docker Compose |
| Backend | Node.js / Python |
| Banco de dados | PostgreSQL / MySQL |
| SSL | Let's Encrypt |
| DNS dinâmico | DuckDNS |
| Firewall | UFW |
| Proteção contra força bruta | Fail2Ban |

---

## O que estou aprendendo

- Administração de sistemas Linux
- Redes: DNS, proxies, TLS, firewall
- Docker e orquestração de containers
- Deploy e manutenção de aplicações em produção
- Bancos de dados relacionais e não-relacionais
- Monitoramento de infraestrutura
- CI/CD e automação

---

## Estrutura do repositório

```
/
├── docs/
│   ├── setup.md        # Configuração base do servidor
│   └── services.md     # Serviços instalados
├── nginx/              # Configurações de virtual hosts e proxy reverso
├── docker/             # Compose files e configurações de containers
├── scripts/            # Scripts de automação, backup e manutenção
└── apps/               # Configurações específicas de cada aplicação
```

---

## Documentação

| Documento | Descrição |
|---|---|
| [`docs/setup.md`](./docs/setup.md) | Configuração base: usuário, rede, SSH, firewall e Fail2Ban |
| [`docs/services.md`](./docs/services.md) | Serviços instalados: Nginx, Docker, bancos de dados e mais |

---

## Status

Em andamento.