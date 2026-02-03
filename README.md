# GuiaVPS v0.1

Guia completo de comandos para configurar e gerenciar servidores web seguros com Nginx + PHP-FPM + SQLite.

**Compatibilidade:** Ubuntu 20.04 LTS, 22.04 LTS e 24.04 LTS  
**Provedores sugeridos:** AWS EC2, Google Cloud, Oracle Cloud, DigitalOcean, Hostinger, Vultr, Linode

Este projeto educacional foi criado para facilitar a configuração de servidores web seguros. Ele contém comandos organizados por tópicos, permitindo que você configure e gerencie seu servidor de forma eficiente.

> **DICA:** Use `Ctrl+F` para buscar comandos específicos.

## Índice

- [Instalação](Instalação.md)
- [Segurança](Segurança.md)
- [Git](git.md)
- [Domínios](Domínios.md)
- [Backups](Backups.md)
- [Performance](Performance.md)
- [Monitoramento](Monitoramento.md)
- [Utilitários](Utilitários.md)

---

## Pré-requisitos

- Acesso SSH ao servidor VPS
- Conhecimento básico de linha de comando Linux
- Ubuntu instalado (versões suportadas acima)

## Como Usar

1. Clone ou baixe este repositório.
2. Navegue pelos arquivos .md conforme o tópico desejado.
3. Copie e execute os comandos na sequência que precisar em seu servidor.

> **Atenção:** Sempre faça backup antes de executar comandos que alterem configurações do sistema.

---

## Notas

Antes de rodar os comandos, verifique as regras de Firewall do painel da sua hospedagem.

### AWS (EC2)
* Usuário padrão: ubuntu
* Firewall: Edite o "Security Group" associado à instância.
* Portas para liberar: 80 (HTTP), 443 (HTTPS), 22 (SSH).

### Google Cloud Platform (Compute Engine)
* Usuário padrão: Depende do seu login (use 'sudo' sempre).
* Firewall: Menu "VPC Network" > "Firewall". Crie uma regra liberando tcp:80 e tcp:443 com a tag "http-server".

### Oracle Cloud
* Usuário padrão: ubuntu (para imagens Ubuntu) ou opc (Oracle Linux).
* Firewall (Painel): Menu "Virtual Cloud Networks" > "Security Lists". Adicione Ingress Rules para 80 e 443.
* Firewall (Sistema): A Oracle bloqueia portas no iptables do sistema. Rode este comando extra antes de instalar a stack:
    `sudo iptables -F && sudo netfilter-persistent save`
* **ATENÇÃO**: Este comando limpa TODAS as regras de firewall. Use apenas em servidores novos.

### DigitalOcean / Hostinger / Vultr
* Usuário padrão: root
* Firewall: Geralmente vem liberado. Se usar UFW, rode `ufw allow 80/tcp`.
* Nota: Como você já é root, o comando 'sudo' é opcional, mas os scripts abaixo funcionam mesmo assim.

---

## Contribuição

Contribuições são bem-vindas! Sinta-se à vontade para abrir issues ou pull requests.

## Licença

Este projeto está licenciado sob a [MIT License](LICENSE.md).
