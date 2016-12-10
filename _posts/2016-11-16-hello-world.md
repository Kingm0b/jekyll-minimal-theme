---
layout: post
title: "Indecisões do \"primeiro\" blog"
---

> Neste post meio "Hello World", detalho minhas indecisões que levaram ao surgimento deste blog.

Desde o início meu objetivo era utilizar a plataforma **Medium**. Já há um bom tempo sou fascinado por essa plataforma por sua maneira elegante e clara de se disponibilizar conteúdo. Alguns links que me chamaram a atenção uns tempos atrás:

* [Medium: seu próximo blog](https://medium.com/web-tecnologia/medium-seu-proximo-blog-9ecd110985e1)
* [Por que a indústria do empreendedorismo de palco irá destruir você.](https://medium.com/o-novo-mercado/porque-a-ind%C3%BAstria-do-empreendedorismo-de-palco-ir%C3%A1-destruir-voc%C3%AA-3e18309ab47f)
* [Porque os jovens profissionais da geração Y estão infelizes](http://qga.com.br/comportamento/jovem/2013/09/porque-os-jovens-profissionais-da-geracao-y-estao-infelizes)

O último link lembro de ter lido do Medium, mas não achei o link original. Esses são apenas alguns exemplos... mas já li *muita* coisa deste site.

Mas estudando mais as características da plataforma, descobri que não era o ambiente certo para o tipo de conteúdo que eu esperava publicar. Características como "ambiente colaborativo", "coisas que importam", e etc, me desmotivou. A impressão que dava é que todos os meus posts deveriam ser uma espécie de transcrição de uma palestra do **TED** rs.

Procurando por outras plataformas gratuitas com templates simplistas, encontrei o Postagon. Fiquei empolgado com ele. Olhando no Google, vi que várias pessoas já o utilizam:

![Google](https://github.com/Kingm0b/kingm0b.github.io/raw/master/images/2016-11-16/google.png "Resultados da busca no Google")
_site:postagon.com_


Criei uma conta, e publiquei um artigo lá. Até descobrir que teria que pagar uma taxa $ 7.99 dólares todo mês para permitir que todo mundo visualize o blog sem precisar criar uma conta.

No fundo, no fundo, eu queria mesmo era evitar usar as duas plataformas de blog mainstream: Blogger e Wordpress. Já tive experiências com ambas. Tava afim de algo diferente.

Então, você poderia pensar, já que está tão exigente assim: **FAÇA SEU PRÓPRIO SITE** !!!

Acontece que sou muito metódico, e fazer do zero meu próprio site com HTML e CSS iria me tirar o foco (que seria o compartilhamento dos meus artigos, e não ficar ajustando a margem ou o padding de box CSS).

![Google](https://github.com/Kingm0b/kingm0b.github.io/raw/master/images/2016-11-16/cafe.jpeg "CSS é impressionante!")

Foi aí que eu lembrei do Bootstrap e seus templates prontos:

[http://getbootstrap.com/getting-started/#examples-framework](http://getbootstrap.com/getting-started/#examples-framework)

Blz! Era isso que eu queria. Cheguei a fazer todo o site estático usando ele. Mas tinha um probleminha: onde eu iria hospedá-lo? De cara, a hospedagem iria ser no Dropbox mesmo… até receber este e-mail em 31/08/2016:

![Google](https://github.com/Kingm0b/kingm0b.github.io/raw/master/images/2016-11-16/dropbox.png "Triste mensagem do Dropbox")

Ok, mas serviços de hospedagem gratuita de HTML é o que mais tem por aí, eu pensei. Lembro que há uns 4 anos atrás estes serviços reinavam! Até o Bol tinha um serviço desses. O HDFree era muito bom. Mas hoje as coisas se inverteram. Está mais fácil encontrar [hospedagem gratuita com suporte a PHP e MySQL](http://pt-br.lmgtfy.com/?q=html%20free%20hosting) do que somente HTML. E qual o problema? **R**: sou paranóico.

Não queria serviços de hospedagem brasileiros. Queria algum serviço gringo, gratuito e sem propagandas. Quando vi que estava perdendo muito tempo nessa procura, resolvi apelar ao XPG mesmo. Apesar de ser um serviço brasileiro, que armazena sua senha em texto puro, e que (agora) insere propagandas nas páginas alheias, descobri uma forma de esconder as propagandas. Elas eram chamadas por meio de funções Javascript disponibilizadas no arquivo tag.js, algo assim:

```html
<script type="text/javascript" src="http://js.xpg.com.br/tag/h/xuxa/tag.js"></script>
```

A linha a cima era injetada no final do bloco da tag <code><head></code>.

**Solução**: Criar meu próprio tag.js declarando todas as funções do XPG, só que vazias:

```html
<script type="text/javascript" src="tag.js"></script>
```

Sendo “tag.js” com o seguinte conteúdo:

```javascript
function XPGLog() {}
function XPGLocation() {}
function XPGRandom() {}
function XPGCodeHTML() {}
function XPGAppendHrefCSS() {}
function XPGAppendCodeCSS() {}
function XPGLoadJS() {}
function XPGCodeJS() {}
function XPGAppendJS() {}
function XPGLockedByURL() {}
function XPGSnippetCriteo() {}
function crtg_getCookie() {}
function XPGBanner() {}
function XPGBannerTag() {}
function XPGBannerDHTML() {}
function XPGAff() {}
function XPGAffTag() {}
function XPGBannerDimension() {}
```

Declarando estas funções já no início da minha tag <head>, os browsers dão preferência a elas do que as originais, declaradas depois ao fim da tag. Foi aí que passado um tempo, eles descobriram a técnica e começaram a declarar o script deles bem antes, até mesmo, da tag <html>. Apesar de quebrar todo o padrão e a semântica… **XPG Wins**!

![Google](https://github.com/Kingm0b/kingm0b.github.io/raw/master/images/2016-11-16/html-xpg.png "Source Code")
_Observem: Tag script entre DOCTYPE e html na linha 2 vs minha tag script na linha 5_


Tudo bem. Apesar do mundo estiver conspirando para eu não possuir um blogzinho, continuei pela minha busca pela hospedagem perfeita (lembre-se: ela tinha que ser gringa, sem propagandas, sem suporte a PHP e gratuita).

Até que eu conheci o **GitHub Pages**: nada melhor do que hospedar minha página na maior “rede social” nerd do mundo. Cheguei a hospedar o sitezinho aqui por alguns dias. Mas algo começou a me incomodar… não estava curtindo o visual defaultzão do meu site em bootstrap. Foi aí que passei a procurar por algum “Static Site Generator” que gerava páginas o mais semelhante possível ao Medium.

Quando me deparei com este site: https://variadic.me, me convenci de que o **Hakyll** era o que eu precisava. Mas esse lance de ter que mecher com haskell me desapontava. Não me entendam mal: eu gosto de aprender coisas novas… mas só estava afim de algo simples, *fast-food*, sem curva de aprendizado.

Cheguei até a apelar para o HTML puro mesmo, depois de ver o **["Este é um website do caralho"](https://websitedocaralho.com.br/)**. Porque não? Olha que bacana estas páginas:

* http://www.tcpdump.org/pcap.html
* http://thobias.org/doc/shell_bd.html

Até que recentemente eu me deparei com o **Bashblog**:

[https://github.com/cfenollosa/bashblog](https://github.com/cfenollosa/bashblog)


Fiquei muuuito empolgado com esse projeto! Esta ferramenta foi inteiramente escrita em bash script e integrado com o editor de texto que você definir na variável de ambiente $EDITOR (vim ou  nano, na maioria dos casos). Toppzeira (hauahua) !!!

Quando estava conformado com o uso do bashblog para a geração das páginas estáticas do meu sitezinho no GitHub Pages, percebi que o Medium estava “perdendo a sua essência”:

* [Como configurar seu domínio no Medium](https://marceloscherer.com.br/como-configurar-seu-dom%C3%ADnio-no-medium-21acde390f1a)
* [O Medium mudou. Você está pronto?](https://medium.com/brasil/o-medium-mudou-voc%C3%AA-est%C3%A1-pronto-89b0cc240c4d)
* [How I Wrote a Client-Server App in Two Minutes Flat](https://medium.com/@bartobri/how-i-wrote-a-client-server-app-in-two-minutes-flat-ed83807388bb)
* [Medium’s CSS is actually pretty f***ing good.](https://medium.com/@fat/mediums-css-is-actually-pretty-fucking-good-b8e2a6c78b06)
* [Sketch Tutorial](https://medium.com/google-design/sketch-tutorial_01-b76271a095e3)
* [Linux kernel bug delivers corrupt TCP/IP data to Mesos, Kubernetes, Docker containers](https://tech.vijayp.ca/linux-kernel-bug-delivers-corrupt-tcp-ip-data-to-mesos-kubernetes-docker-containers-4986f88f7a19)


Opa! Era isso que eu queria haha


Novamente, me empolguei com a ideia de usar o Medium.com. Apesar de agora ele ter objetivos mais semelhantes aos blogs convencionais, algumas coisinhas ainda me incomoda: a divulgação automática dos meus posts aumentaria muito os _views_ das páginas, e a interação com os leitores também não me é um atrativo.

Ou seja, a hospedagem que eu estava procurando tinha que ser gringa, sem propagandas, sem suporte a PHP, gratuita, escondida do público e sem interação com os leitores hauhauahau.

Estou meio calejado com _"esses negócios de Internet"_ nesta minha vida. Não quero fazer questão de divulgar esta página.

Minha pretensão aqui é "consolidar" o resultado dos meus estudos na forma de artigos. Para documentar as coisas pra eu mesmo. Inclusive, este primeiro post é mais uma encheção de linguiça do que um desabafo (ou algo do tipo). A ideia aqui é só pegar o jeito e preparar terreno para os próximos posts.

Enfim, continuando a saga...

Observando o rodapé da página do [Aurelio.net](http://aurelio.net) vi que ele utiliza um tal de... **Jekyll**.

![Imagem](https://github.com/Kingm0b/kingm0b.github.io/raw/master/images/2016-11-16/rodape-aurelio.png "Rodapé do site do Aurelio")

O Jekyll é um gerador de sites estáticos desenvolvido em ruby. Ele se sustenta sobre dois outros projetos (também em ruby): o Kramdown e o Liquid. O Kramdown é um "conversor" de Markdown para HTML. O Liquid é um "**template engine**", basicamente, uma ferramenta que te permite declarar variáveis, controle de decisão e loops dentro de códigos HTML, para permitir que em tempo de execução páginas HTML estáticas sejam geradas, reaproveitando a estrutura da página e "dinamizando" o seu conteúdo.

A grande sacada do Jekyll é fazer com que os posts do blog (escrito em markdown) sejam automaticamente convertidas em HTML (usando o Kramdown) e inseridas na estrutura do template do blog (usando o Liquid) automaticamente.

Se você quiser "sentir o gostinho" de como é isto na prática, siga o mini-tutorial:

1° - Instale o git e o pacote ruby (o que além do interpretador, inclui o *gem*):

```bash
# pacman -Sy git ruby
# apt-get update && apt-get install -y git ruby
```

2° - Redefina sua variável de ambiente $PATH acrescentando o diretório onde as gems do usuário serão instaladas:

```bash
$ PATH="$(ruby -e 'print Gem.user_dir')/bin:$PATH"
```
3° - Liste os gems atualmente instalados (isto é apenas para você ter uma ideia do que está fazendo):

```bash
$ gem list
```

4° - Instale o Jekyll:

```bash
$ gem install jekyll
```

5° - Liste novamente os gems instalados:

```bash
$ gem list
```

Observe que, dentre os novos gems, temos o kramdown, o liquid e o próprio jekyll (claro).

6° - Baixe um template de exemplo do GitHub:

```bash
$ cd /tmp
$ git clone https://github.com/sharu725/gatok.git
$ cd gatok
```

7° - Uma vez dentro do diretório do template, execute o Jekyll:

```bash
$ jekyll server --baseurl "" --watch
```

![Imagem](https://github.com/Kingm0b/kingm0b.github.io/raw/master/images/2016-11-16/jekyll-site.png "Rodapé do site do Aurelio")

Obs: O jekyll suporta vários subcomandos. Veja o help dele:

```
$ jekyll --help
jekyll 3.3.0 -- Jekyll is a blog-aware, static site generator in Ruby

Usage:

  jekyll <subcommand> [options]

Options:
        -s, --source [DIR]  Source directory (defaults to ./)
        -d, --destination [DIR]  Destination directory (defaults to ./_site)
            --safe         Safe mode (defaults to false)
        -p, --plugins PLUGINS_DIR1[,PLUGINS_DIR2[,...]]  Plugins directory (defaults to ./_plugins)
            --layouts DIR  Layouts directory (defaults to ./_layouts)
            --profile      Generate a Liquid rendering profile
        -h, --help         Show this message
        -v, --version      Print the name and version
        -t, --trace        Show the full backtrace when an error occurs

Subcommands:
  docs                  
  import                
  build, b              Build your site
  clean                 Clean the site (removes site output and metadata file) without building.
  doctor, hyde          Search site and print specific deprecation warnings
  help                  Show the help message, optionally for a given subcommand.
  new                   Creates a new Jekyll site scaffold in PATH
  new-theme             Creates a new Jekyll theme scaffold
  serve, server, s      Serve your site locally
```

Dentre os subcomandos utilizados, usei o *server*. Para verificar as opções específicas do server:

```
$ jekyll server --help
jekyll serve -- Serve your site locally

Usage:

  jekyll serve [options]

Options:
            --config CONFIG_FILE[,CONFIG_FILE2,...]  Custom configuration file
        -d, --destination DESTINATION  The current folder will be generated into DESTINATION
        -s, --source SOURCE  Custom source directory
            --future       Publishes posts with a future date
            --limit_posts MAX_POSTS  Limits the number of posts to parse and publish
        -w, --[no-]watch   Watch for changes and rebuild
        -b, --baseurl URL  Serve the website from the given base URL
            --force_polling  Force watch to use polling
            --lsi          Use LSI for improved related posts
        -D, --drafts       Render posts in the _drafts folder
            --unpublished  Render posts that were marked as unpublished
        -q, --quiet        Silence output.
        -V, --verbose      Print verbose output.
        -I, --incremental  Enable incremental rebuild.
            --ssl-cert [CERT]  X.509 (SSL) certificate.
        -H, --host [HOST]  Host to bind to
        -o, --open-url     Launch your site in a browser
        -B, --detach       Run the server in the background
            --ssl-key [KEY]  X.509 (SSL) Private Key.
        -P, --port [PORT]  Port to listen on
            --show-dir-listing  Show a directory listing instead of loading your index file.
            --skip-initial-build  Skips the initial site build which occurs before the server is started.
        -h, --help         Show this message
        -v, --version      Print the name and version
        -t, --trace        Show the full backtrace when an error occurs
        -s, --source [DIR]  Source directory (defaults to ./)
        -d, --destination [DIR]  Destination directory (defaults to ./_site)
            --safe         Safe mode (defaults to false)
        -p, --plugins PLUGINS_DIR1[,PLUGINS_DIR2[,...]]  Plugins directory (defaults to ./_plugins)
            --layouts DIR  Layouts directory (defaults to ./_layouts)
            --profile      Generate a Liquid rendering profile
        -h, --help         Show this message
        -v, --version      Print the name and version
        -t, --trace        Show the full backtrace when an error occurs
```

De todas estas opções, utilizei <code>--baseurl ""</code>, para dizer ao Web Server builtin do Jekyll que a raiz do servidor é o diretório corrente, e <code>--watch</code>, para fazer com que o Jekyll monitore e tome decisões sobre toda alteração feita em seus arquivos (menos no _config.yml, explico agorinha).

Para postar algo, devo escrever um arquivo de extensão .md ou .markdown e colocá-lo no subdiretório <code>_posts</code>. O nome deste arquivo não pode ser qualquer coisa, ele **deve** ser a data do post em formato americano (YYYY-MM-DD) seguido do nome do futuro arquivo html gerado. Exemplo: 2016-28-12-hello-world.md

Duas observações: o Jekyll é bem flexível, tanto o formato da data no nome dos arquivos quanto o nome da extensão são ajustáveis (até mesmo o uso de markdown é ajustável). Estes ajustes são definidos no arquivo **_config.yml** (YML é o equivalente ao Json no universo Ruby rs).

Neste exemplo, se eu quiser postar algo novo terei que escrever um arquivo com, pelo menos, as seguintes linhas como cabeçalho:

```
---
layout: post
title: "M0blabs"
---

Olá! Esta é minha primeira publicação pelo Jekyll.

```

Salvarei o arquivo com a data de hoje: **2016-12-03-hello-world.md** no diretório _posts. Por causa do <code>--watch</code> o Jekyll automaticamente irá gerar a respectiva página e jogá-la no subdiretório em <code>_site</code>.

![Imagem](https://github.com/Kingm0b/kingm0b.github.io/raw/master/images/2016-11-16/hagura-m0blabs.png "Rodapé do site do Aurelio")
_Primeiro post da lista_


**Observação**: A data do arquivo não deverá ser no futuro. A data das publicações deverão ser da data corrente para trás.

No caso do template usado como exemplo, sua estrutura é definida como:

```
.
├── about.md
├── _config.yml
├── contact.md
├── css
├── images
├── _includes
├── index.html
├── js
├── _layouts
├── _posts
├── README.md
├── _sass
└── _site

```

Quase todos os templates para Jekyll giram em torno desta estrutura. O diretório <code>_posts</code> contém as publicações "brutas" do blog (normalmente em markdown, mas pode ser Latex se quiser), e o <code>_site</code> contém o blog propriamente dito gerado pelo Jekyll (quando você estava rodando o *jekyll server* seu browser estava acessando o conteúdo de _site).

Apenas como teste, experimente entrar no diretório <code>_site</code> e rodar o seguinte comando:

```bash
$ cd _site
$ python2 -m SimpleHTTPServer # para Arch Linux
$ python -m SimpleHTTPServer  # distros gerais
```

No seu browser acesse: http://127.0.0.1:8000

Toda a configuração do blog gira em torno do *_config.yml*. Experimente trocar o valor da variável <code>title</code> para alterar o título do seu blog:

![Imagem](https://github.com/Kingm0b/kingm0b.github.io/raw/master/images/2016-11-16/titulo-alterado.png "Alteração feita no _config.yml")

Toda alteração nos posts são aplicados automaticamente. Alterações no _config.yml exigem a reinicialização do Jekyll.

O legal é que existe dezenas de outros templates para o Jekyll (além, é claro, de você poder criar o seu próprio):

* [http://jekyllthemes.org](http://jekyllthemes.org)
* [https://jekyllthemes.io](https://jekyllthemes.io)
* [http://themes.jekyllrc.org](http://themes.jekyllrc.org)

O mais massa de tudo isto é que o backend do GitHub Pages faz uso do Jekyll como engine !!!

Isto quer dizer que eu não preciso atualizar localmente o blog e upar "manualmente" o _site no GitHub. É só fazer um fork do repositório de algum template, fazer os ajustes necessários e pronto! Você já tem um blog em Jekyll. Para novas publicações, basta upar seus posts em markdown no diretório _posts, automaticamente o "GitHub" irá atualizar o blog para você.

Bem, e aqui se encerra a saga "*em busca da hospedagem perfeita*": GitHub Pages + Jekyll + Jekyll Minimal Theme (com fontes grandes e serifadas). Não sou especialista em nada! Tenho como áreas de interesse redes, Linux/Unix e Segurança da Informação. Com o tempo irei compartilhar muita coisa aqui. Fiquem de olho!

(PS: Sou totalmente aberto a correções. Qualquer explicação equivocada da minha parte, favor entre em contato comigo para eu corrigir... Aprenderemos juntos)

EOF
