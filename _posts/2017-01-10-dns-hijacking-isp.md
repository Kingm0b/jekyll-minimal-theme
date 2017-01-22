---
layout: post
title: "Hijacking de DNS dos provedores"
---

> Os provedores estão nos espionando?

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

OS MESMOS ENDEREÇOS !!!

SERÁ QUE ESTAMOS SOFRENDO ATAQUES DE **HIJACKING DNS** PELOS NOSSOS PRÓPRIOS PROVEDORES ?

Seria um acordo entre os provedores e o Governo Federal para nos espionarem?

Bem, durante um tempo esta foi a minha desconfiança.

Se, supostamente, o tráfego direcionado ao servidor DNS do Google estiver sendo desviado para servidores de terceiros, poderíamos perceber isto com um traceroute:

```
$ traceroute -n 8.8.8.8
traceroute to 8.8.8.8 (8.8.8.8), 30 hops max, 60 byte packets
 1  192.168.1.1  0.934 ms  1.440 ms  1.972 ms
 2  * * *
 3  100.120.13.21  36.217 ms 201.10.10.1  37.645 ms 201.10.197.17  39.122 ms
 4  201.10.55.44  78.926 ms 100.120.10.24  64.262 ms  73.329 ms
 5  177.2.192.123  83.846 ms 100.120.18.77  71.902 ms 100.120.16.227  88.664 ms
 6  201.10.241.11  80.810 ms 201.10.241.22  80.394 ms 100.120.16.140  79.136 ms
 7  72.14.194.186  82.543 ms  50.945 ms  51.418 ms
 8  216.239.51.230  67.190 ms  48.339 ms  48.456 ms
 9  209.85.244.79  101.354 ms 216.239.43.47  98.401 ms  100.740 ms
10  72.14.238.145  94.082 ms 72.14.234.179  100.433 ms 216.239.62.198  105.168 ms
11  * * *
12  * * *
13  * * *
14  * * *
15  * * *
16  * * *
17  * * *
18  * * *
19  8.8.8.8  124.079 ms  125.366 ms  126.731 ms
```

Um simples traceroute nos mostrou que o tráfego entre a nossa máquina e o 8.8.8.8 saiu dos domínios da Oi para a infra da Google (no *hop* 7, a partir de **72.14.194.186**). Isto é bom.

Mas nada impede que o tráfego supostamente desviado só afete datagramas UDP com porta de destino 53. Então para isto, vamos fazer um traceroute que satisfaça essa condição hipotética:

```
$ sudo hping3 -n --traceroute --udp -p 53 8.8.8.8
HPING 8.8.8.8 (enp2s0 8.8.8.8): udp mode set, 28 headers + 0 data bytes
hop=1 TTL 0 during transit from ip=192.168.1.1
hop=1 hoprtt=3.1 ms
hop=2 TTL 0 during transit from ip=100.120.13.21
hop=2 hoprtt=0.0 ms
hop=3 TTL 0 during transit from ip=100.120.64.139
hop=3 hoprtt=26.3 ms
hop=4 TTL 0 during transit from ip=201.10.55.44
hop=4 hoprtt=66.2 ms
hop=5 TTL 0 during transit from ip=177.2.192.123
hop=5 hoprtt=56.2 ms
hop=6 TTL 0 during transit from ip=100.120.17.118
hop=6 hoprtt=49.4 ms
hop=7 TTL 0 during transit from ip=191.223.6.1
hop=7 hoprtt=0.0 ms
hop=8 TTL 0 during transit from ip=216.239.51.228
hop=8 hoprtt=52.7 ms
hop=9 TTL 0 during transit from ip=191.223.6.1
hop=9 hoprtt=0.0 ms
hop=10 TTL 0 during transit from ip=209.85.241.184
hop=10 hoprtt=109.3 ms
hop=11 TTL 0 during transit from ip=191.223.6.1
hop=11 hoprtt=0.0 ms
len=237 ip=8.8.8.8 ttl=46 id=42906 seq=0 rtt=0.0 ms

--- 8.8.8.8 hping statistic ---
392 packets tramitted, 19 packets received, 96% packet loss
round-trip min/avg/max = 3.1/51.9/109.3 ms
```

Mesmo assim, vimos que o tráfego ainda está saindo dos domínios do provedor.

