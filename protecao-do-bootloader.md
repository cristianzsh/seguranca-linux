# 💻 Proteção do bootloader

Um _bootloader_ desprotegido pode permitir que pessoas mal-intencionadas alterem parâmetros de iniciação do Kernel. Por exemplo, as seguintes alterações no GRUB nos dão acesso a um _shell_ como usuário root:

<img src=".gitbook/assets/image (3).png" alt="" data-size="original">

### Configurando uma senha para o GRUB

É possível configurar uma credencial para que seja possível acessar as opções do GRUB no momento do _boot_:

```bash
grub2-setpassword
grub2-mkconfig -o /boot/grub2/grub.cfg
```

Para remover a senha, basta apagar o arquivo `/boot/grub2/user.cfg`.

### Conclusão

A proteção do _bootloader_ é fundamental para evitar que usuários maliciosos consigam alterar parâmetros de iniciação e, consequentemente, ganhem acesso a recursos não autorizados. Portanto, sempre utilize credenciais de acesso para minimizar os riscos.
