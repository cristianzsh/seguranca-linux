# 🆔 Trabalhando com certificados

Certificados digitais funcionam como uma identidade digital para entidades (como sites, servidores e indivíduos). Eles garantem a autenticidade, integridade e criptografia das comunicações na Internet, permitindo que usuários confiem nas informações transmitidas.

### Tipos de certificados digitais

#### Certificados autoassinados

* São emitidos e assinados pela mesma entidade.
* São adequados para ambientes de desenvolvimento e teste.

#### Certificados de autoridade (CA)

* São emitidos por uma autoridade de certificação confiável.
* Utilizados para validar a identidade de terceiros.
* Ideais para ambientes de produção.

### OpenSSL

Implementação de código aberto dos protocolos SSL e TLS. A biblioteca permite gerar, assinar e gerenciar certificados digitais. Ademais, pode ser utilizada em várias linguagens de programação por meio de _wrappers_.

### Gerando uma chave privada

```bash
openssl genrsa -aes128 -out private_key.pem 2048
```

Este comando irá gerar uma chave privada utilizando o algoritmo AES com uma chave de 128 bits. A seguir estão as explicações dos parâmetros:

* **genrsa:** indica que queremos gerar uma chave RSA.
* **-aes128:** especifica que utilizaremos o AES com chave de 128 bits.
* **-out:** indica o nome do arquivo que conterá a chave privada.
* **2048:** especifica o tamanho da chave. Quanto maior o tamanho, maior a segurança. Entretanto, isso pode tornar as operações de criptografia mais lentas.

Ao final da execução, você deverá informar uma senha para proteção da chave.

### Gerando um certificado autoassinado

```bash
openssl req -utf8 -new -key private_key.pem -x509 -days 365 -out mycert.crt
```

O comando irá gerar um certificado autoassinado utilizando a chave previamente gerada.

* **req:** indica que faremos uma solicitação de certificado.
* **-utf8:** indica que o certificado deve ser gerado utilizando a codificação UTF-8.
* **-new:** especifica que queremos criar uma nova solicitação de certificado.
* **-key:** caminho da chave privada.
* **-x509:** especifica o formato do certificado. Esse parâmetro tipicamente indica que iremos gerar um certificado autoassinado.
* **-days:** validade do certificado em dias.

O OpenSSL solicitará algumas informações adicionais necessárias preencher os campos do certificado.

### Visualizando informações sobre um certificado

```bash
openssl x509 -in mycert.crt -text -noout
```

* **-in:** caminho para o arquivo que contém o certificado.
* **-text:** especifica que queremos visualizar o conteúdo do certificado em formato de texto.
* **-noout:** informa ao OpenSSL para não exibir informações adicionais além do conteúdo do certificado.

O comando exibirá informações detalhadas sobre o certificado, incluindo quem o emitiu e sua validade.

### Gerando uma Certificate Signing Request (CSR)

```bash
openssl req -new -key private_key.pem -out myreq.csr
```

O arquivo gerado deverá ser enviado a uma autoridade de certificação (CA) para geração do certificado digital. Esse é o cenário ideal para ambientes de produção (públicos na Internet).

### Proteção da chave privada

Independentemente de se utilizar um certificado autoassinado ou assinado por uma CA, é fundamental manter a chave privada em segurança, pois sua perda ou exfiltração pode resultar no comprometimento da segurança e falsificação de assinaturas.

### Conclusão

Este capítulo se dedicou a fornecer uma breve introdução aos certificados digitais e utilização do OpenSSL. O certificado e chave privada gerados serão utilizados nos capítulos posteriores para criptografia das comunicações de diferentes serviços. Portanto, execute os comandos e armazene os arquivos gerados em um local seguro.
