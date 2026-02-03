# Instalação

Este comando realiza a instalação completa e aplica as primeiras camadas de segurança.

Copie e cole:

```bash
USER_WEB=$(whoami); sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt install -y nginx php-fpm php-sqlite3 sqlite3 git unzip curl ufw php-mbstring php-xml php-bcmath php-gd php-intl php-curl php-opcache php-mysql php-readline php-gmp php-tokenizer php-fileinfo php-redis fail2ban unattended-upgrades && sudo ufw allow OpenSSH && sudo ufw allow 'Nginx Full' && sudo ufw --force enable && sudo chown -R $USER_WEB:www-data /var/www/html && sudo chmod -R 775 /var/www/html && printf "limit_req_zone \$binary_remote_addr zone=general:10m rate=15r/s;\nlimit_req_zone \$binary_remote_addr zone=login:10m rate=5r/m;\nlimit_conn_zone \$binary_remote_addr zone=addr:10m;" | sudo tee /etc/nginx/conf.d/rate_limit.conf > /dev/null && printf "server { listen 80 default_server; root /var/www/html; index index.php index.html; server_name _; server_tokens off; client_max_body_size 64M; limit_req zone=general burst=20 nodelay; limit_conn addr 10; location ~ /\.git { deny all; } location / { try_files \$uri \$uri/ =404; } location ~ \.php$ { include snippets/fastcgi-php.conf; fastcgi_pass unix:%s; } }" "$(ls /run/php/php*-fpm.sock | head -n 1)" | sudo tee /etc/nginx/sites-available/default > /dev/null && printf "[DEFAULT]\nbantime = 3600\nfindtime = 600\nmaxretry = 5\n\n[sshd]\nenabled = true\nport = ssh\nlogpath = /var/log/auth.log\n\n[nginx-http-auth]\nenabled = true\nfilter = nginx-http-auth\nport = http,https\nlogpath = /var/log/nginx/error.log\n\n[nginx-limit-req]\nenabled = true\nfilter = nginx-limit-req\nport = http,https\nlogpath = /var/log/nginx/error.log\nmaxretry = 10" | sudo tee /etc/fail2ban/jail.local > /dev/null && sudo systemctl enable fail2ban && sudo systemctl restart fail2ban && sudo dpkg-reconfigure -plow unattended-upgrades && sudo systemctl restart nginx && echo "Stack Segura Instalada. Usuário: $USER_WEB | Firewall UFW + Fail2Ban ativos | Rate Limiting configurado"

```

## O que será instalado:

**Servidor Web e PHP:**
- **Nginx** - Servidor web de alta performance
- **PHP-FPM** - Interpretador PHP (Fast Process Manager)
- **Extensoes PHP**: sqlite3, mbstring, xml, bcmath, gd, intl, curl, opcache, mysql, readline, gmp, tokenizer, fileinfo, redis

**Banco de Dados:**
- **SQLite3** - Banco de dados leve e sem necessidade de servidor separado

**Ferramentas de Desenvolvimento:**
- **Git** - Controle de versao para gerenciar projetos
- **Unzip/Curl** - Utilitários para downloads e descompactação

**Seguranca:**
- **UFW (Uncomplicated Firewall)** - Firewall simplificado do Linux
   - **Fail2Ban** - Proteção contra ataques de força bruta (banimento automático de IPs)
- **Unattended-Upgrades** - Atualizações automáticas de segurança

## O que será configurado:

**1. Firewall (UFW):**
   - Bloqueia TODAS as portas por padrao
   - Libera apenas: SSH (22), HTTP (80) e HTTPS (443)
   - Ativa proteção automática contra ataques

**2. Permissões de Arquivos:**
   - Define seu usuario como dono dos arquivos em /var/www/html
   - Grupo www-data (usado pelo Nginx) tem acesso para leitura/escrita
   - Permissões 775 (dono e grupo podem tudo, outros podem ler)

**3. Rate Limiting (Limite de Requisições):**
   - **Zone General**: Máximo 15 requisições por segundo por IP
   - **Zone Login**: Máximo 5 tentativas de login por minuto por IP
   - **Limit Connections**: Máximo 10 conexões simultâneas por IP
   - Previne ataques DDoS e abuso de recursos

**4. Configuração do Nginx:**
   - Virtual Host padrao criado em /var/www/html
   - `server_tokens off` - Oculta versao do Nginx (previne exploits direcionados)
   - `client_max_body_size 64M` - Permite uploads de ate 64MB (padrao recomendado)
   - Bloqueia acesso a pasta `.git` (previne vazamento de código)
   - Configurado para processar PHP automaticamente

**5. Fail2Ban (Defesa Ativa):**
   - **SSH**: Bane IP apos 5 tentativas de login falhas em 10 minutos (banimento: 1 hora)
   - **Nginx HTTP Auth**: Protege áreas com autenticação básica
   - **Nginx Rate Limit**: Bane IPs que ultrapassam 10 requisições no limite configurado
   - Logs monitorados: /var/log/auth.log e /var/log/nginx/error.log

**6. Atualizações Automáticas:**
   - Sistema configurado para instalar atualizações de segurança automaticamente
   - Reduz janela de vulnerabilidade do servidor
