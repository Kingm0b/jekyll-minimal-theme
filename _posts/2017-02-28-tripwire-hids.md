---
layout: post
title: "Detecção de intrusão em hosts com o Tripwire"
---

> Neste post, vamos discutir métodos para detecção de possíveis alterações no sistema em decorrência da ação de algum malware/atacante em sitemas Linux.

Objetivos:
Serão abordados de medidas básicas até soluções mais elaboradas para este tipo de detecção de intrusões por checagem de integridade de arquivos.

Detecção por enumeração de arquivos recentemente alterados:
Utilizando a ferramenta find.

Podemos fazer esta detecção baseando-se na data de alteração/criação de um arquivo teste.

Criando o arquivo:
```
# touch /tmp/teste.txt
```

Definindo uma nova data de criação que será utilizada como base para pesquisa com o find:

```
# touch -t 06012015 /tmp/teste.txt
```

*(Neste caso definimos como data de criação e modificação do arquivo teste.txt: 01/06/2015)*

Usando o find para encontrar arquivos com data de criação/modificação superior a 01/06/2015:

```
# find / -type f -cnewer /tmp/teste.txt
```

Comando otimizado para excluir diretórios com conteúdo dinâmico:

```
# find / -not -path "*/proc*" -and -not -path "*/sys*" -type f -cnewer /tmp/teste.txt
```

Desvantagens:
- Não garante detecção por modificação do arquivo, apenas informações MAC presentes no inode do mesmo.
- O atacante pode facilmente alterar o horário de criação/modificação do arquivo com o comando touch, despistando sua atuação.
- Indo mais além, ele poderia verificar o horário de criação/modificação de um arquivo antes de alterá-lo e definir a data como estava anteriormente após a alteração.

Detecção por teste de integridade de arquivos utilizando algoritmos de hash
Ferramenta utilizada: md5sum

Primeiro criaremos nossa base contendo uma lista arquivos e seus respectivos hashes:

```
# md5sum /etc/passwd /etc/shadow /root/arquivos/* > /root/database
```

Verificando conteúdo:

```
# cat /root/database
ca4a3966e08db1c8e0cbafe821cf6621  /etc/passwd
dada3a0aa04f329ac1b175d66d2b5c3b  /etc/shadow
06da4de7abf893665e1581fc82b1e845  /root/arquivos/1
06da4de7abf893665e1581fc82b1e845  /root/arquivos/2
06da4de7abf893665e1581fc82b1e845  /root/arquivos/3
06da4de7abf893665e1581fc82b1e845  /root/arquivos/4
93ed3b1dcb2eaf3112ba1c21855820d4  /root/arquivos/5
```

Fazendo checagens periódicas para detecção de alteração em um destes arquivos:

```
# md5sum -c database
/etc/passwd: SUCESSO
/etc/shadow: SUCESSO
/root/arquivos/1: SUCESSO
/root/arquivos/2: SUCESSO
/root/arquivos/3: SUCESSO
/root/arquivos/4: SUCESSO
/root/arquivos/5: SUCESSO
```

Caso algum arquivo seja alterado, teremos este retorno:

```
# md5sum -c database
/etc/passwd: FALHOU
/etc/shadow: SUCESSO
/root/arquivos/1: SUCESSO
/root/arquivos/2: SUCESSO
/root/arquivos/3: SUCESSO
/root/arquivos/4: SUCESSO
/root/arquivos/5: SUCESSO
md5sum: WARNING: 1 computed checksum did NOT match
```

Proteger database:

```
# chown root.root /root/database
# chmod 400 /root/database
# chattr +i /root/database
```

Desvantagens:
- A ferramenta md5sum (e suas variantes sha256sum, sha512sum ...) não faz checksum de diretórios e metadados dos arquivos (ex.: Não detecta alteração nos timestamps MAC de um arquivo e nem alterações em permissões).
- Caso o sistema seja comprometido, o comando md5sum e /root/database poderão ser alterados.

Utilizando o Tripwire

Instalação:

```
# apt-get install tripwire
```

Componentes 

```
tripwire	- a file integrity checker for UNIX systems
twprint		- Tripwire database and report printer
twadmin		- Tripwire administrative and utility tool
siggen		- signature gathering routine for Tripwire
```

Localização dos executáveis:
- /usr/sbin/tripwire
- /usr/sbin/twprint
- /usr/sbin/twadmin
- /usr/sbin/siggen

Arquivos de configuração:
/etc/tripwire
/etc/tripwire/twpol.txt
/etc/tripwire/twcfg.txt

