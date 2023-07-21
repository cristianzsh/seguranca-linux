# 📄 Segurança de servidores HTTP

A segurança de servidores web é essencial para garantir a confidencialidade, integridade e disponibilidade dos recursos de uma organização disponibilizados online. Nesse contexto, diversas práticas podem ser empregadas para mitigar ameaças e evitar explorações de vulnerabilidades. Neste capítulo, iremos aplicar diversas práticas de segurança no servidor Apache. Entretanto, os conceitos também se aplicam a outros servidores (e.g., Nginx, IIS).

O Apache pode ser instalado por meio do comando `yum install httpd -y`. Seu principal arquivo de configuração é o `/etc/httpd/conf/httpd.conf`.

### Regras básicas

Basta "subir" um servidor web na Internet e você verá que é uma questão de minutos para que ele comece a ser atacado. Logo, algumas algumas regras básicas devem ser aplicadas para minimizar os riscos, entre elas:

* Atualizar regularmente o servidor e seus módulos.
* Sempre utilizar e forçar o uso do HTTPS.
* Utilizar cifras seguras.
* Proteger o servidor contra ataques de força bruta.
* Armazenar logs em um local seguro e com backups.
* Utilizar um Web Application Firewall (WAF).

### Apache + mod\_ssl

O `mod_ssl` é um módulo de criptografia para o Apache, sua instalação pode ser feita por meio do comando `yum install mod_ssl -y`. É importante notar que você precisará realizar liberações no _firewall_ do sistema para acessar os websites:

```bash
firewall-cmd --add-service=http --permanent
firewall-cmd --add-service=https --permanent
firewall-cmd --reload
```

**Exemplo de configuração de um VirtualHost no Apache:**

```
<VirtualHost 10.10.10.237:443>
	ServerName www.site1.com.br
	DocumentRoot /var/www/html/site1
	SSLEngine on
	SSLCertificateFile /etc/ssl/certs/site1.com.br/mycert.crt
	SSLCertificateKeyFile /etc/ssl/certs/site1.com.br/private.pem
</VirtualHost>
```

### Autenticação em diretórios

Muitas vezes precisamos restringir o acesso a determinados diretórios de um servidor web e uma das formas é via autenticação por usuário e senha. A seguinte configuração pode ser utilizada:

```
<Directory "/var/www/html/site1/confidencial">
	AuthType Basic
	AuthName "Acesso restrito"
	AuthUserFile /var/www/passwd/.htpasswd
	Require valid-user
</Directory>
```

Essa configuração deve ser inserida dentro do bloco `VirtualHost` criado anteriormente.

Utilize o comando `htpasswd -c /var/www/passwd/.htpasswd <usuario>` para gerar o arquivo com as credenciais. Ao acessar o diretório `/confidencial`, será exigido um usuário e senha válidos.

### Autenticação em diretórios por IP

Uma outra prática interessante é permitir o acesso a determinados recursos apenas a endereços IP autorizados (e.g., colaboradores que estão fisicamente no escritório ou via VPN corporativa). Para isso, pode-se utilizar a seguinte configuração:

```
<Directory "/var/www/html/site1/restrito">
	Require ip 10.10.10.240
	Require ip 172.16
	Require ip 192.168.0.0/24
</Directory>
```

Essa configuração irá permitir o acesso apenas para a máquina de IP `10.10.10.240`, qualquer máquina cujo IP comece por `172.16` ou qualquer máquina da rede `192.168.0.0/24`.

### Mutual SSL

Este recurso permite autenticarmos também o cliente antes do acesso ao site. Dessa forma, apenas usuários com um certificado válido podem acessar determinadas áreas do site ou até mesmo o site como um todo.

O primeiro passo é criarmos uma CA interna:

```bash
openssl genrsa -des3 2048 > /etc/pki/CA/private/cakey.pem
openssl req -utf8 -new -key /etc/pki/CA/private/cakey.pem -x509 -days 365 -out /etc/pki/CA/cacert.pem -set_serial 0
touch /etc/pki/CA/index.txt
echo 1000 > /etc/pki/CA/serial
openssl ca
```

O arquivo `cacert.pem` deve ser especificado no Apache. Ele será utilizado para validação do certificado do cliente.

```
SSLCACertificateFile /etc/pki/CA/cacert.pem
<Directory "/var/www/html/site1/mutual">
	SSLVerifyClient require
</Directory>
```

Após isso, gere um certificado e o assine

```bash
openssl genrsa -des3 -out user.key 2048
openssl req -new -key user.key -out user.csr
mv user.csr newreq.pem
/etc/pki/tls/misc/CA -signreq
openssl pkcs12 -export -inkey user.key -in newcert.pem -out user.p12
```

O arquivo `user.p12` deve ser enviado ao cliente e instalado em seu navegador. Dessa forma, apenas usuários com certificados válidos podem acessar essa área do website.

