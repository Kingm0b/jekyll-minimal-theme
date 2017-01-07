---
layout: post
title: "\"Hijacking\" de DNS dos provedores"
---

> OS PROVEDORES ESTÃO NOS ESPIONANDO! Salvem-se quem puder!

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
