# 📖 Introdução

### O que é defender?

A defesa cibernética consiste em um conjunto de técnicas para proteger servidores, sistemas, redes e dispositivos contra ataques.

A proteção de sistemas é de fundamental importância para a segurança da informação. Nesse contexto, é fundamental adotar boas práticas de _hardening_ para tornar sistemas mais resilientes a ameaças cibernéticas, garantindo a confidencialidade, integridade e disponibilidade das informações. Com o aumento dos ataques direcionados a sistemas Linux, é essencial implementar medidas robustas de defesa para mitigar riscos e fortalecer a segurança desse sistema operacional.

Enquanto atacantes necessitam apenas idenficar uma vulnerabilidade para invadir um alvo, os agentes de defesa precisam idenficar todas as fragilidades e aplicar as contramedidas adequadas, exigindo um amplo conhecimento e habilidades em diversas subáreas da segurança. O _hardening_ do sistema Linux envolve, entre outras medidas, a adoção de boas configurações de segurança, desativação de serviços e componentes desnecessários, configuração de permissões de acesso de forma adequada e implementação medidas adicionais de proteção (_firewalls_, sistemas de IDS e IPS, entre outras).

Vale notar que o processo de tornar um sistema Linux mais seguro é constante, visto que novas ameaças e vulnerabilidades são publicadas diariamente. Logo, é necessário realizar revisões periódicas das configurações de segurança e ajustá-las conforme necessário. Ao implementar medidas de segurança adequadas, é possível melhorar a segurança do ambiente e dificultar a execução bem-sucedida de ataques cibernéticos, protegendo os dados e recursos críticos da organização contra ameaças.

### Segurança de servidores Linux

O Linux é o sistema operacional mais utilizado em servidores e dispositivos IoT. Portanto, sua proteção é fundamental para a segurança das informações armazenadas e processadas. A defesa de sistemas Linux pode ser dividida em:

* Proteção de serviços do sistema (e.g., HTTP, DNS, SMTP).
* Proteção do sistema operacional (e.g., desabilitar ICMP, habilitar o ASLR).

### Ataque x defesa

Como mencionado anteriormente, um _hacker_ precisa identificar apenas uma vulnerabilidade para invadir um sistema. Já um agente de defesa precisa identificar todas as fragilidades e aplicar as devidas contramedidas.

A área de defesa de sistemas está ligada ao time de _blue team_ de uma organização. A função desse time é identificar falhas de segurança, aplicar as correções necessárias e verificar se as medidas implantadas estão sendo efetivas na mitigação de ataques.

A área de ataque de uma empresa é responsabilidade do _red team_, onde seus integrantes realizam análises de vulnerabilidades e testes de intrusão (_pentests_) utilizando as mesmas técnicas que _hackers_ maliciosos. Os _pentests_ podem ser realizados em aplicações web, infraestruturas internas ou em aplicativos _mobile_.

### Áreas de atuação

Um profissional que atua na defesa de sistemas pode atuar em uma ou mais das seguintes áreas:

1. Engenheiro de segurança cibernética;
2. Consultor de cibersegurança;
3. Engenheiro de DevSecOps;
4. Forense digital;
5. Ethical hacking;
6. Entre outras.

### Boas práticas e frameworks

Podemos utilizar alguns _frameworks_ e boas práticas de mercado para nos auxiliar na definição das políticas de segurança que guiarão os processos de defesa. As subseções a seguir exploram alguns dos principais deles.

#### [Guia do hardening (NIC.br)](https://bcp.nic.br/i+seg/acoes/hardening/)

Consiste em um conjunto geral de práticas para proteger infraestruturas e minimizar riscos de segurança. O guia é bem objetivo e fácil de ler e entender, sendo dividido nas seguintes áreas:

* Autenticação.
* Autorização.
* Auditoria.
* Acesso.
* Registros.
* Sistema.
* Configurações.

#### [NIST Cyber Security Framework (CSF)](https://www.nist.gov/cyberframework)

O CSF foi desenvolvido para ajudar as organizações a definir quais atividades elas precisam realizar para atingir diferentes padrões de cibersegurança. Ele permite a comunicação entre equipes multidisciplinares utilizando uma linguagem simples e não necessariamente técnica. O core do framework consiste em três partes:

* Funções: Identificar, detectar, proteger, responder e recuperar.
* Categorias: Existem 23 categorias divididas nas cinco funções.
* Subcategorias: Existem 108 subcategorias divididas nas 23 categorias.

As cinco funções se aplicam ao gerenciamento de riscos em geral. As categorias abrangem a amplitude dos objetivos de segurança cibernética. As subcategorias são declarações orientadas a resultados que fornecem considerações para criar ou melhorar um programa de segurança. Para cada subcategoria, é fornecido um recurso informativo que faz referência a seções específicas de outros padrões de segurança da informação, incluindo:

* ISO 27001.
* COBIT.
* NIST SP 800-53.
* ANSI.
* ISA-62443.

#### [ISO 27001 e 27002](https://www.iso.org/standard/27001)

A ISO 27001 é uma norma internacional para gestão de segurança da informação. Seu principal objetivo é a adoção de um conjunto de requisitos, processos e controles para gerir adequadamente os riscos de segurança da informação presentes nas organizações. Normalmente os controles do Anexo A são implementados em conjunto com as práticas definidas pela ISO 27002.

#### [CIS Critical Security Controls](https://www.cisecurity.org/controls)

Os CIS Controls são um conjunto prescritivo, prioritizado e simplificado de melhores práticas que você pode utilizar para fortalecer sua postura de segurança cibernética. Hoje, milhares de profissionais de segurança cibernética ao redor do mundo utilizam os CIS Controls e/ou contribuem para o seu desenvolvimento por meio de um processo de consenso comunitário. A seguir é apresentada a lista de CIS Controls:

* CIS Control 1: Inventory and Control of Enterprise Assets
* CIS Control 2: Inventory and Control of Software Assets
* CIS Control 3: Data Protection
* CIS Control 4: Secure Configuration of Enterprise Assets and Software
* CIS Control 5: Account Management
* CIS Control 6: Access Control Management
* CIS Control 7: Continuous Vulnerability Management
* CIS Control 8: Audit Log Management
* CIS Control 9: Email and Web Browser Protections
* CIS Control 10: Malware Defenses
* CIS Control 11: Data Recovery
* CIS Control 12: Network Infrastructure Management
* CIS Control 13: Network Monitoring and Defense
* CIS Control 14: Security Awareness and Skills Training
* CIS Control 15: Service Provider Management
* CIS Control 16: Application Software Security
* CIS Control 17: Incident Response Management
* CIS Control 18: Penetration Testing