Base de dados:
/var/lib/tripwire
/var/lib/$(HOSTNAME).twd - Base de dados contendo a relação de arquivos e seus respectivos hashes.
/var/lib/tripwire/report/$(HOSTNAME)-$(DATE).twr


Configuração do Tripwire:

Criando as chaves
﻿﻿
1 - Chave para assinatura dos arquivos de configuração do Tripwire:

```
twadmin --generate-keys --site-keyfile site.key
```

2 - Chave para uso local, assinatura da database e dos reports:

```
twadmin --generate-keys --local-keyfile $(hostname)-local.key
```

3 - Gerando o arquivo de configuração tw.cfg:

```
twadmin --create-cfgfile --cfgfile tw.cfg --site-keyfile site.key twcfg.txt
```

4 - Gerando arquivo de políticas tw.pol:

```
twadmin --create-polfile --cfgfile tw.cfg --polfile tw.pol --site-keyfile site.key twpol.txt
```

Interagindo com as ferramentas do TripWire

Visualizando conteúdo de tw.cfg:

```
twadmin --print-cfgfile --cfgfile /etc/tripwire/tw.cfg
```

Visualizando conteúdo de tw.pol:

```
twadmin --print-polfile --polfile /etc/tripwire/tw.pol
```

Gerando banco de dados de arquivos e respectivos hashes definidos segundo as políticas do tw.pol:

```
tripwire --init --cfgfile /etc/tripwire/tw.cfg --polfile /etc/tripwire/tw.pol
```

Fazendo checagem de integridade de arquivos da database:

```
tripwire --check --cfgfile /etc/tripwire/tw.cfg --polfile /etc/tripwire/tw.pol
```

Visualizando conteúdo de $(hostname).twd:

```
twprint --print-dbfile --dbfile /var/lib/tripwire/Tripwire.twd --local-keyfile /etc/tripwire/Tripwire-local.key
```

Visualizando conteúdo de algum report:

```
twprint --print-report --twrfile /var/lib/tripwire/report/Tripwire-20150606-145524.twr
```


Escrevendo políticas do Tripwire:

Por padrão, o Tripwire inclui no pacote um twpol.txt com regras pré-definidas. Acontece que estas políticas podem não estar de acordo com as necessidades do host a ser protegido (existem muitos arquivos e diretórios referenciados que podem não estar presentes no seu sistema). O ideal seria criar do zero novas regras (um novo twpol.txt), e para isto devemos entender como são escritas as regras para o TripWire.

As políticas são constituídas por quatro elementos: comentários, variáveis, regras e diretivas.

Atenção: Toda regra deverá terminar com ponto e vírgula ;

Comentários
São frases iniciadas com #

Ex:

```
# Isto é um comentário

/bin  ->  $(ReadOnly); # Isto também é um comentário
```

Variáveis
Sintaxe de declaração:

```
variavel = valor;
```

Exemplos:

```
senhas = /etc/shadow;
arquivos = /media/backup/;
```

Para se referir ao valor de alguma variável, utilizamos $(variavel), ex.:

```
$(senhas)
$(arquivos)
```

Regras
Sintaxe de declaração:

```
Objeto -> PropriedadeDoObjeto;
```

Na prática, um "objeto" é um diretório ou um arquivo que será monitorado. A "propriedade do objeto" ou, de acordo com a documentação, um property_mask, é um caractere ou conjunto de caracteres que representam quais informações deverão ser monitorados em um arquivo/diretório.

Segue a lista de possíveis caracteres aceitos como "máscara":

```
-      Ignore the following properties
+     Record and check the following properties
a     Access timestamp
b     Number of blocks allocated
c     Inode timestamp (create/modify)
d     ID of device on which inode resides
g     File owner's group ID
i     Inode number
l     File is increasing in size (a "growing file")
m     Modification timestamp
n     Number of links (inode reference count)
p     Permissions and file mode bits
r     ID of device pointed to by inode
      (valid only for device objects)
s     File size
t     File type
u     File owner's user ID
C     CRC-32 hash value
H     Haval hash value
M     MD5 hash value
S     SHA hash value
```

Com base no que vimos, é possível criar simples regras para o Tripwire, exemplo:

```
/root/arquivos/ -> +S;
```

Esta regra diz que deverão ser monitorados alterações nos hashes SHA dos arquivos abaixo do diretório "arquivos". O símbolo + deverá sempre estar precedendo os caracteres que constituem a máscara. Se quisermos monitorar também as datas de criação e modificação dos arquivos podemos adicionar o caractere c :

```
/root/arquivos/ -> +cS;
```

