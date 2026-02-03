# Segurança

## HTTP Headers

Injeta cabeçalhos de segurança globais no Nginx. Protege contra Clickjacking, XSS e Sniffing de MIME-Type.

> **NOTA:** O CSP (Content-Security-Policy) está restritivo. Se usar CDNs externos (Google Fonts, jQuery, etc), edite o arquivo após criar ou use o comando alternativo abaixo.

```bash
printf 'add_header X-Frame-Options "SAMEORIGIN" always;\nadd_header X-Content-Type-Options "nosniff" always;\nadd_header Referrer-Policy "strict-origin-when-cross-origin" always;\nadd_header Content-Security-Policy "default-src \'self\'; script-src \'self\'; style-src \'self\' \'unsafe-inline\'; img-src \'self\' data: https:; font-src \'self\' data:; connect-src \'self\'; frame-ancestors \'self\';" always;\nadd_header Permissions-Policy "geolocation=(), microphone=(), camera=()" always;' | sudo tee /etc/nginx/conf.d/security_headers.conf && sudo systemctl restart nginx && echo "✓ Cabeçalhos de Segurança HTTP aplicados (CSP restritivo)."

```

## CSP Permissivo (Para sites com CDNs externos)

Use este se seu site carrega recursos de CDNs como Google Fonts, cdnjs, unpkg, etc.

```bash
printf 'add_header X-Frame-Options "SAMEORIGIN" always;\nadd_header X-Content-Type-Options "nosniff" always;\nadd_header Referrer-Policy "strict-origin-when-cross-origin" always;\nadd_header Content-Security-Policy "default-src \'self\'; script-src \'self\' \'unsafe-inline\' \'unsafe-eval\' https://cdnjs.cloudflare.com https://unpkg.com https://cdn.jsdelivr.net; style-src \'self\' \'unsafe-inline\' https://fonts.googleapis.com https://cdnjs.cloudflare.com; img-src \'self\' data: https:; font-src \'self\' data: https://fonts.gstatic.com; connect-src \'self\' https:; frame-ancestors \'self\';" always;\nadd_header Permissions-Policy "geolocation=(), microphone=(), camera=()" always;' | sudo tee /etc/nginx/conf.d/security_headers.conf && sudo systemctl restart nginx && echo "✓ Cabeçalhos de Segurança HTTP aplicados (CSP permissivo para CDNs)."

```

## Blindagem PHP

Edita o php.ini para desativar funções que permitem a execução de comandos do sistema operacional via script (como exec, system, shell_exec). Impede que uma vulnerabilidade no site dê acesso total ao servidor.

> **IMPORTANTE:** Se usar Laravel Horizon, Laravel Queue, Symfony Process ou similares, use a versão alternativa abaixo que mantém funções `pcntl_*` necessárias para workers em background.

### Versão Segura Padrão (Recomendada)

```bash
INI_FILE=$(php --ini | grep "Loaded Configuration" | sed -e "s|.*:\s*||"); sudo sed -i 's/disable_functions = .*/disable_functions = exec,passthru,shell_exec,system,proc_open,popen,curl_exec,curl_multi_exec,parse_ini_file,show_source,pcntl_exec,pcntl_fork,pcntl_signal,pcntl_waitpid,pcntl_wexitstatus,pcntl_wifexited,pcntl_wifsignaled,pcntl_wifstopped,pcntl_wstopsig,pcntl_wtermsig,pcntl_sigprocmask,pcntl_sigwaitinfo,pcntl_sigtimedwait,pcntl_get_last_error,pcntl_strerror,pcntl_async_signals,dl,symlink,link,pfsockopen,phpinfo/' "$INI_FILE" && sudo sed -i 's/;opcache.enable=1/opcache.enable=1/' "$INI_FILE" && sudo sed -i 's/expose_php = On/expose_php = Off/' "$INI_FILE" && sudo sed -i 's/upload_max_filesize = .*/upload_max_filesize = 64M/' "$INI_FILE" && sudo sed -i 's/post_max_size = .*/post_max_size = 64M/' "$INI_FILE" && sudo systemctl restart php*-fpm && echo "PHP Blindado: Funcoes perigosas desativadas + OPcache ativo + expose_php off."

```

### Versão para Laravel/Frameworks Modernos (Com pcntl_* habilitado)

Use esta versão se precisar de filas/workers em background:

```bash
INI_FILE=$(php --ini | grep "Loaded Configuration" | sed -e "s|.*:\s*||"); sudo sed -i 's/disable_functions = .*/disable_functions = exec,passthru,shell_exec,system,proc_open,popen,curl_exec,curl_multi_exec,parse_ini_file,show_source,dl,symlink,link,pfsockopen,phpinfo/' "$INI_FILE" && sudo sed -i 's/;opcache.enable=1/opcache.enable=1/' "$INI_FILE" && sudo sed -i 's/expose_php = On/expose_php = Off/' "$INI_FILE" && sudo sed -i 's/upload_max_filesize = .*/upload_max_filesize = 64M/' "$INI_FILE" && sudo sed -i 's/post_max_size = .*/post_max_size = 64M/' "$INI_FILE" && sudo systemctl restart php*-fpm && echo "PHP Blindado (com pcntl): Funcoes perigosas desativadas + OPcache ativo + expose_php off."

```

## Alterar Porta SSH (Anti-Scanner)

Muda a porta padrão 22 para outra, dificultando ataques automatizados.