Humm... talvez o critério do hijacking deles seja desviar a rota somente de datagramas **UDP** com porta de destino **53** e contendo uma query DNS para resolução algum nome de domínio da Google.

Certo, então vamos ver no que dá.

Vamos captura uma query dns para resolução do fqdn "www.google.com" e usar a query capturada como payload para nosso traceroute (loko, né?).

No shell, mandei um ping para induzir a glibc a fazer a consulta DNS:

```
$ ping -c1 www.google.com
```

Em paralelo, capturei a query DNS com o Wireshark. Copiei a query em "Offset Hex":

```
0000   0e 43 01 00 00 01 00 00 00 00 00 00 03 77 77 77
0010   06 67 6f 6f 67 6c 65 03 63 6f 6d 00 00 01 00 01
```

Em um editor de texto, deixei em uma linha só:

" 0e 43 01 00 00 01 00 00 00 00 00 00 03 77 77 77 06 67 6f 6f 67 6c 65 03 63 6f 6d 00 00 01 00 01"

E no shell, adicionei caracteres de escape para indicar os bytes como hexadecimais:

```
$ echo " 0e 43 01 00 00 01 00 00 00 00 00 00 03 77 77 77 06 67 6f 6f 67 6c 65 03 63 6f 6d 00 00 01 00 01" | sed 's; ;\\x;g'
\x0e\x43\x01\x00\x00\x01\x00\x00\x00\x00\x00\x00\x03\x77\x77\x77\x06\x67\x6f\x6f\x67\x6c\x65\x03\x63\x6f\x6d\x00\x00\x01\x00\x01
```

Reproveito a saída do sed para usar no comando printf:

```
$ printf "\x0e\x43\x01\x00\x00\x01\x00\x00\x00\x00\x00\x00\x03\x77\x77\x77\x06\x67\x6f\x6f\x67\x6c\x65\x03\x63\x6f\x6d\x00\x00\x01\x00\x01" > /tmp/dns_query
```

Pego a quantidade de bytes que foi escrita no arquivo (útil para o próximo passo):

```
$ wc -c /tmp/dns_query
32 /tmp/dns_query
```

E executamos a consulta:

```
$ sudo hping3 -n --traceroute --udp --destport 53 --file /tmp/dns_query --data 32 8.8.8.8
HPING 8.8.8.8 (enp2s0 8.8.8.8): udp mode set, 28 headers + 32 data bytes
[main] memlockall(): Operation not supported
Warning: can't disable memory paging!
hop=1 TTL 0 during transit from ip=192.168.1.1
hop=1 hoprtt=2.9 ms
hop=2 TTL 0 during transit from ip=191.223.6.1
hop=2 hoprtt=17959.4 ms
hop=3 TTL 0 during transit from ip=201.10.199.114
hop=3 hoprtt=28.0 ms
hop=4 TTL 0 during transit from ip=191.223.6.1
hop=4 hoprtt=17972.6 ms
hop=5 TTL 0 during transit from ip=100.120.16.235
hop=5 hoprtt=50.5 ms
hop=6 TTL 0 during transit from ip=191.223.6.1
hop=6 hoprtt=17972.6 ms
hop=7 TTL 0 during transit from ip=72.14.194.186
hop=7 hoprtt=140.2 ms
hop=8 TTL 0 during transit from ip=191.223.6.1
hop=8 hoprtt=17969.2 ms
hop=9 TTL 0 during transit from ip=209.85.244.79
hop=9 hoprtt=96.8 ms
hop=10 TTL 0 during transit from ip=191.223.6.1
hop=10 hoprtt=17922.4 ms
len=237 ip=8.8.8.8 ttl=46 id=53283 seq=0 rtt=0.0 ms
len=189 ip=8.8.8.8 ttl=46 id=35057 seq=0 rtt=0.0 ms
len=237 ip=8.8.8.8 ttl=46 id=23909 seq=0 rtt=0.0 ms
^C
--- 8.8.8.8 hping statistic ---
43 packets tramitted, 24 packets received, 45% packet loss
round-trip min/avg/max = 2.9/9011.5/17972.6 ms
```

Se vocês sniffarem o tráfego, verão uma série de consultas DNS sendo lançadas!

Mesmo assim, observe no hop 7 que os pacotes alcançaram a infra da Google.


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