Lembrando que estas novas regras deverão ser definidas em um novo twpol.txt, encodada e assinada pelo twadmin e uma nova base deverá ser criada, tudo isto a cada alteração nas políticas do tw.pol.

A saída dos reports possuem este cabeçalho:

```
---------------------------------------------------------------------------------------------
  Section: Unix File System
---------------------------------------------------------------------------------------------

  Rule Name                       Severity Level    Added    Removed    Modified
  --------------                      ------------------   -----------   -------------  -----------
```

"Rule Name": Nome da regra. Por padrão, o nome da regra é o nome do arquivo/diretório monitorado. Podemos alterar isto por nomes mais sugestivos á administração.

"Severity Level": Grau de severidade. É possível definir níveis de criticidade nas regras, apenas para facilitar a análise por parte do administrador.

"Added": Adicionado. Caso seja criado arquivos no diretório monitorado.

"Removed": Removido. Caso algum diretório/arquivo monitorado pela regra for removido.

"Modified":  Modificado. Contabiliza caso houver modificações nos arquivos monitorados.


Podemos adicionar nomes e severidade nas regras da seguinte forma, exemplo de twpol.txt:

```
/root/arquivos/ -> +cS (rulename="Arquivos da Empresa" ,severity=50);
/tmp/ -> +m (rulename="Temporarios" ,severity=10);
/etc/passwd -> +Sc (rulename="Arquivos do Sistema", severity=100);
```

Como resultado, teremos algo como isto nos reports da checagem:

```
-----------------------------------------------------------------------------------------
  Section: Unix File System
-----------------------------------------------------------------------------------------

  Rule Name                                Severity Level    Added    Removed  Modified
  ---------                                      ------------------   ----------  -------------  --------
  Arquivos da Empresa            50                         0              0                   0        
  (/root/arquivos)
* Temporarios                          10                          0              0                   1        
  (/tmp)
  Arquivos do Sistema               100                       0              0                    0        
  (/etc/passwd)

Total objects scanned:  17
Total violations found:  1
```

Se na regra "Arquivos do Sistema" quisermos aumentar a quantidade de arquivos monitorados, poderemos proceder desta maneira:

**ATENÇÃO**: Esta maneira não é a recomendada

```
/root/arquivos/ -> +cS (rulename="Arquivos da Empresa" ,severity=50);
/tmp/ -> +m (rulename="Temporarios" ,severity=10);
/etc/passwd -> +Sc (rulename="Arquivos do Sistema", severity=100);
/etc/shadow -> +Sc (rulename="Arquivos do Sistema", severity=100);
/etc/motd -> +Sc (rulename="Arquivos do Sistema", severity=100);
```

Uma forma mais elegante poderia ser:

```
/root/arquivos/ -> +cS (rulename="Arquivos da Empresa" , severity=50);
/tmp/ -> +m (rulename="Temporarios", severity=10);

(rulename="Arquivos do Sistema", severity=100) {
        /etc/passwd -> +Sc;
        /etc/shadow -> +Sc;
        /etc/motd -> +Sc;
}
```

O formato:

```
(attribute list)
{
    rule list;
}
```

Permite uma melhor estrutura para agrupamento de regras.

As regras suportam o operador de negação para especificar objetos (arquivos ou diretórios) que NÃO deverão ser checados pelo TripWire. Exemplos:

```
/root/arquivos/ -> +cS (rulename="Arquivos da Empresa" , severity=50);
/tmp/ -> +m (rulename="Temporarios", severity=10);

(rulename="Arquivos do Sistema", severity=100) {
        /etc/passwd -> +Sc;
        /etc/shadow -> +Sc;
        /etc/motd -> +Sc;
        !/proc;
        !/sys;
}
```

### Diretivas

O Tripwire suporta com o conceito de "diretivas". O propósito desta funcionalidade é permitir debug de políticas e aplicação condicional das regras. Segundo a documentação, o principal objetivo das "diretivas" do Tripwire é o compartilhamento de políticas para diversas máquinas, ou seja, posso ter vários servidores utilizando o mesmo tw.pol, por exemplo, mesmo possuindo arquivos e hierarquia de diretórios diferentes.

A sintaxe de uma diretiva é como se segue:

```
@@ diretiva [argumentos opcionais]
```

As seções default são as seguintes:

@@section   # Declara uma seção, na maioria das vezes haverá uma seção GLOBAL e/ou uma FS.  Uma seção GLOBAL é utilizada apenas para declarar variáveis que poderão ser reaproveitadas em outros arquivos de política. Uma seção FS define toda a área do arquivo de políticas onde serão encontradas definição de máscaras, variáveis e regras que afetem sistemas de arquivos UNIX. Em sistemas Windows esta seção será referenciada como @@section NTFS.

