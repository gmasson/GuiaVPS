# GuiaVPS

Este guia foi criado para ajudar desenvolvedores iniciantes a configurarem seus próprios servidores web de forma segura e eficiente. Os comandos estão organizados por tópicos e podem ser executados em sequência, eliminando a necessidade de conhecimento avançado em administração de servidores.

**Stack:** Nginx + PHP-FPM + SQLite  
**Sistema:** Ubuntu 20.04 LTS, 22.04 LTS e 24.04 LTS

## Documentacao

| Topico | Descricao |
|--------|-----------|
| [Instalacao](Instalação.md) | Instalacao completa da stack |
| [Seguranca](Segurança.md) | Headers HTTP, blindagem PHP, SSH e Fail2Ban |
| [Dominios](Domínios.md) | Virtual Hosts e certificados SSL |
| [Git](git.md) | Deploy e atualizacao de projetos |
| [Backups](Backups.md) | Backup manual e automatico com Cron |
| [Performance](Performance.md) | Swap, Gzip, cache e otimizacao OPcache |
| [Monitoramento](Monitoramento.md) | Logs, recursos e status dos servicos |
| [Utilitarios](Utilitários.md) | Gerenciamento de sites e solucao de problemas |

## Pre-requisitos

- Servidor VPS com Ubuntu instalado
- Acesso SSH ao servidor
- Conhecimento basico de terminal Linux

## Como Usar

1. Clone, baixe ou somente deixe aberto este repositorio para consulta rápida
2. Acesse o servidor via SSH
3. Siga os comandos do arquivo [Instalacao](Instalação.md) para configurar a stack
4. Consulte os demais arquivos conforme necessidade

**Importante:** Execute os comandos na ordem apresentada. Sempre faca backup antes de alterar configuracoes do sistema.

**Dica:** Use `Ctrl+F` para localizar comandos especificos dentro dos arquivos.

## Configuracao por Provedor

Antes de iniciar, verifique as configuracoes de firewall no painel do seu provedor.

### AWS EC2

| Configuracao | Valor |
|--------------|-------|
| Usuario padrao | `ubuntu` |
| Firewall | Security Group da instancia |
| Portas | 22 (SSH), 80 (HTTP), 443 (HTTPS) |

### Google Cloud Platform

| Configuracao | Valor |
|--------------|-------|
| Usuario padrao | Seu usuario do Google (use `sudo`) |
| Firewall | VPC Network > Firewall > Criar regra |
| Portas | tcp:80, tcp:443 com tag `http-server` |

### Oracle Cloud

| Configuracao | Valor |
|--------------|-------|
| Usuario padrao | `ubuntu` ou `opc` (Oracle Linux) |
| Firewall Painel | Virtual Cloud Networks > Security Lists |
| Firewall Sistema | Requer liberacao manual do iptables |

**Atencao:** A Oracle bloqueia portas no iptables do sistema. Execute antes da instalacao:

```bash
sudo iptables -F && sudo netfilter-persistent save
```

Este comando limpa todas as regras de firewall. Use apenas em servidores novos.

### DigitalOcean, Hostinger, Vultr, Linode

| Configuracao | Valor |
|--------------|-------|
| Usuario padrao | `root` |
| Firewall | Geralmente liberado por padrao |
| Nota | Comando `sudo` e opcional |

## O que sera instalado

**Servidor Web**
- Nginx (servidor web de alta performance)
- PHP-FPM (interpretador PHP)
- Extensoes PHP: sqlite3, mbstring, xml, bcmath, gd, intl, curl, opcache, mysql, readline, gmp, redis

**Banco de Dados**
- SQLite3 (banco de dados leve e sem servidor separado)

**Ferramentas**
- Git, Unzip, Curl

**Seguranca**
- UFW (firewall simplificado)
- Fail2Ban (protecao contra forca bruta)
- Unattended-Upgrades (atualizacoes automaticas)

## O que sera configurado

- Firewall bloqueando todas as portas exceto SSH, HTTP e HTTPS
- Rate limiting para prevenir ataques DDoS (15 req/s geral, 5 req/min login)
- Fail2Ban monitorando SSH e Nginx
- Permissoes seguras para arquivos web
- Virtual Host padrao em `/var/www/html`

## Contribuicao

Contribuicoes sao bem-vindas. Para contribuir:

1. Faca um fork do projeto
2. Crie uma branch para sua feature (`git checkout -b feature/nova-feature`)
3. Commit suas alteracoes (`git commit -m 'Adiciona nova feature'`)
4. Push para a branch (`git push origin feature/nova-feature`)
5. Abra um Pull Request

Para reportar bugs ou sugerir melhorias, abra uma issue.

## Licenca

Este projeto esta licenciado sob a [MIT License](LICENSE).
