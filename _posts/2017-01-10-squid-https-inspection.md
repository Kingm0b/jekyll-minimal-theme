---
layout: post
title: "Interceptando tráfego HTTPS com o Squid"
---

> Vamos discutir a problemática do Squid-in-the-middle em tráfego TLS

O TLS é um protocolo de segurança que garante o sigilo da comunicação entre clientes e servidores. Nem preciso citar alguma estatística da *Netcraft*, *SANS Institute* ou algo do tipo para dizer que atualmente uma porcentagem muito alta do tráfego de máquinas clientes fazem uso de algum protocolo de aplicação *over TLS* (como é o caso do **HTTPS**). Se você administra algum web proxy ou firewall, faça o teste: verifique quantas conexões HTTPS estão passando por seus equipamentos e compare com o número de conexões HTTP.

Movimentos como o [Encrypt All The Things](https://encryptallthethings.net/), Let's Encrypt, e as ações da Google (como melhor rankeamento para páginas em https e alerta do Chrome quando o site é em HTTP) estão colaborando para o aumento de páginas em HTTPS.

Diante deste cenário, administradores de rede estão ficando muito preocupados. Afinal, cada vez mais está ficando difícil  fazer cache, aplicar ACLs como **url_regex** e usar filtros de conteúdo como o Dansguardian. Vi casos de administradores que estão deixando de usar proxy por este motivo.

Para redes que necessitam do uso de Proxy Transparente o problema é maior ainda. Já que passivamente o Squid (até a versão 3.4) não conseguia trabalhar com HTTPS nesta modalidade.

O Squid (em suas versões atuais) consegue fazer a interceptação de tráfego HTTPS de duas maneiras diferentes: ativa e passiva (até a versão 3.4, somente **ativa**).

A interceptação passiva passou a ser possível a partir da versão **3.5** com a entrada de uma nova feature chamada "*SslBump Peek and Splice*".

Até a versão 3.4, a relação tráfego *HTTPS vs Proxy Transparente* era bem problemática. Com este novo recurso, basicamente, é possível usar o Squid como proxy transparente mesmo se o tráfego for HTTPS.

Quando configurado para trabalhar no modo "**splice**", o Squid reconstrói o tráfego desencapsulando o payload de aplicação do pacote IP que sofreu DNAT (contendo tráfego TLS) e o re-injetando como payload de uma mensagem HTTP do tipo CONNECT (o tráfego é tunelado temporariamente apenas em "tempo de execução").

Desta forma, passivamente, o Squid consegue aplicar ACLs como **dst** e **dstdomain** de forma transparente em tráfego TLS.

A versão 3.5 do Squid ainda não se encontra nos repositórios stable do Debian. Portanto, teremos que compilá-lo.

```bash
# apt-get install pkg-config gcc g++ make autoconf libtool build-essential
# wget http://www.squid-cache.org/Versions/v3/3.5/squid-3.5.20.tar.gz
# tar -zxvf squid-3.5.20.tar.gz
# cd squid-3.5.20
# ./configure \
--disable-maintainer-mode \
--disable-silent-rules \
--disable-snmp \
--disable-htcp \
--disable-wccp \
--disable-wccpv2 \
--disable-auth-basic \
--disable-auth-digest \
--disable-auth-negotiate \
--disable-auth-ntlm \
--disable-unlinkd \
--enable-ssl-crtd \
--enable-delay-pools \
--enable-icmp \
--enable-follow-x-forwarded-for \
--with-large-files \
--enable-build-info="M0blabs" \
--with-default-user=proxy \
--mandir=/usr/share/man \
--sysconfdir=/etc/squid3 \
--datadir=/usr/share/squid3 \
--prefix=/usr \
--with-openssl


# make all
# make install
# make install-pinger
# mkdir /var/log/squid3/
# chown -R proxy.proxy /var/log/squid3/
# /usr/libexec/ssl_crtd -c -s /var/log/squid3/ssl_db
```

Regras de DNAT

```
# iptables -t nat -A PREROUTING ! -d 192.168.1.121 -p tcp --dport 80 -j REDIRECT --to-ports 80
# iptables -t nat -A PREROUTING ! -d 192.168.1.121 -p tcp --dport 443 -j REDIRECT --to-ports 443
```

**squid.conf**

```
http_port 80 intercept
https_port 443 intercept
ssl-bump cert=/etc/squid3/ssl/ca/intermediate/certs/wilcard.pem key=/etc/squid3/ssl/ca/intermediate/private/wildcard.key generate-host-certificates=off version=4 options=NO_SSLv2,NO_SSLv3,SINGLE_DH_USE
cache_log /var/log/squid3/cache.log
access_log daemon:/var/log/squid3/access.log squid
netdb_filename stdio:/var/log/squid3/netdb.state
sslcrtd_program /usr/libexec/ssl_crtd -s /var/log/squid3/ssl_db -M 4MB sslcrtd_children 1 startup=1 idle=1
cache_effective_user proxy
cache_effective_group proxy
pinger_enable off
dns_v4_first on
acl HTTPS dstdomain "/etc/squid3/https"
acl BLOCK url_regex "(torrent)|sex(y|o)"
cache deny all
ssl_bump bump HTTPS
ssl_bump splice all
http_access deny BLOCK
http_access allow all
```
Mas o que realmente queremos é fazer man-in-the-middle em tráfego HTTPS, fazendo cache e aplicando ACLs como url_regex, sem precisar instalar cadeias de certificados em todas as máquinas clientes, certo?

Para o "bumping" de HTTPS, o Squid dispõe de duas maneiras diferentes de se lidar com certificados digitais:

* exigindo a criação de uma AC local (ou seja, certificado raiz contendo o campo CA:TRUE), ele re-assina os certificados dos servidores originais antes de repassar para os clientes (funcionalidade esta chamada de "Dynamic SSL Certificate Generation"); ou
* utilizando um único certificado digital, com o campo CA:FALSE.

Podemos descartar o primeiro cenário, pois esta exige a instalação de um certificado na máquina cliente.

Nos meus laboratórios, foquei no segundo caso. A princípio a solução seria solicitar um certificado á GlobalSign, utilizando como subdomínio wildcards de *gTLD* (exemplo: *.br, *.com, *.net no campo subjAltName do certificado). Mas isso vai contra as definições da RFC 6125. Os navegadores simplesmente impedem o acesso a sites com wildcards inválidos.

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