@@ifhost     # Permite uma aplicação condicional
@@else        # de uma regra.
@@endif

@@print    # Imprime uma mensagem
@@error   # Imprime uma mensagem de erro

@@end     # Marks the logical end-of-file.



### Monitoramento seletivo com Tripwire


Não é possível realizar checagens em regras específicas das políticas do Tripwire individualmente, ou você checa todas a regras de um determinado arquivo de políticas ou não realiza nenhuma checagem ;)

A ideia para que seja possível fazer verificações em regras específicas é declarando-as em arquivos de políticas diferentes. Uma vez criado o arquivo de políticas, deverá ser criado também seu respectivo arquivo de configuração.

Situação exemplo:

Determinadas políticas deverão ser checadas com mais frequência que outras.

Objetivo:

Criar dois arquivos de políticas e dois arquivos de configuração (um para cada database e política).

Por organização, é interessante colocar cada tw.pol e tw.cfg em diretórios diferentes.

```
root@WordPress:~# mkdir /etc/tripwire/arquivosdossistema
root@WordPress:~# mkdir /etc/tripwire/arquivosdefirewall
root@WordPress:~#
root@WordPress:~# cp /etc/tripwire/twcfg.txt /etc/tripwire/arquivosdossistema/
root@WordPress:~# cp /etc/tripwire/twcfg.txt /etc/tripwire/arquivosdefirewall/
root@WordPress:~#
```

Conteúdo de /etc/tripwire/arquivosdossistema/twcfg.txt

```
ROOT          =/usr/sbin
POLFILE       =/etc/tripwire/arquivosdosistema/tw.pol
DBFILE        =/var/lib/tripwire/$(HOSTNAME)-1.twd
REPORTFILE    =/var/lib/tripwire/report/$(HOSTNAME)-$(DATE).twr
SITEKEYFILE   =/etc/tripwire/site.key
LOCALKEYFILE  =/etc/tripwire/$(HOSTNAME)-local.key
EDITOR        =/usr/bin/editor
LATEPROMPTING =false
LOOSEDIRECTORYCHECKING =false
MAILNOVIOLATIONS =true
EMAILREPORTLEVEL =3
REPORTLEVEL   =3
SYSLOGREPORTING =true
MAILMETHOD    =SMTP
SMTPHOST      =localhost
SMTPPORT      =25
TEMPDIRECTORY =/tmp
```

Conteúdo de /etc/tripwire/arquivosdefirewall/twcfg.txt

```
ROOT          =/usr/sbin
POLFILE       =/etc/tripwire/arquivosdefirewall/tw.pol
DBFILE        =/var/lib/tripwire/$(HOSTNAME)-2.twd
REPORTFILE    =/var/lib/tripwire/report/$(HOSTNAME)-$(DATE).twr
SITEKEYFILE   =/etc/tripwire/site.key
LOCALKEYFILE  =/etc/tripwire/$(HOSTNAME)-local.key
EDITOR        =/usr/bin/editor
LATEPROMPTING =false
LOOSEDIRECTORYCHECKING =false
MAILNOVIOLATIONS =true
EMAILREPORTLEVEL =3
REPORTLEVEL   =3
SYSLOGREPORTING =true
MAILMETHOD    =SMTP
SMTPHOST      =localhost
SMTPPORT      =25
TEMPDIRECTORY =/tmp
```

```
~# twadmin --create-cfgfile --cfgfile /etc/tripwire/arquivosdosistema/tw.cfg --site-keyfile site.key /etc/tripwire/arquivosdosistema/twcfg.txt
```

```
~# twadmin --create-cfgfile --cfgfile /etc/tripwire/arquivosdefirewall/tw.cfg --site-keyfile site.key /etc/tripwire/arquivosdefirewall/twcfg.txt
```

```
~# twadmin --create-polfile --cfgfile /etc/tripwire/arquivosdosistema/tw.cfg --polfile /etc/tripwire/arquivosdosistema/tw.pol --site-keyfile site.key /etc/tripwire/arquivosdosistema/twpol.txt
```

```
~# twadmin --create-polfile --cfgfile /etc/tripwire/arquivosdefirewall/tw.cfg --polfile /etc/tripwire/arquivosdefirewall/tw.pol --site-keyfile site.key /etc/tripwire/arquivosdefirewall/twpol.txt
```

