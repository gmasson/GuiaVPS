# Performance

## Swap Anti-Crash

Cria swap automaticamente baseado na RAM disponível (recomendado para servidores com menos de 2GB).

> **NOTA:** Execute apenas uma vez. O comando verifica se ja existe swap configurado.

```bash
if [ -f /swapfile ]; then echo "AVISO: Swap ja existe. Use 'swapon --show' para verificar."; else RAM_MB=$(free -m | awk '/^Mem:/{print $2}'); SWAP_SIZE=$((RAM_MB < 512 ? 1024 : RAM_MB < 1024 ? 1024 : RAM_MB < 2048 ? 2048 : RAM_MB < 4096 ? 2048 : 1024)); sudo fallocate -l ${SWAP_SIZE}M /swapfile && sudo chmod 600 /swapfile && sudo mkswap /swapfile && sudo swapon /swapfile && echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab && sudo sysctl vm.swappiness=10 && echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf && echo "✓ Swap de ${SWAP_SIZE}MB ativado (RAM: ${RAM_MB}MB)"; fi

```

## Verificar Swap Atual

```bash
swapon --show && free -h | grep -i swap

```

## Remover Swap (Se Necessário)

```bash
sudo swapoff /swapfile && sudo rm /swapfile && sudo sed -i '/swapfile/d' /etc/fstab && echo "✓ Swap removido"

```

## Habilitar Compressão Gzip no Nginx

Reduz tamanho dos arquivos transferidos em até 70%.

```bash
printf 'gzip on;\ngzip_vary on;\ngzip_proxied any;\ngzip_comp_level 6;\ngzip_types text/plain text/css text/xml text/javascript application/json application/javascript application/xml+rss application/rss+xml font/truetype font/opentype application/vnd.ms-fontobject image/svg+xml;\ngzip_disable "msie6";' | sudo tee /etc/nginx/conf.d/gzip.conf && sudo systemctl restart nginx && echo "✓ Compressão Gzip ativada."

```

## Habilitar Cache de Assets Estáticos

Configura cache de 1 ano para arquivos estáticos (imagens, CSS, JS).

> **NOTA:** Este snippet deve ser incluído dentro de um bloco `server {}`. Para aplicar globalmente, adicione manualmente em cada Virtual Host ou use dentro do bloco server do seu site.

```bash
echo '# Adicione este bloco dentro do server {} do seu site em /etc/nginx/sites-available/seusite.com.br
# location ~* \.(jpg|jpeg|png|gif|ico|css|js|svg|woff|woff2|ttf|eot|webp|avif)$ {
#     expires 1y;
#     add_header Cache-Control "public, immutable";
#     access_log off;
# }' && echo "" && echo "✓ Copie o bloco acima para o arquivo de configuração do seu site."

```

## Cache Global via Mapa (Alternativa)

Configura cache baseado em tipo de arquivo globalmente.

```bash
printf 'map \$sent_http_content_type \$expires {\n    default                    off;\n    text/html                  epoch;\n    text/css                   1y;\n    application/javascript     1y;\n    ~image/                    1y;\n    ~font/                     1y;\n}\n' | sudo tee /etc/nginx/conf.d/cache_expires.conf && sudo systemctl restart nginx && echo "✓ Mapa de cache global configurado."

```

## Otimizar OPcache do PHP

Melhora performance do PHP em até 3x configurando cache de bytecode.

### Comando 1: Aumentar Memória do OPcache

```bash
INI_FILE=$(php --ini | grep "Loaded Configuration" | sed -e "s|.*:\s*||"); sudo sed -i 's/;opcache.memory_consumption=.*/opcache.memory_consumption=256/' "$INI_FILE" && echo "✓ Memória OPcache: 256MB"
```

### Comando 2: Buffer de Strings Internas

```bash
INI_FILE=$(php --ini | grep "Loaded Configuration" | sed -e "s|.*:\s*||"); sudo sed -i 's/;opcache.interned_strings_buffer=.*/opcache.interned_strings_buffer=16/' "$INI_FILE" && echo "✓ Buffer de strings: 16MB"
```

### Comando 3: Limite de Arquivos em Cache

```bash
INI_FILE=$(php --ini | grep "Loaded Configuration" | sed -e "s|.*:\s*||"); sudo sed -i 's/;opcache.max_accelerated_files=.*/opcache.max_accelerated_files=10000/' "$INI_FILE" && echo "✓ Máximo de arquivos: 10000"
```

### Comando 4: Frequência de Revalidação

```bash
INI_FILE=$(php --ini | grep "Loaded Configuration" | sed -e "s|.*:\s*||"); sudo sed -i 's/;opcache.revalidate_freq=.*/opcache.revalidate_freq=60/' "$INI_FILE" && echo "✓ Revalidação: 60 segundos"
```

### Comando 5: Aplicar Configurações

```bash
sudo systemctl restart php*-fpm && echo "✓ OPcache otimizado e aplicado"
```

## Limitar Workers do Nginx (Para VPS com pouca RAM)

```bash
sudo sed -i 's/worker_processes.*/worker_processes 1;/' /etc/nginx/nginx.conf && sudo sed -i '/worker_processes/a worker_rlimit_nofile 2048;' /etc/nginx/nginx.conf && sudo systemctl restart nginx && echo "✓ Workers do Nginx otimizados para baixa RAM."

```
