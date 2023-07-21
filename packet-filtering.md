# 🗃 Packet filtering

Diversos softwares estão disponíveis para filtragem de pacotes. Soluções nativas, como o `iptables`, continuam sendo bastante utilizadas em ambientes domésticos e corporativos. O `iptables` funciona baseado no endereço/porta de origem/destino do pacote. Também pode ser utilizado para modificar e monitorar o tráfego da rede, fazer NAT, redirecionamento de pacotes, entre outras funções.

Existem três componentes principais no `iptables`: chains, tabelas e regras.

### Chains

Sequências de regras que determinam o que acontece com um pacote. Existem cinco chains predefinidas no `iptables`:

* **INPUT**: controla os pacotes destinados ao próprio sistema.
* **OUTPUT**: controla os pacotes gerados pelo próprio sistema.
* **FORWARD**: controla os pacotes que passam pelo sistema, roteados de uma interface de rede para outra.
* **PREROUTING**: controla os pacotes antes que eles sejam roteados.
* **POSTROUTING**: controla os pacotes após terem sido roteados.

### Tabelas

Estruturas que organizam as regras do `iptables`. Existem cinco tabelas principais:

* **filter**: tabela padrão, utilizada para filtragem básica de pacotes. Ela controla o tráfego com base em regras definidas nas chains INPUT, OUTPUT e FORWARD.
* **nat**: utilizada para alterar os endereços de origem ou destino dos pacotes.
* **mangle**: utilizada para realizar alterações especiais nos cabeçalhos dos pacotes (e.g., modificar o TTL).
* **raw**: utilizada para configurações especiais antes que as decisões de filtragem sejam aplicadas.
* **security**: utilizada para recursos de controle de acesso.

### Regras

São as instruções que especificam as ações a serem tomadas com base nas características dos pacotes. As regras definem o comportamento do _firewall_ e podem permitir, bloquear, redirecionar ou modificar pacotes com base em diferentes critérios. Critérios comuns:

* Endereço IP de origem ou destino.
* Porta de origem ou destino.
* Protocolo.
* Interface de rede.

### Comandos básicos

A seguir estão exemplos de configurações típicas que podem ser adaptados de acordo com a sua necessidade.

#### Listagem de regras

```bash
iptables -L -n -v # lista com detalhes as regras da tabela filter.
```

#### Remoção de regras

```bash
iptables -F # apaga todas as regras.
iptables -F INPUT # apaga todas as regras da chain INPUT.
iptables -L --line-numbers # lista as regras de forma enumerada.
iptables -D INPUT 7 # deleta a regra da linha 7.
```

#### Alteração das políticas das chains

```bash
iptables -P INPUT DROP # descarta todos os pacotes destinados ao host.
iptables -P FORWARD DROP # descarta todos os pacotes de encaminhamento.
iptables -P OUTPUT DROP # descarta todos os pacotes de saída.
```

#### Liberando tráfego SSH

```bash
# Liberação da porta 22.
iptables -A INPUT -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
```

#### Permitindo tráfego de uma rede específica

```bash
# Liberação de tráfego destinado às portas 80 e 443 de pacotes da rede 10.10.10.0/24.
iptables -A INPUT -p tcp -s 10.10.10.0/24 -m multiport --dports 80,443 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p tcp -m multiport --sports 80,443 -m state --state ESTABLISHED -j ACCEPT

```

#### Habilitando tráfego de saída

```bash
# Liberação de tráfego DNS.
iptables -A OUTPUT -p udp --dport 53 -j ACCEPT
iptables -A INPUT -p udp --sport 53 -j ACCEPT

# Liberação de tráfego HTTPS.
iptables -A OUTPUT -p tcp --dport 443 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -p tcp --sport 443 -m state --state ESTABLISHED -j ACCEPT
```

#### Port forwarding

```bash
# Tráfego destinado à porta 9090 será destinado à porta 22.
iptables -t nat -A PREROUTING -p tcp -d 10.10.10.216 --dport 9090 -j DNAT --to 10.10.10.216:22
iptables -A INPUT -p tcp --sport 9090 -m state --state ESTABLISHED -j ACCEPT
iptables -A OUTPUT -p tcp --dport 9090 -m state --state NEW,ESTABLISHED -j ACCEPT
```

#### IP routing

```bash
# Habilita o encaminhamento de pacotes.
echo 1 > /proc/sys/net/ipv4/ip_forward

# Encaminha pacotes entre as redes 192.168.211.0/24 e 10.10.10.0/24.
iptables -I FORWARD -s 192.168.211.0/24 -d 10.10.10.0/24 -j ACCEPT
iptables -I FORWARD -s 10.10.10.0/24 -d 192.168.211.0/24 -j ACCEPT

```

#### Salvando e restaurando regras

```bash
iptables-save > /etc/sysconfig/iptables
iptables-restore < /etc/sysconfig/iptables
```

### Conclusão

O `iptables` é uma poderosa ferramenta de _firewall_ que permite configurar regras precisas para controlar o fluxo de pacotes em uma rede, garantindo a segurança e o desempenho adequado do sistema. Neste capítulo, abordamos detalhadamente a utilização dele para a filtragem de pacotes em sistemas Linux. Você pode adaptar os comandos apresentados de acordo com as suas necessidades.
