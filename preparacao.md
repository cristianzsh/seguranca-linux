# ⚙ Preparação

### Configuração do ambiente

Para um melhor proveito do conteúdo aqui apresentado, é recomendado que o leitor prepare um ambiente (virtualizado ou não) para colocar em prática os conhecimentos adquiridos. A maior parte das configurações será realizada em um sistema [CentOS](https://www.centos.org/download/), mas o mesmo pode ser realizado em outras distribuições com algumas pequenas adaptações.

No caso de um ambiente virtualizado, uma boa prática é sempre tirar _snapshots_ do sistema após alterações substanciais. Dessa forma, você poderá reverter facilmente o estado da máquina em caso de problemas.

### Erros comuns

Cometer erros é algo bastante natural, principalmente em arquivos de configuração, que possuem diferentes características de sintaxe e palavras-chave. Caso você obtenha erros ao configurar algum serviço, adote os seguintes passos:

1. Verifique a sintaxe do arquivo de configuração (ausência de chaves, ponto e vírgula, etc.);
2. Verifique as configurações do _firewall_ do sistema;
3. Analise os _logs_ das aplicações em busca de erros (`systemctl status <nome_serviço>`, `cat /var/log/<nome_serviço>/<arquivo_de_log>`, `journalctl -xe`);
4. Utilize o Google para buscar informações a respeito do erro.
