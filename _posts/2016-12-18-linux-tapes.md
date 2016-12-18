---
layout: post
title: "Manipulação de fitas no Linux"
---

Uma vez conectado o robô de fita na máquina virtual, o udev o reconhece como um dispositivo SCSI:

```bash
# lsscsi
[0:0:0:0]   disk     VMware   Virtual disk     1.0      /dev/sda
[0:0:1:0]   tape     IBM      ULTRIUM-HH4      C7R3     /dev/st0
[2:0:0:0]   cd/dvd   NECVMWar VMware IDE CDR10 1.00     /dev/sr0
```

Detalhes das informações retornadas:

```bash
# lsscsi [0:0:1:*]
[0:0:1:0] tape IBM ULTRIUM-HH4 C7R3 /dev/st0
^         ^    ^   ^           ^    ^
|         |    |   |           |    |_ Respectivo arquivo no devfs
|         |    |   |           |
|         |    |   |           |_ Firmware
|         |    |   |_ Equipamento
|         |    |
|         |    |_ Fabricante
|         |
|         |_ Tipo do dispositivo
|
|_ Endereço do dispositivo no barramento SCSI
```

O interessante é que dois pseudo-arquivos são criados para representar a MESMA unidade de fita:

```bash
# ls -l /dev/*st0
crw-rw---- 1 root tape 9, 128 Fev 24 17:00 /dev/nst0
crw-rw---- 1 root tape 9, 0   Fev 24 17:00 /dev/st0
```

Na listagem avançada, vemos que o dispositivo de fitas é um dispositivo de caractere, isto indica que os dados manipulados nas fitas (lidos/escritos) ocorrem um byte por vez (a uma velocidade de 120 MB/s - de acordo com a documentação do equipamento).

Além disto, observe que o minor number é diferente nos dois arquivos. De acordo com a documentação do kernel, o major number 9 indica um dispositivo SCSI para manipulação de fitas, e o minor number 0 (no caso do st0) se refere a primeira unidade SCSI de fita no modo "auto-rewind" (se existisse um segundo robô de fita, o minor number dele seria 2). Isto quer dizer que ao término da execução de cada operação sobre a fita (leitura ou escrita) o robô irá automaticamente rebobinar a mesma.

No dispositivo nst0 temos como minor number 128. Isto é um indicativo de que para qualquer operação na fita o driver não avisará a controladora SCSI do robô para rebobiná-la.

Escrevendo na fita

Como o kernel abstrai todos os detalhes com relação a comunicação das aplicações com a controladora da unidade de fita, podemos nos aproveitar do VFS para realizar operações simples com a fita.

Escrevendo a string "Kingm0b_" na fita:

```bash
# echo "Kingm0b_" > /dev/st0
```

Na prática serão escritos nove bytes na fita: os oito primeiros caracteres + \n (new line). Como o dispositivo que está sendo manipulado é o st0, ao término da escrita do último byte a fita será rebobinada.

Recuperando a mensagem

Partindo do princípio que a fita já se encontra rebobinada, podemos dizer a ferramenta dd que copie os 9 primeiros bytes do dispositivo nst0 e jogue sua saída para o "arquivo.txt":

```bash
# dd if=/dev/nst0 of=arquivo.txt bs=9 count=1
1+0 registros de entrada
1+0 registros de saída
9 bytes (9 B) copiados, 0,0506426 s, 0,2 kB/s

# cat arquivo.txt
Kingm0b_
# _
```

Como esta operação utilizou o dispositivo nst0, após a leitura do 9° byte o robô congelará a fita no início do primeiro bit do 10° byte da fita.

ATENÇÃO: Estas operações não são recomendadas, pois as ferramentas utilizadas (echo, printf, dd) não conseguem se comunicar adequadamente com o driver de fita, portanto, alguns imprevistos podem ocorrer. Recomenda-se a utilização de ferramentas que ofereçam suporte apropriado para este tipo de mídia (realizando chamadas ioctl convenientes).

Operações com o tar

O tar originalmente foi desenvolvido para realizar operações de backup em fita. Ele dispõe de um formato rudimentar para compactação de arquivos e diretórios. Cada arquivo compactado pelo tar, é chamado de tarball.

Obs: Compactação é diferente de compressão, apesar da maioria das ferramentas de compressão também compactarem os arquivos primeiro.

Basicamente, um tarball é composto de um cabeçalho contento os metadados do arquivo original (como: nome do arquivo, uid e gid, checksum, tamanho, data de modificação, por exemplo) seguido dos bytes do arquivo propriamente. Caso o tarball contenha mais de um arquivo (o que é muito comum) não há um índice ou algo do tipo que centralize e facilite a enumeração - listagem - de arquivos presentes no tarball.
Estrutura do tar

## Estrutura de um tarball

Para que a listagem dos arquivos seja possível, o tar deverá varrer todo o conteúdo do tarball lendo o início do cabeçalho de cada bloco de dados e nos informando os nomes dos arquivos na medida em que vão sendo encontrados.

A aplicação do formato tar em fitas magnéticas funciona muito bem, uma vez que facilita a manipulação de arquivos (compactação, descompactação e listagem) neste tipo de dispositivo de armazenamento sequencial. Vejamos algumas experiências:

Rebobinando a fita:

```bash
# mt -f /dev/nst0 rewind
```

Criando o tarball, adicionando o diretório "arquivos" na fita:

```bash
# tar -cvf /dev/st0 arquivos/
arquivos/
arquivos/passwd
arquivos/shadow
```

Acrescentando mais um arquivo na fita:

```bash
# tar --append -f /dev/st0 cpio_2.11+dfsg.orig.tar.xz
```

Listando o conteúdo do tarball:

```bash
# tar -tvf /dev/st0
drwxr-xr-x root/root       0 2016-02-24 17:46 arquivos/
-rw-r--r-- root/root       1434 2016-02-24 17:46 arquivos/passwd
-rw-r----- root/root       966 2016-02-24 17:46 arquivos/shadow
-rw-r--r-- root/root    802940 2012-12-30 00:41 cpio_2.11+dfsg.orig.tar.xz
```

Descompactando todo o tarball:

```bash
# tar -xvf /dev/st0
arquivos/
arquivos/passwd
arquivos/shadow
cpio_2.11+dfsg.orig.tar.xz
```

Descompactando arquivos específicos:

```bash
# tar -xvf /dev/st0 arquivos/passwd
arquivos/
arquivos/passwd
arquivos/shadow
cpio_2.11+dfsg.orig.tar.xz
```

Podemos realizar marcações no decorrer da fita para

