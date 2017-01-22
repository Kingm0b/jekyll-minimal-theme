---
layout: post
title: "Hijacking de DNS dos provedores"
---

> OS PROVEDORES ESTÃO NOS ESPIONANDO! 

Se você é cliente da Oi, Vivo/GVT/Telefônica, NET, G8, ALGAR ou Claro, faça o teste:

Consulte o servidor DNS do seu provedor para a resolução de nomes do Google:

```
$ dig a www.google.com +short
201.34.205.101
201.34.205.99
201.34.205.113
201.34.205.88
201.34.205.112
201.34.205.123
201.34.205.110
201.34.205.91
201.34.205.121
201.34.205.117
201.34.205.90
201.34.205.102
201.34.205.80
201.34.205.84
201.34.205.95
201.34.205.106
```

*(Ou simplesmente mande um ping para obter o IP de www.google.com.br)*

Até aí tudo bem! Agora, vamos fazer uma resolução reversa de algum destes endereços IP do "Google":

```
$ dig -x 201.34.205.117 +short
201-34-205-117.pvoce300.ipd.brasiltelecom.net.br.
```

Hã?! Ok, vamos ver a geolocalização deste endereço:

```
$ geoiplookup 201.34.205.117
GeoIP Country Edition: BR, Brazil
```

**Brazil** ? Um traceroute:

```
$ sudo traceroute -In 201.34.205.110
traceroute to 201.34.205.110 (201.34.205.110), 30 hops max, 60 byte packets
 1  192.168.1.1  1.052 ms  1.653 ms  2.239 ms
 2  * * *
 3  100.120.13.21  33.792 ms  37.883 ms  38.198 ms
 4  201.10.55.44  44.327 ms  45.902 ms  48.391 ms
 5  100.120.66.1  51.991 ms  59.347 ms 177.2.248.2  57.412 ms
 6  100.120.66.109  57.720 ms  58.690 ms  60.922 ms
 7  201.34.205.110  62.337 ms  35.071 ms  30.429 ms
```

Humm... Confirmamos que nosso tráfego não está saindo dos limites do Brasil!

Vamos ver o que a base whois da AS tem para nos dizer:

```
$ whois 201.34.205.117

inetnum:     201.34.0.0/16
aut-num:     AS8167
abuse-c:     CSIOI
owner:       Brasil Telecom S/A - Filial Distrito Federal
ownerid:     76.535.764/0326-90
responsible: Brasil Telecom S. A. - CNBRT
country:     BR
owner-c:     BTC14
tech-c:      BTC14
inetrev:     201.34.205.0/24
nserver:     ns03-cta.brasiltelecom.net.br
nsstat:      20170122 AA
nslastaa:    20170122
nserver:     ns04-bsa.brasiltelecom.net.br
nsstat:      20170122 AA
nslastaa:    20170122
created:     20050511
changed:     20050511

nic-hdl-br:  BTC14
person:      Brasil Telecom S. A. - CNRS
e-mail:      ld-numeracaoip@oi.net.br
country:     BR
created:     20031003
changed:     20170106

nic-hdl-br:  CSIOI
person:      CSIRT OI
e-mail:      abuse@oi.net.br
country:     BR
created:     20140127
changed:     20140127
```

Como assim? A Oi está direcionando o acesso a serviços do Google para servidores dela?

Vamos fazer mais um teste. Vamos consultar diretamente o servidor DNS da Google e ver o que ele tem para nos dizer:

```
$ dig a www.google.com @8.8.8.8 +short
201.34.205.110
201.34.205.95
201.34.205.123
201.34.205.90
201.34.205.113
201.34.205.88
201.34.205.80
201.34.205.102
201.34.205.99
201.34.205.84
201.34.205.101
201.34.205.106
201.34.205.91
201.34.205.112
201.34.205.117
201.34.205.121
```

OS MESMOS ENDEREÇOS !!! SERÁ QUE ESTAMOS SOFRENDO ATAQUES DE HIJACKING DNS PELOS NOSSOS PRÓPRIOS PROVEDORES ? Seria um acordo entre os provedores e o Governo Federal para nos espionarem?

Bem, durante um tempo esta foi a minha desconfiança.

Baseado nesta notícia:

http://www.theregister.co.uk/2016/05/27/blue_coat_ca_certs/

A minha suspeita inicial era que a BlueCoat utilizava sua AC para assinar os certificados dos seus clientes sem comprovação de posse de domínio.

Se realmente ela faz isto, estará agindo contra as recomendações de organizações como WebTrust e CAB Forum:

    CAB Forum (tópico 3.2.2.4):
    https://cabforum.org/wp-content/uploads/CA-Browser-Forum-BR-1.4.1.pdf

    WebTrust (página 57):
    http://www.webtrust.org/homepage-documents/item54279.pdf

Boa parte das aplicações (o que inclui os navegadores) consideram ACs alheias como confiável por padrão quando elas são homologadas por estas organizações.

O estranho é que as próprias documentações do ProxySG orientam a criação de uma AC auto-assinada:

https://bto.bluecoat.com/webguides/proxysg/security_first_steps/Content/Solutions/SSL/ssl_solution.htm

Então, descartado esta possibilidade, uma situação que possa ocorrer é o ISP fazer parcerias com grandes provedores de conteúdo (como a Google).

A Google tem um programa chamado "Google Global Cache":

    https://isp.google.com/iwantggc/

Com este programa, o ISP terceiriza alguns de seus servidores para que ela os utilize como CDN. Boa parte dos maiores ISPs estão fazendo uso desta solução. Exemplo da Brasil Telecom (agora Oi):

    https://189.73.192.227/

Como é a Google quem gerencia este servidor (e também tem posse dos domínios *.google.com, por exemplo), ela consegue facilmente emitir certificados digitais para seus CDNs.

Ou seja, mesmo neste cenário, nem o ISP tem acesso ao conteúdo do tráfego TLS.
