# Domínios

## Configurar Domínio

Cria o Virtual Host com bloqueio de arquivos ocultos (.git, .env) e ocultação de versão.

> **SUBSTITUA antes de executar:**
> - `seusite.com.br` → Seu dominio (sem http://)
> - `PASTA_DO_PROJETO` → Nome da pasta em /var/www/ onde esta o projeto

```bash
DOMAIN="seusite.com.br"; FOLDER="PASTA_DO_PROJETO"; printf "server { listen 80; server_name %s www.%s; root /var/www/%s; index index.php index.html; server_tokens off; client_max_body_size 64M; limit_req zone=general burst=20 nodelay; limit_conn addr 10; location ~ /\.(?!well-known).* { deny all; } location ~* \\.(env|git|gitignore|htaccess|log|sql|sqlite)$ { deny all; } location ~* /uploads/.*\\.php$ { deny all; } location / { try_files \$uri \$uri/ =404; } location ~ \\.php$ { include snippets/fastcgi-php.conf; fastcgi_pass unix:%s; fastcgi_param PHP_VALUE \"upload_max_filesize=64M\\npost_max_size=64M\"; } }" "$DOMAIN" "$DOMAIN" "$FOLDER" "$(ls /run/php/php*-fpm.sock | head -n 1)" | sudo tee /etc/nginx/sites-available/$DOMAIN > /dev/null && sudo ln -sf /etc/nginx/sites-available/$DOMAIN /etc/nginx/sites-enabled/ && sudo nginx -t && sudo systemctl restart nginx && echo "✓ Domínio $DOMAIN configurado com proteções avançadas."

```

## Ativar SSL (Let's Encrypt)

Requer domínio apontado para o IP do servidor.

> **PRE-REQUISITO:** O dominio deve estar apontando para o IP do servidor (DNS propagado).
> Teste com: `ping seusite.com.br`

```bash
sudo apt install -y certbot python3-certbot-nginx && sudo certbot --nginx

```

## Ativar SSL para Domínio Específico

> **SUBSTITUA antes de executar:**
> - `seusite.com.br` → Seu dominio

```bash
sudo certbot --nginx -d seusite.com.br -d www.seusite.com.br

```

## Configurar Renovação Automática

O Certbot já configura renovação automática, mas você pode verificar:

```bash
sudo systemctl status certbot.timer && sudo certbot renew --dry-run

```
