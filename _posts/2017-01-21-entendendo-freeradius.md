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


# O protocolo RADIUS

Assim como a grande maioria dos outros protocolos de aplicação, o RADIUS é um protocolo cliente-servidor. O diferencial dele para os demais protocolos é que ele é um **protocolo binário**! Isso quer dizer que ao invés de mensagens contendo comandos e atributos legíveis a humanos, os tipos e atributos das mensagens deste protocolo são identificados por bytes específicos.

**Por exemplo**: mensagens RADIUS tendo como primeiro byte o identificador <code>0x01</code>, indica uma tentativa de autenticação (ou Access-Request como veremos mais a baixo). Caso o primeiro byte seja <code>0x03</code>, trata-se de uma mensagem de acesso rejeitado (ou Access-Reject). Já no corpo de uma mensagem, uma sequência de 2 bytes contendo <code>0x010a</code> é um indicador da presença do nome de usuário com 8 caracteres (provavelmente em um Access-Request).

Apesar da maioria dos protocolos chamarem o cliente de "cliente" e o servidor de "servidor", alguns protocolos diferentões tentam quebrar esse paradigma, como é o caso do SNMP com seu "gerente-agente". Acontece que o RADIUS também é diferentão! E seu cliente é chamado de **NAS**, *não confunda com aquela solução de storage*, a sigla aqui quer dizer: Network Access Server. Daqui pra frente, para qualquer referência a sigla NAS entenda como "cliente RADIUS".

Apesar de também poder rodar sobre TCP, o protocolo RADIUS é amplamente implementado sobre UDP, usando como porta de escuta a **1812** por parte do servidor.

## Arquitetura de uma rede RADIUS

Basicamente, o NAS requisita autenticação a um servidor RADIUS e o servidor RADIUS decide se aceita ou rejeita a tentativa de autenticação. Pronto!

Acontece que nem sempre o NAS é um simples cliente final. Na parte maior dos casos ele é um intermediário entre um dispositivo final e o servidor RADIUS. Como uma imagem vale mais que mil palavras, vejam a seguinte ilustração: 

![](https://raw.githubusercontent.com/m0blabs/m0blabs.github.io/master/images/2017-01-21/radius.fig336.epsi.gif)

Na perspectiva do dispositivo do usuário (um notebook, smartphone, tablet, roteador SOHO, e etc, por exemplo) ele não faz ideia da existência de um servidor RADIUS, o interesse dele é apenas ter a permissão concedida para usufruir da rede (ou outro recurso requisitado). Cabe ao equipamento responsável por liberar acesso ao cliente "tercerizar" a autenticação, repassando as credenciais fornecidas pelo cliente ao servidor RADIUS (utilizando aqui o protocolo RADIUS). O servidor RADIUS, por sua vez, irá aceitar ou rejeitar a autenticação, informando isto ao dispositivo intermediário, que por sua vez irá conceder ou negar acesso ao cliente da maneira como a autenticação foi solicitada (via PPP, 802.1x, HTTP, módulo PAM!!!, que seja).

Em redes muito grandes, onde o volume de "Access-Request's" são intensos, uma boa prática é utilizar vários servidores em round-robin para balanceamento de carga, ou uma hierarquia de servidores proxys RADIUS para aliviar o(s) servidor(es) principal(is). Veja a imagem: 

[](https://raw.githubusercontent.com/m0blabs/m0blabs.github.io/master/images/2017-01-21/IC195441.gif)

