---
layout: post
title: "Interceptando tráfego HTTPS com o Squid"
---

> Vamos discutir a problemática do Squid-in-the-middle em tráfego TLS

O Squid (em suas versões atuais) consegue fazer a interceptação de tráfego HTTPS de duas maneiras diferentes: ativa e passiva (até a versão 3.4, somente **ativa**).

A interceptação passiva passou a ser possível a partir da versão **3.5** com a entrada de uma nova feature chamada "*SslBump Peek and Splice*".

Até a versão 3.4, a relação tráfego *HTTPS vs Proxy Transparente* era bem problemática. Com este novo recurso, basicamente, é possível usar o Squid como proxy transparente mesmo se o tráfego for HTTPS.

Quando configurado para trabalhar no modo "**splice**", o Squid reconstrói o tráfego desencapsulando o payload de aplicação do pacote IP que sofreu DNAT (contendo tráfego TLS) e o re-injetando como payload de uma mensagem HTTP do tipo CONNECT (o tráfego é tunelado temporariamente apenas em "tempo de execução").

Desta forma, passivamente, o Squid consegue aplicar ACLs como **dst** e **dstdomain** de forma transparente em tráfego TLS.

Mas o que realmente queremos é fazer man-in-the-middle em tráfego HTTPS, fazendo cache e aplicando ACLs como url_regex, sem precisar instalar cadeias de certificados em todas as máquinas clientes, certo?

Para o "bumping" de HTTPS, o Squid dispõe de duas maneiras diferentes de se lidar com certificados digitais:

* exigindo a criação de uma AC local (ou seja, certificado raiz contendo o campo CA:TRUE), ele re-assina os certificados dos servidores originais antes de repassar para os clientes (funcionalidade esta chamada de "Dynamic SSL Certificate Generation"); ou
* utilizando um único certificado digital, com o campo CA:FALSE.

Podemos descartar o primeiro cenário, pois esta exige a instalação de um certificado na máquina cliente.

Nos meus laboratórios, foquei no segundo caso. A princípio a solução seria solicitar um certificado á GlobalSign, utilizando como subdomínio wildcards de TLD (exemplo: *.br, *.com, *.net no campo subjAltName do certificado). Mas isso vai contra as definições da RFC 6125. Os navegadores simplesmente impedem o acesso a sites com wildcards inválidos.

A única solução seria a emissão de um certificado multidomínio contendo o endereço de todos os sites que vocês queiram monitorar (exemplo: *.google.com, *.google.com.br, *.facebook.com, e etc, todos no mesmo certificado).

Mas infelizmente (ou felizmente, do ponto de vista da segurança), nenhuma Autoridade Certificadora emite certificados multidomínio sem comprovação prévia da posse dos domínios especificados.

Pesquisei por várias soluções proprietárias de **HTTPS Proxy**:

* Fortigate
* SonicWall
* Checkpoint
* Cisco CWS
* SOPHOS
* WatchGuard

Todas instruem o administrador a gerar uma AC local!