> **SUBSTITUA antes de executar:**
> - `2222` → Porta desejada (escolha entre 1024-65535, evite portas comuns como 80, 443, 3306)
>
> **⚠️ IMPORTANTE:** 
> 1. Abra DUAS sessões SSH antes de executar
> 2. Teste a nova porta em uma sessao antes de fechar a outra
> 3. Se algo der errado, use o console web do provedor para corrigir

```bash
NEW_PORT=2222; sudo ufw allow $NEW_PORT/tcp comment 'SSH Nova Porta' && sudo sed -i "s/^#Port 22/Port $NEW_PORT/" /etc/ssh/sshd_config && sudo sed -i "s/^Port 22/Port $NEW_PORT/" /etc/ssh/sshd_config && sudo systemctl restart sshd && echo "✓ Porta SSH alterada para $NEW_PORT" && echo "TESTE AGORA em outra aba: ssh -p $NEW_PORT \$(whoami)@\$(curl -s ifconfig.me)" && echo "Se funcionar, remova a porta 22: sudo ufw delete allow OpenSSH"

```

## Reverter Porta SSH para 22

Caso precise voltar a porta padrão.

```bash
sudo sed -i 's/^Port .*/Port 22/' /etc/ssh/sshd_config && sudo ufw allow OpenSSH && sudo systemctl restart sshd && echo "✓ Porta SSH revertida para 22"

```

## Desabilitar Login Root via SSH

Impede acesso direto como root. Usuários devem usar sudo.

```bash
sudo sed -i 's/^PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config && sudo sed -i 's/^#PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config && sudo systemctl restart sshd && echo "✓ Login root via SSH desabilitado."

```

## Configurar Autenticação por Chave SSH

Mais seguro que senha. Execute no seu computador LOCAL primeiro para gerar a chave:

```bash
# NO SEU COMPUTADOR LOCAL (não no servidor):
ssh-keygen -t ed25519 -C "seu_email@exemplo.com"
ssh-copy-id -i ~/.ssh/id_ed25519.pub usuario@ip_do_servidor

```

Depois, no servidor, desabilite autenticação por senha:

```bash
sudo sed -i 's/^#PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config && sudo sed -i 's/^PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config && sudo systemctl restart sshd && echo "✓ Autenticação por senha desabilitada. Use apenas chave SSH."

```

## Timeout de Sessão SSH Inativa

Desconecta sessões SSH inativas após 10 minutos.

```bash
echo -e "ClientAliveInterval 300\nClientAliveCountMax 2" | sudo tee -a /etc/ssh/sshd_config && sudo systemctl restart sshd && echo "✓ Timeout de 10 minutos configurado para sessões inativas."

```

## IPs Banidos pelo Fail2Ban

```bash
sudo fail2ban-client status sshd && echo && sudo fail2ban-client status nginx-limit-req

```

## Desbanir IP Bloqueado

> **SUBSTITUA antes de executar:**
> - `IP_AQUI` → Endereco IP que deseja desbloquear (ex: 192.168.1.100)
> - `sshd` → Pode trocar por `nginx-limit-req` se o bloqueio foi por excesso de requisições

```bash
sudo fail2ban-client set sshd unbanip IP_AQUI

```

## Desabilitar Listagem de Diretórios

Impede que usuários vejam o conteúdo de pastas sem index.

```bash
printf 'autoindex off;' | sudo tee /etc/nginx/conf.d/disable_autoindex.conf && sudo systemctl restart nginx && echo "✓ Listagem de diretórios desabilitada."

```

## Bloquear Bots Maliciosos

Cria mapa de bloqueio para bots conhecidos de scanner/ataque.

> **NOTA:** Use apenas se não precisar de webhooks ou comunicação externa via curl/wget.

```bash
printf 'map \$http_user_agent \$bad_bot { default 0; ~*nikto 1; ~*sqlmap 1; ~*masscan 1; ~*nmap 1; ~*dirbuster 1; ~*gobuster 1; ~*wpscan 1; ~*scrapy 1; }' | sudo tee /etc/nginx/conf.d/block_bots.conf && echo && echo 'IMPORTANTE: Adicione este bloco em cada server {} do nginx:' && echo 'if (\$bad_bot) { return 403; }' && sudo systemctl restart nginx && echo "✓ Mapa de bots criado. Configure o bloqueio em cada virtual host."

```

## Proteger Contra Slowloris (DoS)

Configura timeouts agressivos para prevenir ataques Slowloris.

```bash
printf 'client_body_timeout 10;\nclient_header_timeout 10;\nkeepalive_timeout 5 5;\nsend_timeout 10;' | sudo tee /etc/nginx/conf.d/timeouts.conf && sudo systemctl restart nginx && echo "✓ Protecao contra Slowloris ativada."

```

## Auditoria de Segurança (Lynis)

Instala e executa auditoria completa de segurança do servidor.

```bash
sudo apt install -y lynis && sudo lynis audit system --quick

```

## Verificar Certificado SSL

> **SUBSTITUA antes de executar:**
> - `seusite.com.br` → Seu dominio

```bash
DOMAIN="seusite.com.br"; echo | openssl s_client -servername $DOMAIN -connect $DOMAIN:443 2>/dev/null | openssl x509 -noout -dates

```

## Renovar Certificados SSL Manualmente

```bash
sudo certbot renew --dry-run && echo "✓ Teste de renovação OK" || echo "✗ Erro na renovação"

```

## Forçar Renovação de Certificado

> **SUBSTITUA antes de executar:**
> - `seusite.com.br` → Seu dominio

```bash
sudo certbot certonly --nginx -d seusite.com.br -d www.seusite.com.br --force-renewal

```
