# Documentação para Apoio na Análise Forense em Ambientes Operacionais Linux

## Ações Forenses Preparatórias

As asções forenses preparatórias visam expor os cuidados necessários para que a coleta das evidências sejam realizadas com o menor impacto para o ambiente analisado, bem como garantir que possíveis artefatos não sejam perdidos durante a análise.

## 1. Garantir a Integridade das Evidências
Todas as evidências coletadas devem ser rigorosamente catalogadas minimamente com os seguintes dados:

- Nome e descrição da evidência;
- Data e hora da coleta;
- Local (físico e virtual) onde a evidência foi coletada;
- Identificação do analista que coletou as evidências;
- Qual informação útil para a investigação aquela evidência traz;
- Hash da evidência e/ou do conjunto de evidências;


## 2. Garantir o Estado do ambiente
Preferencialmente, o ambiente analisado deve ter sua integridade garantida, ou seja, efetuar o menor número de modificações possíveis. Se possível, não realizar nenhuma modificação nas evidências originais.

**a) Ambiente ligado**

- Isolar a máquina da rede.
- Criar uma imagem de memória RAM usando lime.
- Gerar hash dos logs e arquivos críticos.
- Documentar todas as ações realizadas.

**b) Ambiente desligado**

- Criar imagem forense do disco rígido com ferramentas como dd ou dc3dd.
- Verificar integridade da imagem gerada com hashes.
- Proteger a mídia original contra gravação.

**c) Máquinas físicas**

- Utilizar bloqueadores de escrita em discos.
- Utilizar discos secundários para criar uma imagem clone do disco original (conforme item b acima)
- Inspeção visual para identificar dispositivos suspeitos.
- Documentar componentes físicos como números de série.

**d) Máquinas virtuais**

- Exportar snapshots e armazenar cópias seguras.
- Clonar a máquina original para que as análises sejam feitas em um ambiente seguro.
- Capturar logs de hipervisores.
- Monitorar atividades de rede da máquina virtual.


## 3. Imagem do Disco
**a) Clonando um disco usando a ferramenta DD**
Exemplo de criação de imagem forense com dd, onde 'if=' é o disco a ser clonado e 'of=' é o destino da nova imagem:

```sh
dd if=/dev/sdX of=/caminho/para/imagem/disco.img bs=4M status=progress
```

Após a criação da imagem, é necessário registrar o hash da imagem recém criada para se garantir sua integridade:

```sh
$ md5sum disco.img
$ sha256sum disco.img
```

**b) Utilizando a ferramenta FTK Imager**
Exemplo de criação de Imagem Forense com FTK Imager:

- Abra o FTK Imager.
- Vá para File > Create Disk Image.
- Escolha o tipo de origem (Físico, Lógico, Arquivo).
- Selecione o disco ou partição a ser analisado.
- Escolha o formato de saída (E01, RAW, etc.) e configure a compressão.
- Especifique o local de destino e nome da imagem.
- Clique em Finish para iniciar a criação da imagem.

**Mais informações sobre o FTK Imager:**

- **Referência e dicas de uso:** https://academiadeforensedigital.com.br/ftk-imager-ferramenta-gratuita-de-forense-digital/

- **Download:** https://www.exterro.com/ftk-product-downloads/ftk-imager-4-7-3-81

- **Uso via CLI Linux:** https://www.cybrary.it/blog/using-ftk-imager-cli-challenging-new-disks-technologies


## 4. Forense ao vivo (Memória volátil)
**a) Dump da memória RAM**

Execute o lime com o comando abaixo para efetuar o dump da memória volátil:

```sh
insmod lime.ko "path=/caminho/dump.mem format=raw"
```

Verifique se o arquivo foi criado corretamente com:

```sh
ls -lh /caminho/dump.mem
```

**Referência e download do lime:**

- https://levelblue.com/blogs/security-essentials/memory-dump-analysis-using-lime-for-acquisition-and-volatility-for-initial-setup
- https://medium.com/@_bleedblack_1/exploring-live-ram-data-on-linux-using-lime-63f46b24c220

**Ferramentas alternativas ao lime:**

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

**b) Busca de conexões de rede ativas**
Execute os comandos abaixo para visualizar as conexões de rede ativas em um ambiente Linux:

```sh
netstat -antp
ss -tunap
lsof -i
```

**c) Usuários ativos no ambiente**
Execute os comandos abaixo para visualizar os usuários ativos e logados no ambiente naquele momento:

```sh
who
w
users
last
```

**d) Processos em execução no ambiente**
Execute os comandos abaixo para visualizar os processos em execução naquele momento:

```sh
ps aux
top
htop
pstree
```

O resultado dos comandos acima devem ser copiados para arquivos texto e extraídos para uma análise posterior. Abaixo segue um exemplo de como um comando pode ser exportado para um arquivo txt:

```sh
$ ss -tunap >> conexoes_rede_ativas.txt
$ ps aux >> processos_ativos.txt
```

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
/var/run/utmp
/var/log/btmp
/var/log/lastlog
last
umtpdump /var/log/wtmp
```

**b) Busca por Logins Bem-Sucedidos**

```sh
grep 'Accepted password' /var/log/auth.log
awk '/Accepted password/ {print $1, $2, $3, $9, $11}' /var/log/auth.log
```

**c) Identificação de Acessos Remotos Suspeitos**

```sh
grep 'sshd.*session opened' /var/log/auth.log | awk '{print $1, $2, $3, $9, $11}'
grep -E 'Failed password|session opened|Accepted password' /var/log/auth.log
journalctl _SYSTEMD_UNIT=ssh.service | egrep "Failed|Failure"
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
grep -i CRON /var/log/syslog | awk '{print $1, $2, $3, $6, $9}'
journalctl -u cron
```

**g) Análise de Tentativas de Escalonamento de Privilégio**

```sh
grep 'sudo' /var/log/auth.log | grep 'authentication failure'
grep -E 'useradd|passwd|groupadd|usermod|visudo|sudo' /var/log/auth.log
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

**l) Demais Arquivos de Interesse**

```sh
cat /etc/crontab
cat /etc/cron.d/*
cat /var/spool/cron/*
cat .bash_rc (non-login shells)
cat .bash_profile (login shells)
cat /proc/mounts
cat /etc/exports
cat /etc/hosts
cat /etc/resolv.conf
cat /etc/sudoers
find / -name 'authorized_keys'
grep -RPn "(passthru|shell_exec|system|phpinfo|base64_decode|chmod|mkdir|fopen|fclose|readfile|php_uname|eval|exif_read_data|preg_replace) *\(" /var/www
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

