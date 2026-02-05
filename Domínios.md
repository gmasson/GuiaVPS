# Domínios

## Configurar Domínio

Cria Virtual Host com proteções de segurança em múltiplos passos.

> **SUBSTITUA antes de executar:**
> - `seusite.com.br` → Seu dominio (sem http://)
> - `PASTA_DO_PROJETO` → Nome da pasta em /var/www/

### Comando 1: Criar Configuração Base do Domínio

```bash
DOMAIN="seusite.com.br"; FOLDER="PASTA_DO_PROJETO"; printf "server {\n  listen 80;\n  server_name %s www.%s;\n  root /var/www/%s;\n  index index.php index.html;\n  server_tokens off;\n  client_max_body_size 64M;\n  limit_req zone=general burst=20 nodelay;\n  limit_conn addr 10;\n" "$DOMAIN" "$DOMAIN" "$FOLDER" | sudo tee /etc/nginx/sites-available/$DOMAIN > /dev/null && echo "✓ Configuração base criada"
```

### Comando 2: Adicionar Bloqueios de Segurança

```bash
DOMAIN="seusite.com.br"; printf "  location ~ /\\.(?!well-known).* { deny all; }\n  location ~* \\.(env|git|gitignore|htaccess|log|sql|sqlite)$ { deny all; }\n  location ~* /uploads/.*\\.php$ { deny all; }\n" | sudo tee -a /etc/nginx/sites-available/$DOMAIN > /dev/null && echo "✓ Bloqueios de segurança adicionados"
```

### Comando 3: Configurar Processamento de Arquivos

```bash
DOMAIN="seusite.com.br"; printf "  location / { try_files \$uri \$uri/ =404; }\n  location ~ \\.php$ {\n    include snippets/fastcgi-php.conf;\n    fastcgi_pass unix:%s;\n    fastcgi_param PHP_VALUE \"upload_max_filesize=64M\\npost_max_size=64M\";\n  }\n}\n" "$(ls /run/php/php*-fpm.sock | head -n 1)" | sudo tee -a /etc/nginx/sites-available/$DOMAIN > /dev/null && echo "✓ Processamento PHP configurado"
```

### Comando 4: Ativar Site

```bash
DOMAIN="seusite.com.br"; sudo ln -sf /etc/nginx/sites-available/$DOMAIN /etc/nginx/sites-enabled/ && echo "✓ Site ativado"
```

### Comando 5: Testar e Aplicar

```bash
sudo nginx -t && sudo systemctl restart nginx && echo "✓ Domínio configurado com sucesso"
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
