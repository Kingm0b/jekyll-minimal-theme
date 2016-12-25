---
layout: post
title: "Conexão PPPoE no Debian 8"
---

> O PPPoE é ainda o protocolo mais utilizado pelos provedores xDSL. Aqui, documento algumas experiências básicas que vivenciei com este tipo de tecnologia.

De posse de um *Opticom DSLink*, desabilitei o Wifi junto com vários outros serviços habilitados (UPnP, DHCP, Telnet e FTP). Em seguida, re-configurei as definições de conexão com a WAN, configurando o roteadorzinho SOHO a trabalhar como **bridge**. A ideia de configurar o roteador como bridge é para permitir que o tráfego PPPoE gerado pela minha máquina consiga alcançar o PPPoE server do provedor.

A minha missão era configurar um roteador/firewall rodando Debian 8 para autenticar e receber um endereço público do provedor.

Graças aos desenvolvedores do daemon **pppd**, os caras do "Roaring Penguin" com seu **rp-pppoe.so** e os empacotadores do Debian, conexões PPPoE no Linux pode ser bastante simples ("pode ser simples" se as suas necessidades forem simples, dê [uma olhadinha na man page do pppd](https://ppp.samba.org/pppd.html) se você quiser ver o que é flexibilidade rs).

Toda a configuração, *for the impatient*, se resume nestes passos:

```bash
# apt-get update
# apt-get install -y ppp pppoeconf
# pppoeconf
```

Em uma configuração mais detalhada, segue:

#### 1 - Instale o pppd:

```bash
# apt-get update
# apt-get install ppp
```
Antes de partir para o próximo passo, vamos ver o estado em que se encontra aos arquivos e diretórios do ppp:

```bash
# ls -l /etc/ppp/
total 60
-rw------- 1 root root    80 Dez 24 10:03 chap-secrets
-rwxr-xr-x 1 root root  1754 Abr 14  2015 ip-down
drwxr-xr-x 2 root root  4096 Dez 24 10:03 ip-down.d
-rwxr-xr-x 1 root root  1892 Abr 14  2015 ip-up
drwxr-xr-x 2 root root  4096 Dez 24 10:03 ip-up.d
-rwxr-xr-x 1 root root   784 Abr 14  2015 ipv6-down
drwxr-xr-x 2 root root  4096 Abr 14  2015 ipv6-down.d
-rwxr-xr-x 1 root root   922 Abr 14  2015 ipv6-up
drwxr-xr-x 2 root root  4096 Abr 14  2015 ipv6-up.d
-rw-r--r-- 1 root root 13209 Abr 14  2015 options
-rw------- 1 root root  1628 Dez 24 10:03 pap-secrets
drwxr-s--- 2 root dip   4096 Dez 24 10:03 peers
```

Observe o conteúdo do arquivo *chap-secrets*:

```bash
# cat /etc/ppp/chap-secrets
```

e *pap-secrets*:

```bash
# cat /etc/ppp/pap-secrets
```

O diretório *peers* contém praticamente todas as informações que o pppd precisa para estabelecer uma conexão:

```bash
# ls -l /etc/ppp/peers/
total 4
-rw-r----- 1 root dip 1093 Dez 24 10:03 provider
```

Por padrão, ele só contém o arquivo *provider*, **ignore-o**.

Instale o pppoeconf:

```bash
# apt-get install pppoeconf
```

Este utilitário é um script em shell que, usando o *dialog* como interface com o usuário, automatiza todo o processo de criação dos arquivos de configuração do pppd.

Caso seu roteador Linux já esteja em produção e o link DSL que você está configurando **não** for o seu link default: CUIDADO! Se você afobar e confirmar todas as mensagens sem ler antes, o pppoeconf irá substituir a rota default do seu roteador pela rota default do provedor DSL (o que pode não ser uma boa).

De qualquer forma, mesmo o link DSL não sendo o seu link de conexão principal com a Internet, você pode se beneficiar do **pppoeconf** apenas tomando muito cuidado nos procedimentos finais deste utilitário!

#### 2 - Configure o pppd:

Utilizando o pppoeconf (tudo é mais fácil com ele), caso a interface de rede que estiver diretamente conectada com o roteador DSL for a **eth0**, você pode executar o seguinte comando:

```bash
# pppoeconf eth0
```

A partir deste momento, uma tela em ncurses irá te guiar no processo de configuração.

**Obs**: Se você omitir o nome da interface de rede, o pppoeconf irá tentar detectar a interface correta automaticamente. Se você tiver muitas interfaces, isso pode não ser uma boa (pois pode demorar muito e/ou ele pode ignorar a interface correta e te falar que não encontrou).

Em uma das mensagens iniciais, você receberá este questionamento:

![](https://raw.githubusercontent.com/m0blabs/m0blabs.github.io/master/images/2016-12-25/pppoeconf-1.png)

Novamente, se o seu roteador Linux já estiver em produção e o link que você está configurando não for o principal, marque "**Não**".

No questionamento final, você terá o seguinte:

![](https://raw.githubusercontent.com/m0blabs/m0blabs.github.io/master/images/2016-12-25/pppoeconf-2.png)

Marque "**Não**" para podermos acrescentar algumas configurações no próximo passo.

#### 3 - Ajustando as configurações

Observe novamente o conteúdo dos arquivos <code>/etc/ppp/chap-secrets</code> e <code>/etc/ppp/pap-secrets</code>. Não sabendo qual o protocolo de autenticação suportado pelo provedor, o pppoeconf definiu as credenciais fornecidas á ele nestes dois arquivos.

Listando novamente o diretório *peers*, temos:

```
root@debian:~# ls -l /etc/ppp/peers/
total 8
-rw-r----- 1 root dip  258 Dez 24 13:03 dsl-provider
-rw-r----- 1 root dip 1093 Dez 24 10:03 provider
```

Surgiu um novo arquivo: **dsl-provider**. Para agilizar, este é o conteúdo do meu dsl-provider:

```
# Habilita debug - output no syslog
debug

# Endereço IP e Rotas
noipdefault
nodefaultroute
replacedefaultroute

#lcp-echo-interval 30
#lcp-echo-failure 4
noauth
mtu 1492

# Tolerância a "falhas"
persist
maxfail 0
holdoff 20

plugin rp-pppoe.so eth0

# Informações de autenticação
hide-password
user "usuario@provedor"
```

De longe, o protocolo LCP pode ser visto como uma espécie de "*ICMP do PPP*". Ou seja, ele tem a função de permitir que mensagens sejam trocadas entre os pares da comunicação. Ele coordena os três estágios da conexão:

* 
*
*



#### 4 - Ativando o link


### Esclarecimentos

#### Sobre a rota default

#### O login e senha do provedor é importante?

#### Como se dá o *discover* de servidores PPPoE


#### Como são distribuídos os IPs públicos por PPPoE

Para a "configuração" do Internet Protocol (isto é, a distribuição dos endereços IPv4) sobre PPPoE, é utilizado um protocolo próprio chamado **IPCP** (*IP Control Protocol*, Protocolo de Controle do IP).

Basicamente, o cliente envia uma "*Configuration Request*" e o servidor PPPoE do provedor responde com um "*Configuration ACK*" já com o novo IP público do cliente e endereço de servidores DNS.

Em outras palavras, o uso do IPCP para concessão de endereços IP é uma espécie de DHCP a nível de enlace.

Para ilustrar melhor esse processo, veja essa captura no momento do recebimento do endereço IP:

![](https://raw.githubusercontent.com/m0blabs/m0blabs.github.io/master/images/2016-12-25/wireshark-ipcp.jpg)

Neste caso, o roteador Cisco do provedor transmitiu para o cliente uma mensagem IPCP do tipo "*Configuration Nak*", no campo "*Options*" desta mensagem temos o IP concedido: 201.2.31.31. De posse deste endereço, cabe ao equipamento setar este IP á sua interface *ppp*.


#### Qual a relação entre PPPoE e redes ATM ?

Se estou utilizando PPPoE, por que tenho que definir configurações de redes ATM no meu modem (números de VCI e VPI) ?

Por mais que o link entre o seu modem e o roteador do provedor seja Ethernet, o tráfego é ainda encapsulado em frames ATM para chegar aos equipamentos do provedor. Este tipo de gambiarra é chamado por *PPPoE ATM over ADSL*. E por mais estranho que possa parecer este arranjo, é a implementação mais utilizada pelos ISPs!

Ou seja, todos os pacotes IP que partem da sua rede em direção à Internet, primeiro são encapsulados em frames PPP, em seguida são re-encapsulados em frames Ethernet e, por fim, recebem cabeçalhos ATM segundo o padrão AAL5.

EOF