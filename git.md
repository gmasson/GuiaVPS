# Git

## Adicionar Novo Projeto

Clona o repositório e define permissões seguras.

> **SUBSTITUA antes de executar:**
> - `SEU_USUARIO` → Seu usuario do GitHub
> - `SEU_REPO` → Nome do repositório
> - `NOME_DA_PASTA` → Nome da pasta onde ficara o projeto (ex: meusite)

```bash
addProject() { USER_WEB=$(whoami); if [ -d "/var/www/$2" ]; then echo "ERRO: Pasta /var/www/$2 ja existe."; return 1; fi; sudo git clone "$1" "/var/www/$2" && sudo chown -R $USER_WEB:www-data "/var/www/$2" && sudo chmod -R 775 "/var/www/$2" && sudo find "/var/www/$2" -type f -name ".env*" -exec chmod 640 {} \; && echo "✓ Projeto $2 clonado e permissões configuradas."; }; addProject https://github.com/SEU_USUARIO/SEU_REPO.git NOME_DA_PASTA

```

## Atualizar Projeto

Puxa alterações do GitHub e corrige permissões. Cria backup automático antes de atualizar.

> **SUBSTITUA antes de executar:**
> - `NOME_DA_PASTA` → Nome da pasta do projeto em /var/www/ (ex: meusite)

```bash
updateProject() { USER_WEB=$(whoami); if [ ! -d "/var/www/$1" ]; then echo "ERRO: Projeto $1 não encontrado."; return 1; fi; cd "/var/www/$1" && TIMESTAMP=$(date +%Y%m%d_%H%M%S) && sudo mkdir -p /var/backups && sudo tar -czf "/var/backups/${1}_${TIMESTAMP}.tar.gz" . 2>/dev/null && sudo git stash && sudo git pull && sudo chown -R $USER_WEB:www-data . && sudo chmod -R 775 . && sudo find . -type f -name ".env*" -exec chmod 640 {} \; && echo "✓ Projeto $1 atualizado. Backup salvo em /var/backups/${1}_${TIMESTAMP}.tar.gz"; }; updateProject NOME_DA_PASTA

```

## Reset Forçado (Git Hard Reset)

Apaga alterações locais e força o código a ficar idêntico ao GitHub. **CUIDADO: Alterações locais serão perdidas!**

> **SUBSTITUA antes de executar:**
> - `NOME_DA_PASTA` → Nome da pasta do projeto em /var/www/
> - `main` → Nome da branch principal (pode ser `master` em repos antigos)

```bash
forceReset() { USER_WEB=$(whoami); if [ ! -d "/var/www/$1" ]; then echo "ERRO: Projeto $1 não encontrado."; return 1; fi; cd "/var/www/$1" && sudo git fetch --all && sudo git reset --hard origin/main && sudo chown -R $USER_WEB:www-data . && sudo chmod -R 775 . && echo "✓ Reset concluído em $1."; }; forceReset NOME_DA_PASTA

```