# Backups

## Backup Completo de Projeto

Cria backup compactado com timestamp na pasta /var/backups.

> **SUBSTITUA antes de executar:**
> - `NOME_DA_PASTA` → Nome da pasta do projeto em /var/www/

```bash
backupProject() { if [ ! -d "/var/www/$1" ]; then echo "ERRO: Projeto $1 não encontrado."; return 1; fi; TIMESTAMP=$(date +%Y%m%d_%H%M%S); sudo mkdir -p /var/backups && sudo tar -czf "/var/backups/${1}_${TIMESTAMP}.tar.gz" "/var/www/$1" && echo "✓ Backup criado: /var/backups/${1}_${TIMESTAMP}.tar.gz ($(du -h /var/backups/${1}_${TIMESTAMP}.tar.gz | cut -f1))"; }; backupProject NOME_DA_PASTA

```

## Restaurar Backup

Restaurar de um arquivo de backup específico.

> **SUBSTITUA antes de executar:**
> - `ARQUIVO_BACKUP.tar.gz` → Nome do arquivo de backup (use o comando anterior para listar)
> - `NOME_DA_PASTA` → Nome da pasta do projeto em /var/www/

```bash
restoreBackup() { USER_WEB=$(whoami); if [ ! -f "$1" ]; then echo "ERRO: Arquivo $1 não encontrado."; return 1; fi; sudo tar -xzf "$1" -C / && sudo chown -R $USER_WEB:www-data "/var/www/$2" && sudo chmod -R 775 "/var/www/$2" && echo "✓ Backup restaurado: $1"; }; restoreBackup /var/backups/ARQUIVO_BACKUP.tar.gz NOME_DA_PASTA

```

## Listar Últimos Backups Disponíveis

```bash
ls -lh /var/backups/*.tar.gz

```

## Backup Automático Diário (Cron)

Configura backup automático todos os dias as 3h da manha, mantendo apenas os últimos 7 dias.

> **SUBSTITUA antes de executar:**
> - `NOME_DA_PASTA` → Nome da pasta do projeto em /var/www/

```bash
PROJECT="NOME_DA_PASTA"; (crontab -l 2>/dev/null; echo "0 3 * * * sudo tar -czf /var/backups/${PROJECT}_\$(date +\%Y\%m\%d).tar.gz /var/www/$PROJECT && find /var/backups -name '${PROJECT}_*.tar.gz' -mtime +7 -delete") | crontab - && echo "✓ Backup automático configurado para $PROJECT (diário às 3h)"

```

## Listar Tarefas Agendadas (Cron)

```bash
crontab -l

```

## Remover Backup Automático

> **SUBSTITUA antes de executar:**
> - `NOME_DA_PASTA` → Nome da pasta do projeto

```bash
PROJECT="NOME_DA_PASTA"; crontab -l | grep -v "$PROJECT" | crontab - && echo "✓ Backup automático removido para $PROJECT"

```
