# 🌐 Segurança no DNS

O Domain Name System (DNS) é um sistema hierárquico para resolução de nomes na Internet ou em uma rede privada. Os servidores DNS traduzem nomes de domínio em endereços IP, dessa forma você só precisa se lembrar de nomes, como `cristian.sh` ou `google.com`, para acessar um _host_.

### BIND

O BIND (Berkeley Internet Name Domain) é o servidor de DNS mais utilizado hodiernamente e considerado o padrão para sistemas Unix. Sua instalação pode ser realizada via `yum`:

```bash
yum install bind bind-utils -y
systemctl start named
```

Arquivos e diretórios importantes:

* **/etc/named.conf:** principal arquivo de configuração.
* **/var/named:** diretório que contém as zonas de DNS.
* **/var/named/data/named.run:** arquivo de _logs_.

Por padrão o BIND funciona em modo _caching-only_, realizando consultas e salvando os resultados em memória. Você pode testar o seu servidor com os seguintes comandos:

```bash
dig cristian.sh @127.0.0.1
host cristian.sh 127.0.0.1
```

### Configurações importantes

O arquivo `named.conf` possui algumas configurações fundamentais, entre elas:

* **listen-on:** indica a interface na qual o BIND aceitará consultas.
* **allow-query:** indica quem pode realizar consultas.
* **allow-transfer:** indica quais servidores (_slaves_) podem realizar transferências de zonas. Essa é uma configuração de segurança importante, visto que um atacante pode forçar uma transferência de zona para enumerar informações sobre o nosso domínio. Portanto, sempre indique servidores confiáveis nessa configuração.

**Exemplo de configuração:**

```bash
options {
	listen-on port 53 { 10.10.10.237; };
	listen-on-v6 port 53 { ::1; };
	directory 	"/var/named";
	dump-file 	"/var/named/data/cache_dump.db";
	statistics-file "/var/named/data/named_stats.txt";
	memstatistics-file "/var/named/data/named_mem_stats.txt";
	recursing-file  "/var/named/data/named.recursing";
	secroots-file   "/var/named/data/named.secroots";
	allow-query     { 10.10.10.0/24 ; 172.16.10.50 ; };
	allow-transfer  { 10.10.10.240 ; };
	recursion no;

	dnssec-enable yes;
	dnssec-validation yes;

	bindkeys-file "/etc/named.root.key";

	managed-keys-directory "/var/named/dynamic";

	pid-file "/run/named/named.pid";
	session-keyfile "/run/named/session.key";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "." IN {
	type hint;
	file "named.ca";
};

zone "site1.com.br" IN {
	type master;
	file "site1.com.br.zone";
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

Note como estão configuradas as opções citadas anteriormente. Nelas especificamos apenas dispositivos autorizados. Além disso, estamos definindo uma zona chamada `site1.com.br`, cujo arquivo de configuração deverá ser criado em `/var/named/site1.com.br.zone`.

Você pode utilizar o comando `named-checkconf` para verificar se a sua configuração está correta.

### Exemplo de zona

A seguir está um exemplo para a zona `site1.com.br`.

```bash
$TTL 3h

@	IN	SOA	site1.com.br.	admin.site1.com.br. (
		01	; serial
		28800	; refresh (8h)
		7200	; retry (2h)
		604800	; expire (7d)
		3600	; negative caching (1h)
)

		NS	site1.com.br.
@	IN	MX	5	mail
		MX	10	mail2

site1.com.br.	A	10.10.10.237
debian		A	10.10.10.240
mail		A	10.10.10.237
mail2		A	10.10.10.240
www		CNAME	site1.com.br.
ftp		A	10.10.10.237
```

Reinicie o BIND:

```bash
systemctl restart named
```

É importante notar que alterações nesse arquivo requerem um incremento no serial.

Você pode utilizar o comando `named-checkzone site1.com.br /var/named/site1.com.br.zone` para testar a sua configuração.

### Testando a resolução de nomes

Você pode testar a resolução de nomes por meio dos comandos `dig`, `host` ou `nslookup`. Uma prática interessante é adicionar o seu servidor no arquivo `/etc/resolv.conf` para facilitar os testes (`nameserver 10.10.10.237`).

```bash
[cristian@centos ~]$ nslookup 
> site1.com.br
Server:		10.10.10.237
Address:	10.10.10.237#53

