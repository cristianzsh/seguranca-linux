# 🔒 Parâmetros de segurança do Kernel

O Kernel Linux permite a customização de diversos parâmetros em tempo de execução. Isso possibilita a ativação de recursos de segurança para efetivamente mitigar ataques e reforçar a proteção do sistema.

A configuração de parâmetros do Kernel em tempo de execução pode ser feita por meio do comando `sysctl` ou inserindo tais parâmetros no arquivo `/etc/sysctl.conf`. As subseções a seguir exploram alguns dos principais parâmetros.

### Bloqueio de ICMP

A prática de bloquear tráfego ICMP direcionado a um _host_ é útil para minimizar a exposição de informações e evitar ataques DoS:

```bash
sysctl net.ipv4.icmp_echo_ignore_all=1
sysctl net.ipv4.icmp_echo_ignore_broadcasts=1

echo "net.ipv4.icmp_echo_ignore_all=1" >> /etc/sysctl.conf
echo "net.ipv4.icmp_echo_ignore_broadcasts=1" >> /etc/sysctl.conf
```

* **icmp\_echo\_ignore\_all**: faz o Kernel ignorar completamente pacotes ICMP Echo Request recebidos. Isso ajuda a proteger o sistema contra ataques de negação de serviço.
* **icmp\_echo\_ignore\_broadcasts**: ignora pacotes ICMP Echo Request direcionados a endereços de _broadcast_.

### Alteração do TTL padrão

O valor do TTL (Time to Live) padrão pode ser utilizado para identificar o tipo de sistema operacional em uso (sendo 64 o valor para sistemas Linux e 128 para sistemas Windows). Logo, outra prática interessante é alterar esse valor para confundir o atacante:

```bash
sysctl net.ipv4.ip_default_ttl=73

echo "net.ipv4.ip_default_ttl=73" >> /etc/sysctl.conf
```

* **ip\_default\_ttl**: define por quantos roteadores o pacote poderá passar antes de ser eliminado.

### Address Space Layout Randomization (ASLR)

Address Space Layout Randomization (ASLR) é uma técnica de segurança utilizada para proteger sistemas operacionais contra ataques que exploram vulnerabilidades de memória. Seu principal objetivo é tornar mais difícil para os atacantes prever a localização de funções, dados e estruturas na memória do sistema, dificultando assim a execução bem-sucedida de ataques baseados em endereços fixos.

```bash
sysctl kernel.randomize_va_space=2

echo "kernel.randomize_va_space=2" >> /etc/sysctl.conf
```

* **randomize\_va\_space**: ativa o ASLR. Logo, as áreas de memória para os processos são alocadas de forma aleatória no espaço de endereçamento virtual.

### Proteção contra IP spoofing

A proteção contra IP _spoofing_ é uma medida de segurança implementada para prevenir ataques de _spoofing_, que é uma prática em que um atacante falsifica o endereço IP de origem em um pacote de rede para mascarar sua identidade ou enganar sistemas de segurança e filtragem.

```bash
sysctl net.ipv4.conf.all.rp_filter=1
sysctl net.ipv4.conf.all.log_martians=1
sysctl net.ipv4.conf.all.accept_source_route=0

echo "net.ipv4.conf.all.rp_filter=1" >> /etc/sysctl.conf
echo "net.ipv4.conf.all.log_martians=1" >> /etc/sysctl.conf
echo "net.ipv4.conf.all.accept_source_route=0" >> /etc/sysctl.conf
```

* **rp\_filter**: utilizado para proteção contra ataques de IP _spoofing_.
* **log\_martians**: permite que o Kernel registre pacotes com endereços IP inválidos ou suspeitos nos _logs_ do sistema.
* **accept\_source\_route**: controla se o Kernel aceita pacotes com rotas de origem diferentes das rotas padrão do sistema.

### Proteção contra SYN Flood

O SYN Flood é um tipo de ataque de DoS que visa sobrecarregar um servidor ao enviar um grande número de solicitações de conexão TCP (SYN) falsas. O uso de _cookies_ consiste em responder a pacotes SYN com um número gerado aleatoriamente em vez de iniciar uma conexão completa. O servidor usa esses _cookies_ para rastrear as conexões pendentes e, quando um pacote ACK é recebido, a conexão é recriada usando as informações do _cookie_.

```bash
sysctl net.ipv4.tcp_syncookies=1

echo "net.ipv4.tcp_syncookies=1" >> /etc/sysctl.conf
```

* **tcp\_syncookies**: utilizado para proteger o sistema contra ataques de TCP SYN Flood. Quando ativado, o Kernel utiliza _cookies_ SYN para lidar com conexões TCP.

### Conclusão

Neste capítulo foram abordados parâmetros de segurança do Kernel Linux. A utilização do `sysctl` permite a aplicação de tais parâmetros em tempo de execução, tornando a configuração do Kernel bastante flexível.