### Forçando o uso de versões seguras do TLS

Se executarmos um software como o `sslscan`, é possível notar que o website utiliza versões inseguras do SSL e defasadas do TLS:

```
$ sslscan site1.com.br
Version: 2.0.16-static
OpenSSL 1.1.1u-dev  xx XXX xxxx

Connected to 10.10.10.237

Testing SSL server site1.com.br on port 443 using SNI name site1.com.br

SSL/TLS Protocols:
SSLv2     disabled
SSLv3     enabled
TLSv1.0   enabled
TLSv1.1   enabled
TLSv1.2   enabled
TLSv1.3   disabled
```

Uma boa prática é utilizar versões do TLS acima da 1.1, pois elas são mais adequadas aos padrões computacionais modernos. Além disso, também podemos forçar o uso de cifras seguras. A seguinte configuração (que deve ser inserida dentro do `VirtualHost`) pode ser utilizada para tais finalidades:

```
SSLProtocol -all +TLSv1.2
SSLCipherSuite HIGH:!aNULL:!MD5
SSLHonorCipherOrder on
```

### Forçando o uso do HTTPS

Embora o site esteja com boas práticas de segurança aplicadas, ainda é possível acessá-lo via HTTP puro, o que não é recomendado, visto que informações podem ser visualizadas em texto puro por um atacante na rede. Portanto, uma boa prática é sempre forçar o redirecionamento das requisições para o HTTPS. Essa configuração pode ser feita por meio da criação de um outro `VirtualHost` responsável pelo encaminhamento das solicitações:

```
<VirtualHost 10.10.10.237:80>
	ServerName www.site1.com.br
	Redirect / https://www.site1.com.br
</VirtualHost>
```

Usuários que tentarem acessar o site via HTTP receberão um código de _status_ 302 e serão redirecionados para o HTTPS.

### Ocultando informações

Outra boa prática é minimizar a exposição de informações nos cabeçalhos do HTTP, visto que atacantes podem as utilizar para buscar vulnerabilidades públicas para a versão do seu servidor. A seguinte configuração oculta esses dados:

```
ServerTokens Prod
ServerSignature Off
SecServerSignature IIS
TraceEnable Off
```

Note a diferença:

```bash
[cristian@centos ~]$ curl -I https://site1.com.br -k
HTTP/1.1 200 OK
Date: Tue, 18 Jul 2023 00:01:42 GMT
Server: Apache/2.4.6 (CentOS) OpenSSL/1.0.2k-fips
Last-Modified: Mon, 17 Jul 2023 20:10:09 GMT
ETag: "d-600b462a5d30b"
Accept-Ranges: bytes
Content-Length: 13
Content-Type: text/html; charset=UTF-8
```

```bash
[cristian@centos ~]$ curl -I https://site1.com.br -k
HTTP/1.1 200 OK
Date: Tue, 18 Jul 2023 00:02:41 GMT
Server: IIS
Last-Modified: Mon, 17 Jul 2023 20:10:09 GMT
ETag: "d-600b462a5d30b"
Accept-Ranges: bytes
Content-Length: 13
Content-Type: text/html; charset=UTF-8
```

### ModSecurity

O ModSecurity é um Web Application Firewall (WAF), ou seja, um software que trabalha entre o servidor HTTP e o cliente. Sua função é filtrar entradas do cliente e saídas do servidor web, permitindo identificar, registrar e bloquear ataques.

WAFs também são conhecidos como _firewalls_ de camada 7. O ModSecurity foi originalmente projetado como um módulo do Apache e pode ser integrado com as regras da OWASP. Sua instalação pode ser feita por meio do comando `yum install mod_security mod_security_crs –y`.

Após a instalação, basta reiniciar o serviço `httpd` para que o _firewall_ passe a atuar. A seguir está um exemplo do que ocorre ao tentar executar um ataque:

```bash
[cristian@centos ~]$ curl https://site1.com.br/news.php?id=1%27%20or%201=1 -k
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>403 Forbidden</title>
</head><body>
<h1>Forbidden</h1>
<p>You don't have permission to access /news.php
on this server.</p>
</body></html>
```

Também é possível visualizar os _logs_ gerados pela ferramenta em `/var/log/httpd/modsec_audit.log`.

### Conclusão

Este capítulo explorou boas práticas de segurança em servidores web. Inicialmente configuramos o uso do TLS, abordamos a autenticação em diretórios por meio de credenciais e via endereço IP, além da implementação do _mutual_ SSL. Por fim, forçamos a utilização de protocolos seguros para as comunicações. Ao seguir as práticas de segurança discutidas neste capítulo, os administradores de servidores web podem fortalecer a segurança de seus sistemas, protegendo os dados do site, restringindo o acesso a informações confidenciais e garantindo a autenticidade das conexões entre o servidor e os clientes.
