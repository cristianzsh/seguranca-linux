# 🗄 Segurança de servidores FTP

A segurança de servidores FTP é um aspecto crítico na proteção de dados e informações sensíveis. Devido à natureza inerente do protocolo FTP, que transmite dados em texto puro e não oferece criptografia. Muitos servidores FTP atuais, como o vsFTPD, oferecem a opção de utilizar uma chave para criptografar as informações.

Para instalar o vsFTPD, utilize o comando: `yum install vsftpd -y`. Seu principal arquivo de configuração é o `/etc/vsftpd/vsftpd.conf`.

Lembre-se de realizar a liberação da porta do FTP no _firewall_: `firewall-cmd --add-service=ftp --permanent --zone=public`.

### Configurações importantes

* **anonymous\_enable=NO**: desabilita o acesso via usuário anônimo.
* **anon\_upload\_enable=NO**: desabilita o upload de arquivos pelo usuário anônimo.
* **anon\_root=/var/ftp**: diretório padrão do usuário anônimo.
* **anon\_mkdir\_write\_enable=NO**: desabilita a criação de diretórios pelo usuário anônimo.
* **local\_enable=YES**: habilita o login de usuários locais.
* **write\_enable=YES**: habilita o modo de escrita para usuários locais.
* **chroot\_local\_user=YES**: impede que usuários saiam dos seus diretórios.
* **allow\_writeable\_chroot=YES**: habilita a gravação no diretório do usuário.

### Habilitando a criptografia

O primeiro passo é gerar uma nova chave para o serviço. Por características do vsFTPD, a chave não pode ser protegida por senha:

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:4096 -keyout vsftpd.pem -out vsftpd.pem
openssl req -utf8 -new -key vsftpd.pem -x509 -days 365 -out vsftpd.crt
```

No arquivo `vsftpd.conf`, adicione as seguintes configurações:

```
rsa_cert_file=/etc/pki/tls/certs/ftp/vsftpd.crt
rsa_private_key_file=/etc/pki/tls/certs/ftp/vsftpd.pem
ssl_enable=YES
```

Reinicie o serviço com o comando `systemctl restart vsftpd`. Note que conexões sem criptografia não são mais permitidas:

```bash
$ ftp 10.10.10.237
Connected to 10.10.10.237.
220 (vsFTPd 3.0.2)
Name (10.10.10.237:centos): cristian
530 Non-anonymous sessions must use encryption.
ftp: Login failed
```

Instale o comando `ftp-ssl` e o utilize para se conectar ao servidor:

```bash
$ ftp-ssl 10.10.10.237
Connected to 10.10.10.237.
220 (vsFTPd 3.0.2)
Name (10.10.10.237:centos): cristian
234 Proceed with negotiation.
[SSL Cipher DHE-RSA-AES256-SHA]
200 PBSZ set to 0.
200 PROT now Private.
[Encrypted data transfer.]
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
```

### Conclusão

Este capítulo se dedicou a aspectos de segurança do FTP, especialmente boas práticas de configuração e criptografia das comunicações. Vale notar que a adoção de políticas de senhas robustas e a restrição cuidadosa do acesso a usuários autorizados contribuem para fortalecer a segurança dos servidores FTP.
