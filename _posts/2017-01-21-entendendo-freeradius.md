---
layout: post
title: "Entendendo o FreeRADIUS"
---

> Aqui, discuto os princípios do procotolo RADIUS e a configuração do FreeRADIUS

# Introdução

O primeiro contato que tive com o FreeRADIUS foi assustador. A instalação foi simples, um único "apt-get install freeradius" foi o suficiente. O baque veio no momento de verificar o que foi instalado no meu sistema: <code> $ dpkg -L freeradius</code>

**Dezenas de bibliotecas compartilhadas, misturadas a dezenas de arquivos de configuração!**

Para piorar, dentro de cada arquivo de configuração existem centenas de diretivas junto com uma tonelada de comentários. Cheguei até a criar um alias, chamado "sem comentários" rs, para tentar extrair o que não era comentário daqueles arquivos:

```
$ alias semcmt="egrep -v '(^#)|(^$)|(.*#.*)'"
$ semcmt eap.conf
```

Meu objetivo era "apenas" configurar o FreeRADIUS para autenticar clientes (ou seriam "suplicantes"?) de uma rede Wifi.

Na Web, encontrei vários how-to's, tanto em português quanto em inglês, sobre como configurar aquilo que eu pretendia fazer. Mas eu me sentia completamente perdido em ter que seguir uma série de procedimentos, copiando e colando coisas, sem ter ideia do que significava aquilo.

Foi aí que decidi começar do começo.


# O serviço RADIUS

É comum, em diversas fontes, o pessoal se referir ao RADIUS como "um servidor de autenticação". Essa definição pode estar correta, mas, de certa forma, é meio ambígua. Seguindo esta definição, um SGDB como o MySQL ou uma base LDAP também podem ser considerados "servidores de autenticação", afinal, as credenciais do usuário estarão armazenadas lá, e a autenticação ocorrerá mediante a consulta a estas credenciais para comparação com as fornecidas pelo cliente.

Da mesma forma que o Apache é um servidor Web, e o BIND um servidor DNS, o FreeRADIUS é um servidor de **AAA** - *não confundir com o registro AAAA do DNS* -, sigla para "**A**uthentication **A**uthorization and **A**ccounting".

Existem outros servidores de AAA além do FreeRADIUS, proprietários como o NPS da Microsoft, o servidor RADIUS da Cisco e algumas soluções open-source alternativas ao FreeRADIUS, como o GNU RADIUS.

Assim como páginas Web podem ser requisitadas via HTTP, HTTPS, SPDY ou HTTP/2, o protocolo RADIUS não é o único utilizado pelos serviços de AAA. Existe um outro, porém pouco difundido, chamado DIAMETER.

R.A.D.I.U.S é a sigla para "Remote Authentication Dial In User Service". Numa tradução literal ficaria algo como "Serviço de Autenticação Remota para Discagem do Usuário". A parte do "Discagem do Usuário" lembra algo relacionado a Internet Discada, né?

Na verdade o RADIUS inicialmente foi desenvolvido em 1991 para este propósito: facilitar a autenticação de clientes de um ISP via PPP, centralizando o processo de autenticação e autorização em um servidor à parte.

Apesar da "discagem" praticamente não existir mais, até hoje o RADIUS é utilizado pelos provedores, só que nas conexões de banda larga. Onde o login e senha do cliente é enviado para o provedor via **IPCP** sobre alguma variação do PPP (PPPoE ou PPPoA).

Nas redes de telefonia móvel, como a GSM, o RADIUS também está presente nos ERBs das operadoras para a autenticação dos dispositivos por meio de informações presentes no SIM-CARD deles.

Nas redes internas de algumas empresas, o RADIUS é muito utilizado como alternativa a autenticação em Access Points por chave compartilhada.

Com relação ao FreeRADIUS, me acalmei ao saber duas coisas sobre todos aqueles arquivos de configuração e diretivas:

* Aquilo são apenas modelos, você não precisa utilizar todos os arquivos de configuração; e
* Somente uma quantidade pequena de diretivas realmente serão utilizadas;
