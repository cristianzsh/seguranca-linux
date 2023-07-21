# 🐞 Host intrusion detection

A prevenção e detecção de intrusões é essencial para proteger sistemas e dados contra ataques cibernéticos. Normalmente utilizamos ferramentas para monitoramento de tráfego, análise de _logs_, verificação de integridade de arquivos e detecção de atividades suspeitas em tempo real.

Algumas ferramentas:

* PortSentry
* Fail2ban
* OSSEC
* Snort

### PortSentry

O PortSentry é uma aplicação capaz de barrar _scans_ e tentativas de burlar sua segurança. Uma vantagem é a sua simplicidade, pois ela simula portas abertas no seu servidor. Quando as portas recebem alguma requisição, a origem é bloqueada. Utilize o comando `apt install portsentry -y` para o instalar.

No arquivo `/etc/portsentry/portsentry.conf`, habilite as opções `BLOCK_TCP` e `BLOCK_UDP`.

Para ser capaz de detectar _scans_ do tipo SYN, configure o modo `atcp` no arquivo `/etc/default/portsentry`:

```bash
sed -i 's/TCP_MODE="tcp"/TCP_MODE="atcp"/' /etc/default/portsentry
sed -i 's/UDP_MODE="udp"/UDP_MODE="audp"/' /etc/default/portsentry
```

### Fail2ban

O Fail2ban é um software para prevenção de intrusões. Ele foi escrito em Python e inicialmente projetado para prevenir ataques de força bruta. Com base na análise de _logs_, é possível tomar ações como bloquear o atacante. Conceitos importantes:

* **Filters**: definido com expressões regulares que correspondem aos padrões de tentativas de invasões em logs.
* **Actions**: define comandos a serem executados (e.g., bloquear um IP).
* **Jails**: combina filtros com as ações.

Para o instalar, utilize os seguintes comandos:

```bash
yum install epel-release
yum install fail2ban fail2ban-systemd
```

Principais arquivos e diretórios:

* **/etc/fail2ban/jail.conf**: arquivo com as configurações dos serviços a serem monitorados.
* **/etc/fail2ban/action.d/**: diretório onde ficam as regras a serem tomadas.
* **/etc/fail2ban/filter.d/**: diretório com expressões e padrões de detecção.
* **/etc/fail2ban/jail.local**: contém as suas configurações e regras para tomada de ações.

Exemplo de configuração do `/etc/fail2ban/jail.local`:

```
# Bloqueio brute-force no SSH.
[ssh]
enabled = true
filter = sshd
protocol = all
action = iptables[actname=iptables-input, name=brutessh, port="0:65535"]
         iptables[actname=iptables-forward, name=brutessh, chain=FORWARD, port="0:65535"]
logpath = /var/log/secure
maxretry = 3
bantime = 1440m
ignoreip = 127.0.0.1

# Bloqueio brute-force no FTP.
[vsftpd]
enabled = true
action = iptables[actname=iptables-input, name=brutevsftpd, port="0:65535"]
         iptables[actname=iptables-forward, name=brutevsftpd, chain=FORWARD, port="0:65535"]
logpath = /var/log/secure
port = ftp,ftp-data,ftps,ftps-data
logpath = %(vsftpd_log)s
maxretry = 5
bantime = 1440m
```

No exemplo, serão bloqueadas máquinas que fizerem 3 tentativas de login sem sucesso no serviço SSH ou 5 tentativas inválidas no serviço FTP. É possível customizar parâmetros como o tempo de bloqueio e as portas que serão bloqueadas, além de ser possível criar uma _whitelist_ para IPs autorizados.

Você pode verificar os endereços bloqueados por meio dos comandos:

```bash
fail2ban-client status ssh
fail2ban-client banned
```

E remover um determinado IP por meio do comando:

```bash
fail2ban-client set ssh unbanip 10.10.10.221
```

### arpwatch

O arpwatch é uma ferramenta capaz de alertar o administrador sobre alterações na tabela ARP. Seu principal arquivo de configuração fica localizado em `/etc/sysconfig/arpwatch` e seus logs são armazenados em `/var/log/messages`. Os endereços registrados podem ser consultados em `/var/lib/arpwatch/arp.dat`.

```bash
yum install arpwatch
chkconfig --level 35 arpwatch on
arpwatch -i ens33
```

Teste a detecção com o comando `arpspoof -i eth0 -r <roteador> -t <alvo>` executado de outra máquina.

