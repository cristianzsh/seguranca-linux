# 💽 Criptografia de disco

A criptografia de disco é essencial para a proteção das informações em armazenamento. Com ela podemos inviabilizar a leitura de dados de discos roubados ou em cenários onde uma pessoa realiza o _boot_ via live CD. Entre os diversos softwares utilizados para criptografia de disco, o LUKS (Linux Unified Key Setup) se encontra disponível em praticamente todas as distribuições Linux.

Outros programas:

* eCryptfs
* VeraCrypt
* dm-crypt

### Utilizando o LUKS

Adicione um disco de 10 GB ao seu sistema. Após isso, utilize o `fdisk` para criar uma partição para o novo disco. Os seguintes comandos devem ser utilizados para criptografar a nova partição:

```bash
cryptsetup luksFormat /dev/sdb1
cryptsetup luksOpen /dev/sdb1 encvol
mkfs.ext4 /dev/mapper/encvol
mkdir /secret
mount /dev/mapper/encvol /secret
```

Para desmontar a partição e fechar o LUKS, utilize os seguintes comandos:

```bash
umount /secret
cryptsetup luksClose /dev/mapper/encvol
```

### Persistência em partições com o LUKS

Para montar a partição automaticamente no _boot_ do sistema, gere uma chave e a vincule à partição:

```bash
dd if=/dev/urandom of=/root/lukskey bs=4096 count=1
chmod 600 /root/lukskey
cryptsetup luksAddKey /dev/sdb1 /root/lukskey
```

Após isso, configure o arquivo `/etc/crypttab`:

```
encvol /dev/sdb1 /root/lukskey luks
```

E o `/etc/fstab`:

```
/dev/mapper/encvol      /secret                 ext4    defaults        0 0
```

### Utilizando o GPG

Arquivos sensíveis podem ser protegidos por meio do GPG (GNU Privacy Guard). É possível criptografar um arquivo utilizando o seguinte comando:

```bash
gpg --symmetric --cipher-algo AES256 teste.txt
```

E o decriptar da seguinte maneira:

```bash
gpg -o teste.txt -d teste.txt.gpg
```

### Conclusão

A criptografia de disco e o uso de ferramentas como LUKS e GPG são medidas essenciais para garantir a confidencialidade e integridade dos dados armazenados, impedindo o acesso não autorizado e protegendo informações sensíveis contra ameaças externas.
