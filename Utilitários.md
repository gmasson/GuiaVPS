# Utilitários

## Listar Todos os Sites Configurados

```bash
echo "Sites disponíveis:" && ls -la /etc/nginx/sites-available/ && echo && echo "Sites ativos:" && ls -la /etc/nginx/sites-enabled/

```

## Desativar Site (Sem Deletar)

> **SUBSTITUA antes de executar:**
> - `seusite.com.br` → Dominio do site que deseja desativar

```bash
DOMAIN="seusite.com.br"; sudo rm /etc/nginx/sites-enabled/$DOMAIN && sudo nginx -t && sudo systemctl restart nginx && echo "✓ Site $DOMAIN desativado (arquivos mantidos em sites-available)"

```

## Reativar Site

> **SUBSTITUA antes de executar:**
> - `seusite.com.br` → Dominio do site que deseja reativar

```bash
DOMAIN="seusite.com.br"; sudo ln -sf /etc/nginx/sites-available/$DOMAIN /etc/nginx/sites-enabled/ && sudo nginx -t && sudo systemctl restart nginx && echo "✓ Site $DOMAIN reativado"

```

## Deletar Site Completamente

> **SUBSTITUA antes de executar:**
> - `seusite.com.br` → Dominio do site
> - `PASTA_DO_PROJETO` → Pasta do projeto em /var/www/
> **CUIDADO: Esta ação é irreversível!**

```bash
DOMAIN="seusite.com.br"; FOLDER="PASTA_DO_PROJETO"; sudo rm -f /etc/nginx/sites-enabled/$DOMAIN /etc/nginx/sites-available/$DOMAIN && sudo rm -rf /var/www/$FOLDER && sudo nginx -t && sudo systemctl restart nginx && echo "✓ Site $DOMAIN e pasta $FOLDER deletados permanentemente"

```

## Listar Bancos SQLite no Projeto

> **SUBSTITUA antes de executar:**
> - `NOME_DA_PASTA` → Nome da pasta do projeto

```bash
find /var/www/NOME_DA_PASTA -name "*.sqlite" -o -name "*.db" -o -name "*.sqlite3" 2>/dev/null

```

## Backup de Banco SQLite

> **SUBSTITUA antes de executar:**
> - `/var/www/projeto/database.sqlite` → Caminho completo do banco

```bash
DB_PATH="/var/www/projeto/database.sqlite"; TIMESTAMP=$(date +%Y%m%d_%H%M%S); sudo cp "$DB_PATH" "${DB_PATH}_backup_${TIMESTAMP}" && echo "✓ Backup criado: ${DB_PATH}_backup_${TIMESTAMP}"

```

## Verificar Integridade do Banco SQLite

> **SUBSTITUA antes de executar:**
> - `/var/www/projeto/database.sqlite` → Caminho completo do banco

```bash
sqlite3 /var/www/projeto/database.sqlite "PRAGMA integrity_check;"

```

## Otimizar Banco SQLite (VACUUM)

Compacta o banco removendo espaços vazios. Pode melhorar performance.

> **SUBSTITUA antes de executar:**
> - `/var/www/projeto/database.sqlite` → Caminho completo do banco

```bash
sqlite3 /var/www/projeto/database.sqlite "VACUUM;" && echo "✓ Banco otimizado"

```

## Site Retornando Erro 502 Bad Gateway

Geralmente indica que PHP-FPM parou.

```bash
sudo systemctl status php*-fpm && sudo systemctl restart php*-fpm && sudo systemctl restart nginx

```

## Site Retornando Erro 403 Forbidden

Problema de permissões. Verifique e corrija:

> **SUBSTITUA antes de executar:**
> - `NOME_DA_PASTA` → Nome da pasta do projeto

```bash
ls -la /var/www/NOME_DA_PASTA && echo "---" && fixPerms() { USER_WEB=$(whoami); sudo chown -R $USER_WEB:www-data "/var/www/$1" && sudo chmod -R 775 "/var/www/$1" && echo "✓ Permissoes corrigidas"; }; fixPerms NOME_DA_PASTA

```

## Site Retornando Erro 500

Verifique os logs de erro do PHP e Nginx:

```bash
echo "=== Erro Nginx ===" && sudo tail -n 30 /var/log/nginx/error.log && echo && echo "=== Erro PHP ===" && sudo tail -n 30 /var/log/php*-fpm.log 2>/dev/null

```

## PHP-FPM Usando Muita Memória

Ajusta número máximo de processos PHP (recomendado para VPS com pouca RAM).

```bash
PHP_VER=$(php -v | head -n 1 | awk '{print $2}' | cut -d. -f1,2); sudo sed -i 's/^pm.max_children = .*/pm.max_children = 5/' /etc/php/$PHP_VER/fpm/pool.d/www.conf && sudo sed -i 's/^pm.start_servers = .*/pm.start_servers = 2/' /etc/php/$PHP_VER/fpm/pool.d/www.conf && sudo sed -i 's/^pm.min_spare_servers = .*/pm.min_spare_servers = 1/' /etc/php/$PHP_VER/fpm/pool.d/www.conf && sudo sed -i 's/^pm.max_spare_servers = .*/pm.max_spare_servers = 3/' /etc/php/$PHP_VER/fpm/pool.d/www.conf && sudo systemctl restart php*-fpm && echo "✓ PHP-FPM otimizado para baixa RAM"

```

## Servidor Lento ou Travando

Verifique recursos e processos:

```bash
echo "=== Memória ===" && free -h && echo && echo "=== Swap ===" && swapon --show && echo && echo "=== Disco ===" && df -h && echo && echo "=== Top Processos (CPU) ===" && ps aux --sort=-%cpu | head -n 10 && echo && echo "=== Top Processos (RAM) ===" && ps aux --sort=-%mem | head -n 10

```

## Servidor Travado - Matar Processos PHP

```bash
sudo pkill -9 php-fpm && sudo systemctl restart php*-fpm nginx && echo "✓ PHP-FPM reiniciado"

```

## Limpar TODA a Cache (Nginx + PHP + Sistema)

```bash
sudo systemctl restart php*-fpm nginx && sync && echo 3 | sudo tee /proc/sys/vm/drop_caches > /dev/null && echo "✓ Cache limpa"

```
