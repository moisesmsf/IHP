# Manual de Busca de Evidências em Arquivos de Log no Linux

## 1. Principais Arquivos de Log para Busca de Evidências de Intrusão

Para investigação de intrusão em sistemas Linux, os seguintes arquivos de log são essenciais:

**a) Autenticação e Acesso**

- `/var/log/auth.log` (Debian/Ubuntu): Registra tentativas de login, sudo e atividades de autenticação.
- `/var/log/secure` (RedHat/CentOS): Equivalente ao `auth.log`.

**b) Logs do Sistema**

- `/var/log/syslog`: Registra eventos gerais do sistema.
- `/var/log/messages`: Contém mensagens do sistema, principalmente em sistemas RedHat/CentOS.

**c) Logs de Acessos Remotos**

- `/var/log/lastlog`: Histórico de últimos logins.
- `/var/log/wtmp`: Registra logins e logouts.
- `/var/log/btmp`: Registra tentativas de login falhas.

**d) Logs de Auditoria**

- `/var/log/audit/audit.log`: Registra eventos auditáveis do sistema (SELinux, acessos, comandos).

**e) Logs de Serviços Específicos**

- `/var/log/nginx/access.log` e `/var/log/nginx/error.log`: Logs do servidor web Nginx.
- `/var/log/httpd/access_log` e `/var/log/httpd/error_log`: Logs do Apache.
- `/var/log/cron`: Registra execução de tarefas agendadas.

**f) Logs de Firewall**

- `/var/log/ufw.log`: Logs do firewall UFW.
- `/var/log/firewalld`: Logs do firewall firewalld (RedHat/CentOS).

---

## 2. Exemplos de Comandos para Busca de Evidências

A seguir, comandos que podem ser utilizados para busca de indícios de intrusão:

**a) Verificação de Tentativas de Login**

```sh
cat /var/log/auth.log | grep 'Failed password'
awk '/Failed password/ {print $1, $2, $3, $9, $11}' /var/log/auth.log
```

**b) Busca por Logins Bem-Sucedidos**

```sh
grep 'Accepted password' /var/log/auth.log
awk '/Accepted password/ {print $1, $2, $3, $9, $11}' /var/log/auth.log
```

**c) Identificação de Acessos Remotos Suspiciosos**

```sh
grep 'sshd.*session opened' /var/log/auth.log | awk '{print $1, $2, $3, $9, $11}'
```

**d) Detecção de Múltiplas Tentativas de Login**

```sh
grep 'Failed password' /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -nr | head
```

**e) Análise de Erros em Serviços Web**

```sh
grep -i 'error' /var/log/nginx/error.log
awk '/404/ {print $1, $4, $7, $9}' /var/log/nginx/access.log
```

**f) Verificação de Acessos ao Cron**

```sh
grep CRON /var/log/syslog | awk '{print $1, $2, $3, $6, $9}'
```

**g) Análise de Tentativas de Escalonamento de Privilégio**

```sh
grep 'sudo' /var/log/auth.log | grep 'authentication failure'
```

**h) Verificação de Mudanças de Configuração**

```sh
grep -E 'modprobe|iptables|ufw' /var/log/syslog
```

**i) Busca por Alteração de Contas de Usuários**

```sh
grep 'useradd\|passwd\|usermod' /var/log/auth.log
```

**j) Acompanhamento em Tempo Real**

```sh
tail -f /var/log/auth.log
```

**k) Busca do Histórico de Comandos Executados por Usuários Específicos**

```sh
grep 'comando' /home/usuario/.bash_history
cat /home/usuario/.bash_history | grep 'sudo'
```

---

## 3. Dicas de Uso e Exemplos Práticos

**a) Lime (Dump da Memória RAM)**

```sh
insmod lime.ko "path=/caminho/dump.mem format=raw"
```

Verifique se o arquivo foi criado corretamente com:

```sh
ls -lh /caminho/dump.mem
```

**Ferramentas Alternativas ao Lime**

- **AVML (Azure Volatile Memory Lib)**: Pode ser executado diretamente de uma unidade USB sem necessidade de instalação.

  ```sh
  ./avml /mnt/usb/dump.mem
  ```

- **Fmem**: Módulo do kernel que permite acesso à memória como dispositivo de bloco.

  ```sh
  modprobe fmem
  dd if=/dev/fmem of=/mnt/usb/memdump.raw
  ```

- **LiMEaide**: Uma interface simples para capturar dumps de memória usando Lime.

  ```sh
  ./limeaide -o /mnt/usb/dump.mem
  ```

**b) Busca de Conexões de Rede Ativas**

```sh
netstat -antp
ss -tunap
lsof -i
```

**c) Descoberta de Usuários Ativos**

```sh
who
w
users
last
```

**d) Processos em Execução**

```sh
ps aux
top
htop
pstree
```

---

Este manual fornece um guia básico para a busca de evidências em sistemas Linux, utilizando ferramentas nativas para análise forense. Recomendamos sempre trabalhar com cópias dos logs para preservar as evidências originais.