```
~# tripwire --init --cfgfile /etc/tripwire/arquivosdosistema/tw.cfg --polfile /etc/tripwire/arquivosdosistema/tw.pol
```

```
~# tripwire --init --cfgfile /etc/tripwire/arquivosdefirewall/tw.cfg --polfile /etc/tripwire/arquivosdefirewall/tw.pol
```

As checagens poderão ocorrer individualmente:

```
~# tripwire --check --cfgfile /etc/tripwire/arquivosdosistema/tw.cfg --polfile /etc/tripwire/arquivosdosistema/tw.pol
```

```
~# tripwire --check --cfgfile /etc/tripwire/arquivosdefirewall/tw.cfg --polfile /etc/tripwire/arquivosdefirewall/tw.pol
```



**F**NM**AQ** (Frequently *Not Much* Asked Questions):

1 - O que está contido em site.key e local.key?
R: Um par de chaves criptográficas: uma pública e outra privada, geradas pelo algoritmo ELGAMAL e compressadas pelo algoritmo Lempel-Ziv 1977 implementada pela biblioteca zLib.

2 - Qual a diferença entre o site.key e o local.key?
R: O site.key é utilizado para assinar os arquivos de configuração do Tripwire (ex.: tw.cfg e tw.pol) e o local.key é usado para assinar o banco de dados de arquivos monitorados (/var/lib/HOSTNAME.twd) e os reports (/var/lib/reports/*).

3 - Porque o Tripwire utiliza dois pares de chaves?
R: A ideia do site.key é ser utilizada em várias instalações diferentes do Tripwire (em hosts diferentes utilizar o mesmo par de chaves site.key). E o local.key deverá ser utilizado apenas localmente por um único host (isto explica porque da necessidade do nome do arquivo contiver o hostname da máquina).

4 - Como é feito a assinatura digital dos arquivos?
R: Utilizando o conceito convencional de assinatura mesmo. O diferencial é que a chave privada fica comprensada junto com a pública em um único arquivo e extraída quando necessária sua utilização.

5 - Porque temos as versões binárias de twcfg.txt e twpol.txt ?
R: Isto evita, caso o sistema estiver sido comprometido, que o atacante possa alterar os caminhos absolutos dos componentes do Tripwire (no caso do tw.cfg) e também evita que as políticas de verificação do mesmo sejam maliciosamente alteradas com o objetivo de acobertar alterações do atacante/malware (no caso do tw.pol). Estes arquivos, além de estarem no formato binário, são assinados digitalmente pela chave privada de site.key.

6 - Porque ao gerar a database pela primeira vez temos que informar a senha do local.key?
R: Porque o arquivo .twd gerado deverá ser assinado pela chave privada de local.key.

7 - Onde está armazenado a lista de arquivos monitorados e seus respectivos hashes?
R: Em /var/lib/tripwire/HOSTNAME.twd

8 - O Tripwire também monitora inodes? Ou apenas os arquivos em si?
R: O Tripwire não só monitora alterações nos arquivos em si, como também em seus metadados, ex.: alterações em permissões e em datas de acesso são detectadas.


Links:

http://linux.die.net/man/8/twintro
http://linux.die.net/man/4/twpolicy
http://manpages.ubuntu.com/manpages/hardy/man5/twfiles.5.html
http://linux.die.net/man/8/twadmin
http://www.tripwire.com/
http://www.tripwire.org/
http://www.linux-community.de/Internal/Artikel/Print-Artikel/LinuxUser/2013/02/Einbrueche-erkennen-mit-dem-IDS-Tripwire/%28article_body_offset%29/2
https://www.digitalocean.com/community/tutorials/how-to-use-tripwire-to-detect-server-intrusions-on-an-ubuntu-vps
http://www.linux-magazine.com/Online/Features/Detecting-Attackers-with-Tripwire
http://www.linuxsecurity.com.br/article.php?sid=306
http://www.vivaolinux.com.br/artigo/Tripwire-Checando-a-integridade-do-sistema
http://adrenaline.uol.com.br/forum/area-linux-e-open-source/245637-tripwire-em-linux.html
http://www.lsi.usp.br/~volnys/courses/presentations/LinuxSecurity/LinuxSecurity-colorida.PDF
http://www.labbas.eng.uerj.br/links/linux/pg101.html
http://189-90-102-190.life.com.br/article.php?sid=1253
http://manpages.ubuntu.com/manpages/saucy/man5/twfiles.5.html
https://www.safaribooksonline.com/library/view/linux-security-cookbook/0596003919/ch01s01.html
http://www.linuxjournal.com/article/8758?page=0,1
http://erikimh.com/tripwire/