Name:	site1.com.br
Address: 10.10.10.237
> set type=mx
> site1.com.br
Server:		10.10.10.237
Address:	10.10.10.237#53

site1.com.br	mail exchanger = 10 mail2.site1.com.br.
site1.com.br	mail exchanger = 5 mail.site1.com.br.
> set type=cname
> www.site1.com.br
Server:		10.10.10.237
Address:	10.10.10.237#53

www.site1.com.br	canonical name = site1.com.br.
```

### DNSSEC

O DNSSEC (Domain Name System Security Extensions) torna o processo de resolução de nomes mais seguro, reduzindo o risco de manipulação de dados e domínios forjados.

Para habilitar o recurso de DNSSEC, primeiro precisamos gerar uma chave:

```bash
dnssec-keygen -a DSA -b 1024 -r /dev/urandom -n ZONE site1.com.br
```

* **-a:** especifica o algoritmo de criptografia a ser utilizado.
* **-b:** especifica o tamanho da chave a ser gerada.
* **-r /dev/urandom:** define a entropia utilizada para gerar as chaves. No caso, utilizaremos números aleatórios definidos pelo `/dev/urandom`.
* **-n ZONE:** indica que iremos gerar uma assinatura de zona.

O arquivo **.key** gerado deve ser inserido nas configurações da sua zona.

```
site1.com.br. IN DNSKEY 256 3 3 CNy1yDcDJh+LBVUe3784sdhrypZZwcV3PWPtz0KWn5GBJI9KlL3TKOO/ c1FHZAD07gNmbXcCk/ld8AOjt6qydp8VkS9oEWqPLRgDP2f+rwwae6tA NUoQ7t4dgE7jqBfbfioBMNpR8dFS8ycmRvMblQNjD0gySBhyEaCDZPAq dAmSyZThIgRdI/Hjr6tcqpiqF01B+VGnjR9mdPZLG0VKqKPvizL79E89 Day1seilOgJNY0Xf2IS5UQWGcXP2kArEaXIlg2dvRJgs+8fPhN7jsQye KIyo6Y7la07UZt/mydloIcSDtRQqf+RreEBcUQb/hxvOKNLJLTiSRbqn lR+Z0Jqv2J/TB6wsk7OxShPR3L1SENdXqlvyBCQJtvXkM2JF0frxy6GM ccS7LoNNr4H3erpCm0Xo4F2bECRcV1E/u44PUKknzUbeG9h5iiIWHdLn +hzDHOQQma1qVkCH2CFOFYICsbsFEjY4N0ysA+a73AEubvZpnblnmRuP cDmuFDLS5FCwInFnzYLJ2abf7WtTrtuYRLSi
```

Após isso, utilize o comando `dnssec-signzone` para assinar a zona com a chave privada. Em cenários reais, as informações do arquivo **dsset** deverão ser inseridas no seu registrar.

```bash
dnssec-signzone -P -r /dev/urandom -o site1.com.br /var/named/site1.com.br.zone Ksite1.com.br.+003+44097.private
```

O resultado do comando será um arquivo com a extensão `.signed` no diretório `/var/named`. Você deverá alterar o seu `named.conf` para o BIND utilize esse novo arquivo. Teste a configuração com o comando `dig`:

```bash
dig @10.10.10.237 site1.com.br DNSKEY
```

### Enjaulamento do servidor de DNS

Enjaular o servidor de DNS é uma boa prática, pois evita que um atacante tenha acesso ao sistema de arquivos real do Linux no caso da exploração de alguma vulnerabilidade. Para que isso seja feito no BIND, é necessário instalar o pacote `bind-chroot`:

```bash
yum install bind-chroot -y
```

Agora precisamos desativar o BIND atual e ativar o processo `named-chroot`:

```bash
systemctl stop named
systemctl disable named

/usr/libexec/setup-named-chroot.sh /var/named/chroot/ on

systemctl start named-chroot
systemctl enable named-chroot
```

### Conclusão

Neste capítulo, exploramos importantes aspectos de segurança no servidor de DNS BIND, com foco em boas práticas de configuração, a implementação do DNSSEC e enjaulamento. Ao abordar esses tópicos, buscamos fortalecer a segurança da infraestrutura DNS, tornando-a mais resistente a ameaças e protegendo a integridade e autenticidade das informações transmitidas.