### AIDE

O Advanced Intrusion Detection Environment (AIDE) é um utilitário que cria um banco de dados dos arquivos presentes no sistema para garantir a integridade deles e detectar possibilidades de intrusão. Seu principal arquivo de configuração é o `/etc/aide.conf`, que especifica os diretórios que serão monitorados.

```bash
yum install aide -y
aide --init
```

Para verificar o sistema, utilize o comando `aide --check`. Caso tenha feito alterações intencionais nos artefatos monitorados, utilize o `aide --update`.

Caso tenha problemas com o comando de verificação, crie um _link_ simbólico para o banco de dados do AIDE: `ln -s /var/lib/aide/aide.db.new.gz /var/lib/aide/aide.db.gz`.

### Linux Malware Detect (LMD)

Uma das soluções para identificação de artefatos maliciosos em sistemas Linux. O LMD é capaz de colocar _malwares_ em quarentena ou os remover do sistema. Utiliza _hashes_ MD5 para identificação de artefatos maliciosos e bases de dados como a do ClamAV. Ademais, possui sua própria base de assinaturas.

Instalação:

```bash
wget https://www.rfxn.com/downloads/maldetect-current.tar.gz
tar -xvf maldetect-current.tar.gz
cd maldetect-current
./install.sh
```

Para atualizar a base de dados, utilize o comando `maldet -u`, já para realizar um _scan_ de todo o sistema, utilize o comando `maldet -a`.

Alguns comandos úteis:

```bash
maldet -b -r /home/cristian/Downloads/
maldet --report list
maldet --report 230612-1233.7356
maldet --report 230612-1256.4430 root@localhost
```

### ClamAV

O ClamAV é um antivírus gratuito e _open-source_. É possível o utilizar para realizar _scans_ no sistema, verificação de anexos de e-mails, entre outras opções. Além disso, ele possui uma interface gráfica (ClamTk) e utilitários via linha de comando.

Instalação:

```bash
yum install clamav clamav-daemon -y
```

Para realizar um _scan_ em um diretório, utilize o comando `clamscan ./diretorio`.

### Chkrootkit

O Chkrootkit é um programa escrito em _shell script_ e capaz de verificar se um sistema Unix está infectado com _rootkits_ conhecidos.

Instalação e utilização:

```bash
git clone https://github.com/Magentron/chkrootkit.git
cd chkrootkit
./chkrootkit
```

### Rkhunter

Similarmente ao Chkrootkit, o Rkhunter também é capaz de detectar _rootkits_.

Instalação e utilização:

```bash
wget http://downloads.sourceforge.net/project/rkhunter/rkhunter/1.4.6/rkhunter-1.4.6.tar.gz
tar -xvf rkhunter-1.4.6.tar.gz
./installer.sh --install
rkhunter --check
```

### Lynis

O Lynis é uma ferramenta gratuita para auditoria de sistemas Unix. Ela pode ser utilizada para realização de testes automatizados de _compliance_ e detecção de vulnerabilidades.

Utilização:

```bash
git clone https://github.com/CISOfy/lynis
cd lynis && ./lynis audit system
```

### AuditD

O AuditD é um sistema de auditoria para sistemas Linux. Ele permite identificar violações de segurança por meio de regras pré-configuradas no arquivo `/etc/audit/auditd.conf`. Seus principais utilitários são os comandos `auditctl` e `ausearch`.

Instalação:

```bash
yum install audit
```

Exemplo de sintaxe:

```bash
auditctl -w /etc/passwd -p wa -k user-modify
```

Essa regra irá registrar alterações no arquivo `/etc/passwd`. Você pode testá-la realizando uma alteração no arquivo e consultando os _logs_ de auditoria:

```bash
cat /var/log/audit/audit.log | grep user-modify
ausearch -i -k user-modify
```

Vale notar que as configurações são feitas em memória. Para manter a persistência delas, adicione apenas a parte da regra ao arquivo `/etc/audit/rules.d/audit.rules`:

```
-w /etc/passwd -p wa -k user-modify
```

### Conclusão

Este capítulo abordou ferramentas para prevenção e detecção de intrusões, cada uma com suas respectivas funcionalidades para melhorar a segurança do sistema. Tais ferramentas podem ser utilizadas para monitorar o tráfego, analisar _logs_, verificar a integridade de arquivos e identificar atividades suspeitas em tempo real, contribuindo para reforçar a proteção do ambiente contra ameaças cibernéticas.
