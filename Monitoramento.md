# Monitoramento

## Informações do Sistema

```bash
echo "=== Sistema ===" && lsb_release -a && echo && echo "=== Kernel ===" && uname -a && echo && echo "=== Uptime ===" && uptime && echo && echo "=== Disco ===" && df -h && echo && echo "=== Memoria ===" && free -h

```

## Verificar Logs de Acesso (Últimos 20)

Monitora quem está acessando o servidor em tempo real.

```bash
sudo tail -f -n 20 /var/log/nginx/access.log

```

## Verificar Logs de Erro

Diagnóstico para Erro 500 ou tela branca.

```bash
sudo tail -n 50 /var/log/nginx/error.log

```

## Limpeza de Disco e Logs

Remove pacotes inúteis e logs antigos do sistema para liberar espaço.

```bash
sudo apt autoremove -y && sudo apt clean && sudo journalctl --vacuum-time=1d && df -h

```

## Corrigir Permissões

Caso você tenha subido arquivos manualmente e o site esteja com erro de acesso.

> **SUBSTITUA antes de executar:**
> - `NOME_DA_PASTA` → Nome da pasta do projeto em /var/www/

```bash
fixPerms() { USER_WEB=$(whoami); if [ ! -d "/var/www/$1" ]; then echo "ERRO: Pasta /var/www/$1 não existe."; return 1; fi; sudo chown -R $USER_WEB:www-data "/var/www/$1" && sudo chmod -R 775 "/var/www/$1" && sudo find "/var/www/$1" -type f -name ".env*" -exec chmod 640 {} \; && echo "✓ Permissões corrigidas em $1"; }; fixPerms NOME_DA_PASTA

```

## Status dos Serviços

Verifica se Nginx, PHP-FPM e Fail2Ban estão rodando.

```bash
sudo systemctl status nginx php*-fpm fail2ban --no-pager

```

## Uso de Recursos em Tempo Real

Monitora CPU, RAM e disco.

```bash
watch -n 2 'free -h && echo && df -h / && echo && top -bn1 | head -n 12'

```

## Conexões Ativas (Top 10 IPs)

Identifica quais IPs estão fazendo mais requisições.

```bash
sudo tail -n 1000 /var/log/nginx/access.log | awk '{print $1}' | sort | uniq -c | sort -rn | head -n 10

```

## Verificar Uso de PHP-FPM

Monitora processos PHP ativos e consumo de memória.

```bash
ps aux | grep php-fpm | grep -v grep

```

## Verificar Configuração do Nginx

```bash
sudo nginx -t

```

## Recarregar Nginx (Sem Downtime)

```bash
sudo nginx -s reload

```

## Reiniciar Todos os Serviços

```bash
sudo systemctl restart nginx php*-fpm fail2ban && echo "✓ Nginx, PHP-FPM e Fail2Ban reiniciados"

```

## Ver Portas em Uso

```bash
sudo ss -tulpn

```

## Processos Consumindo Mais Recursos

```bash
ps aux --sort=-%mem | head -n 15

```

## Atualizar Sistema Completo

```bash
sudo apt update && sudo apt upgrade -y && sudo apt autoremove -y && echo "✓ Sistema atualizado"

```

## Verificar Atualizações de Segurança Pendentes

```bash
sudo apt list --upgradable 2>/dev/null | grep -i security

```

## Reiniciar Servidor (Se Necessário)

```bash
sudo reboot

```

## Verificar Quanto Tempo o Servidor Está Ligado

```bash
uptime

```

## Verificar Último Reboot e Logins

```bash
last reboot | head -n 5 && echo && last -n 10

```

## Verificação Final

```bash
echo "=== Verificação do Servidor ===" && echo && echo "1. Nginx:" && sudo nginx -t && echo && echo "2. PHP-FPM:" && sudo systemctl is-active php*-fpm && echo && echo "3. Fail2Ban:" && sudo systemctl is-active fail2ban && echo && echo "4. UFW:" && sudo ufw status | head -n 5 && echo && echo "5. SSL (se configurado):" && sudo certbot certificates 2>/dev/null || echo "SSL não configurado" && echo && echo "6. Swap:" && swapon --show && echo && echo "✓ Verificação completa!"

```
