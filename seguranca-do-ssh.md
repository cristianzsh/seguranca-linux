# 🐚 Segurança do SSH

O serviço SSH é um dos mais visados em ataques de força bruta, visto que ele permite o acesso remoto ao servidor. Portanto, seu _hardening_ é essencial. Algumas práticas simples, como a autenticação via chaves, já podem ser bastante efetivas. Entretanto, o arquivo `/etc/sshd_config` permite o uso de diversos outros parâmetros.

### Configurações gerais

* **Port 22**: permite alterar a porta padrão do serviço SSH.
* **Protocol 2**: força o uso da versão mais segura do protocolo.
* **PermitRootLogin no**: desabilita o acesso direto via usuário root.
* **MaxAuthTries 3**: número máximo de tentativas de autenticação para uma determinada sessão.
* **LoginGraceTime 20**: tempo que o usuário tem para concluir todo o procedimento de login.
* **PasswordAuthentication no**: desabilita o login via senhas.
* **PermitEmptyPasswords no**: impede tentativas de login com senhas vazias.
* **X11Forwarding no**: desativa o encaminhamento X11 (para execução de aplicações gráficas).
* **PermitUserEnvironment no**: desativa a passagem de variáveis de ambiente.
* **Banner /etc/ssh/custom\_banner**: altera a mensagem padrão ao acessar o sistema.

Uma boa prática desativar protocolos de autenticação não utilizados:

```
ChallengeResponseAuthentication no
KerberosAuthentication no
GSSAPIAuthentication no
```

Por fim, também podemos desabilitar opções de tunelamento:

```
AllowAgentForwarding no
AllowTcpForwarding no
PermitTunnel no
```

### Restrição de acesso por IP

É possível restringir o acesso ao servidor apenas a endereços ou redes autorizadas:

```
AllowUsers *@10.10.10.217
AllowUsers *@192.168.0.0/24
```

### Autenticação via chaves

O parâmetro `PubkeyAuthentication yes` habilita a autenticação via chaves. Lembre-se de adicionar sua chave pública no arquivo `~/.ssh/authorized_keys` para conseguir acessar o servidor.

### Exemplo de configuração

A seguir está um exemplo de configuração do arquivo `/etc/sshd_config` de acordo com as melhores práticas:

```
Protocol 2
Port 2222

LoginGraceTime 10
PermitRootLogin no
MaxAuthTries 3
IgnoreRhosts yes
PubkeyAuthentication yes
PasswordAuthentication no
PermitEmptyPasswords no

UsePAM yes
ChallengeResponseAuthentication no
KerberosAuthentication no
GSSAPIAuthentication no

AllowAgentForwarding no
AllowTcpForwarding no
X11Forwarding no
PrintMotd no
PrintLastLog yes
PermitUserEnvironment no
ClientAliveInterval 300
ClientAliveCountMax 2
PermitTunnel no

Banner /etc/ssh/custom_banner
AllowUsers *@10.10.10.217
```

### Conclusão

Este capítulo explorou boas práticas de configuração do SSH. Por ser um dos serviços mais visados por atacantes, sua proteção é fundamental para evitar acessos indevidos. Sem dúvidas uma excelente abordagem é combinar o uso de chaves com endereços IP autorizados.
