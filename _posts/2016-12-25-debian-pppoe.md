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

#### Ajuste na MTU dos frames Ethernet

Os frames Ethernet, ao todo, "pesam" no máximo 1518 bytes! Sendo 18 bytes de cabeçalho e 1500 bytes de payload.

Todos os pacotes transmitidos em uma rede PPPoE são primeiro encapsulados em frames PPP. Cada frame possui 8 bytes de cabeçalho.

Portanto, o tamanho máximo do payload Ethernet = header PPP + pacote.

Isto quer dizer que não podemos transmitir pacotes de 1500 bytes. Em razão do overhead gerado por causa do PPP, nossos pacotes deverão ter no máximo **1492 bytes** de tamanho (8 bytes do PPP + 1492 do pacote = 1500 bytes).

Por conta disto, para evitar descarte dos pacotes no roteador, temos que **forçar** nossos segmentos TCP a sempre trabalharem com um MSS diferente do convencional (que é 1448 bytes - o MSS máximo é 1460, mas usualmente os SOs preferem trabalhar com 1448), redefinindo o MSS para 1440 (8 bytes a menos).

Mas aí você poderia se perguntar: e o UDP? Ele não tem campo ou extensão em seu cabeçalho que defina um tamanho máximo para que se adeque a rede em questão. Isso quer dizer que perderei datagramas UDP em uma possível comunicação com a Internet?

Sim! Seu pacotes serão descartados pelo roteador se possuírem tamanho igual ou superior a 1441 bytes.

Mas felizmente, **por convenção** (não há uma limitação técnica que defina isto, tipo número de bits), em razão de sua natureza *connectionless*, o tamanho máximo do payload considerado seguro de datagramas UDP não ultrapassam *512 bytes*.

O ajuste do MSS no TCP é feito por meio do módulo **tcpmss** do iptables/netfilter. No seguinte arquivo, temos:

```bash
# cat /etc/ppp/ip-up.d/0clampmss 

#!/bin/sh
# Enable MSS clamping (autogenerated by pppoeconf)

iptables -t mangle -o "$PPP_IFACE" --insert FORWARD 1 -p tcp --tcp-flags SYN,RST SYN -m tcpmss --mss 1400:65495 -j TCPMSS --clamp-mss-to-pmtu
```

#### O login e senha do provedor é importante?

Na época em que as conexões *dial-up* dominavam, os **provedores de acesso** eram de extrema importância, pois eles realmente proviam acesso a Internet. Por meio de uma placa fax-modem os computadores atuavam como verdadeiros telefones, discando para os RAS (atuando como ramais telefônicos) dos provedores e em seguida fornecendo informações para autenticação. Sem as crendenciais de login, o cliente não teria acesso a Internet.

Nas conexões banda larga, o processo é totalmente diferente.

Qual a necessidade de um provedor ADSL exigir autenticação PPP? Segurança? Controle?

O conceito de "**provedor de internet**" sempre foi muito confuso para muitos. Afinal, se é a Oi, NET ou Vivo quem disponibiliza acesso a Internet, por que pagar a *Uol* para isto?

A autenticação é uma funcionalidade **opcional** do PPP, provida pelo LCP.

Se formos parar para observar, os provedores mantém um link **dedicado** entre as residências e seus roteadores/DSLAMs. Eles tem total controle sobre a disponibilização do sinal DSL e liberação do acesso a Internet.

Se sua internet for cortada por falta de pagamento, não adianta pegar o login e senha PPP do vizinho e setar no seu modem. Não vai funcionar.

A autenticação como justificativa dos provedores para identificação dos seus clientes, não procede, pois eles **já sabem quem você é** (eles têm um link dedicado com você). Além disto, não é possível ter mais de um cliente PPP sobre o mesmo link (afinal, o link é **ponto-a-ponto**, só há dois *peers* na comunicação).

Para continuar ganhando dinheiro, os provedores de acesso fizeram parcerias com os reais ISPs para portar o conceito de "provedor discado" ao mundo da banda larga.

Durante mais de 10 anos os servidores PPP dos provedores terceirizavam a autenticação para servidores RADIUS. Estes servidores RADIUS validavam as credenciais fornecidas via PPP junto aos sistemas dos supostos "provedores de acesso".

Realmente, se você não tivesse uma conta no Uol ou iG (por exemplo) você não conseguiria ter acesso a Internet.

Hoje, isto é considerado venda casada! Você pode opcionalmente pagar por um "provedor". Mas se quiser, também pode utilizar de "*provedores gratuitos*" dependendo de quem está te fornecendo o acesso, como:

* Login: oi@oi
* Senha: oi

* Login: turbonet@turbonet
* Senha: gvt25

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
