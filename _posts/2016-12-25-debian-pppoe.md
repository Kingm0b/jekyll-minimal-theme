---
layout: post
title: "Discagem PPPoE no Debian 8"
---

> O PPPoE é ainda o protocolo mais utilizado pelos provedores (A\|V)DSL. Aqui, documento algumas experiências básicas que vivenciei com este tipo de tecnologia.

De posse de um Opticom DSLink, desabilitei o Wifi junto com vários outros serviços habilitados (UPnP, DHCP, Telnet e FTP). Em seguida, re-configurei as definições de conexão com a WAN, configurando o roteadorzinho SOHO a trabalhar como **bridge**.

A minha missão era configurar um roteador/firewall rodando Debian 8 para "discar" e receber um endereço público do provedor.

Graças aos desenvolvedores do daemon **pppd**, os caras do "Roaring Penguin" com seu **rp-pppoe.so** e os empacotadores do Debian, conexões PPPoE no Linux pode ser bastante simples ("pode" ser simples se as suas necessidades forem simples, dê [uma olhadinha na man page do pppd](https://ppp.samba.org/pppd.html) ) se você quiser ver o que é flexibilidade.

Então, em um *quick how-to* segue os procedimentos para configuração do PPPoE no Debian:

1 - Instale o PPPoE:

```bash
# apt-get update
# apt-get install ppp
```

Observação: se o link ADSL que você está configurando for o seu link principal, instale o utilitário pppoeconf:

```bash
# apt-get install pppoeconf
```

Este utilitário automatiza todo o processo de criação dos arquivos de configuração do pppd. Caso seu roteador Linux já esteja em produção e o link ADSL que você está configurando **não** for o seu link default: CUIDADO! Se você afobar e confirmar todas as mensagens sem ler antes, o pppoeconf irá substituir a rota default do seu roteador pela rota default do provedor ADSL (o que pode não ser uma boa).

