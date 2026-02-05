# Segurança

## HTTP Headers - Segurança Básica

Configura cabeçalhos de segurança essenciais (X-Frame-Options, X-Content-Type-Options, Referrer-Policy).

### Comando 1: Headers Básicos de Proteção

```bash
printf 'add_header X-Frame-Options "SAMEORIGIN" always;\nadd_header X-Content-Type-Options "nosniff" always;\nadd_header Referrer-Policy "strict-origin-when-cross-origin" always;' | sudo tee /etc/nginx/conf.d/security_headers.conf && echo "✓ Headers básicos de segurança aplicados"
```

### Comando 2: Content Security Policy (CSP Restritivo)

Adiciona CSP restritivo. Use apenas se seu site NÃO usa CDNs externos.

```bash
printf '\nadd_header Content-Security-Policy "default-src \'self\'; script-src \'self\'; style-src \'self\' \'unsafe-inline\'; img-src \'self\' data: https:; font-src \'self\' data:; connect-src \'self\'; frame-ancestors \'self\';" always;' | sudo tee -a /etc/nginx/conf.d/security_headers.conf && echo "✓ CSP restritivo adicionado"
```

### Comando 2 (Alternativa): CSP Permissivo para CDNs

Use este ao invés do anterior se usar Google Fonts, cdnjs, unpkg, etc.

```bash
printf '\nadd_header Content-Security-Policy "default-src \'self\'; script-src \'self\' \'unsafe-inline\' \'unsafe-eval\' https://cdnjs.cloudflare.com https://unpkg.com https://cdn.jsdelivr.net; style-src \'self\' \'unsafe-inline\' https://fonts.googleapis.com https://cdnjs.cloudflare.com; img-src \'self\' data: https:; font-src \'self\' data: https://fonts.gstatic.com; connect-src \'self\' https:; frame-ancestors \'self\';" always;' | sudo tee -a /etc/nginx/conf.d/security_headers.conf && echo "✓ CSP permissivo para CDNs adicionado"
```

### Comando 3: Permissions Policy

Bloqueia acesso a recursos do dispositivo (localização, microfone, câmera).

```bash
printf '\nadd_header Permissions-Policy "geolocation=(), microphone=(), camera=()" always;' | sudo tee -a /etc/nginx/conf.d/security_headers.conf && echo "✓ Permissions Policy adicionado"
```

### Comando 4: Aplicar Configurações

```bash
sudo systemctl restart nginx && echo "✓ Todos os headers de segurança aplicados"
```

## Blindagem PHP

Configura o php.ini para desativar funções perigosas e otimizar segurança.

> **IMPORTANTE:** Se usar Laravel Horizon/Queue ou Symfony Process, use a versão alternativa do Comando 1.

### Comando 1: Desativar Funções Perigosas (Versão Segura)

```bash
INI_FILE=$(php --ini | grep "Loaded Configuration" | sed -e "s|.*:\s*||"); sudo sed -i 's/disable_functions = .*/disable_functions = exec,passthru,shell_exec,system,proc_open,popen,curl_exec,curl_multi_exec,parse_ini_file,show_source,pcntl_exec,pcntl_fork,pcntl_signal,pcntl_waitpid,pcntl_wexitstatus,pcntl_wifexited,pcntl_wifsignaled,pcntl_wifstopped,pcntl_wstopsig,pcntl_wtermsig,pcntl_sigprocmask,pcntl_sigwaitinfo,pcntl_sigtimedwait,pcntl_get_last_error,pcntl_strerror,pcntl_async_signals,dl,symlink,link,pfsockopen,phpinfo/' "$INI_FILE" && echo "✓ Funções perigosas desativadas"
```

### Comando 1 (Alternativa): Para Laravel/Frameworks com Workers

Use este ao invés do anterior se precisar de filas/workers em background.

```bash
INI_FILE=$(php --ini | grep "Loaded Configuration" | sed -e "s|.*:\s*||"); sudo sed -i 's/disable_functions = .*/disable_functions = exec,passthru,shell_exec,system,proc_open,popen,curl_exec,curl_multi_exec,parse_ini_file,show_source,dl,symlink,link,pfsockopen,phpinfo/' "$INI_FILE" && echo "✓ Funções perigosas desativadas (pcntl mantido)"
```

### Comando 2: Ativar OPcache

```bash
INI_FILE=$(php --ini | grep "Loaded Configuration" | sed -e "s|.*:\s*||"); sudo sed -i 's/;opcache.enable=1/opcache.enable=1/' "$INI_FILE" && echo "✓ OPcache ativado"
```

### Comando 3: Ocultar Versão do PHP

```bash
INI_FILE=$(php --ini | grep "Loaded Configuration" | sed -e "s|.*:\s*||"); sudo sed -i 's/expose_php = On/expose_php = Off/' "$INI_FILE" && echo "✓ Versão do PHP ocultada"
```

### Comando 4: Aumentar Limites de Upload

```bash
INI_FILE=$(php --ini | grep "Loaded Configuration" | sed -e "s|.*:\s*||"); sudo sed -i 's/upload_max_filesize = .*/upload_max_filesize = 64M/' "$INI_FILE" && sudo sed -i 's/post_max_size = .*/post_max_size = 64M/' "$INI_FILE" && echo "✓ Limites de upload aumentados para 64M"
```

### Comando 5: Aplicar Configurações

```bash
sudo systemctl restart php*-fpm && echo "✓ PHP blindado e reiniciado"
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
