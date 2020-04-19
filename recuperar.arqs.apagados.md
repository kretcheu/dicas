# Recuperar arquivos apagados

Como recuperar arquivos apagados usando o **foremost**.

Conteúdo:

1. Softwares necessários e convenções usadas.
2. Instalação
3. Uso básico
4. Especificando outro diretório de destino
5. O arquivo de configuração
6. Adicionando suporte a outros tipos de arquivo
7. Conclusão

Nesse artigo vamos falar sobre o **foremost**, um programa livre forense muito útil, que é capaz de recuperar arquivos apagados usando um técnica chamada "data carving", algo como reconstrução de dados.
O utilitário foi originalmente desenvolvido pelo departamento de investigações especiais da força aérea dos Estados Unidos.
Pode recuperar vários tipos de arquivo e o suporte para tipos específicos podem ser adicionados pelo usuário através de um arquivo de configuração.
O programa funciona também usando imagens geradas pelo *dd* ou programas similares.

Nesse tutorial você irá aprender:

- Como instalar o foremost.
- Como usar para recuperar arquivos apagados.
- Como adicionar suporte a tipos específicos de arquivo.

Foremost é um programa forense de recuperação de dados usado para recuperar arquivos usando seus cabeçalhos e a estrutura de dados, num processo conhecido como "file carving" ou reconstrução de arquivos.

## 1. Softwares necessários e convenções usadas.

Para recuperar os arquivos apagados usaremos o programa **foremost**, disponível na maioria das distribuições GNU/Linux.\
Caso queira adicionar suporte a outros tipos de arquivo pode precisar de um visualizador hexdecimal como hexdump, xxd ou outro.
É necessário alguma familiaridade com linha de comandos, mas nada difícil.

## 2. Instalação

Para instalar no Debian e seus derivados basta usar o apt.\
Dependendo de suas configurações usando a conta do usuário *root* ou precedendo os comandos por *sudo*.
```
apt install foremost
```

## 3. Uso básico

**ALERTA**

```
Não importando qual ferramenta de recuperação ou processo que usará para recuperar seus arquivos,
antes de começar é recomendado que faça uma cópia de segurança (backup) binária,
evitando que sobrescreva seus dados acidentalmente.

Nesse caso você poderá tentar novamete recuperar seus arquivos mesmo depois de uma tentativa frustrada.
```

Consulte sobre o programa **dd** ou **ddrescue** para fazer o backup.

Exemplo:
```
dd if=/dev/sdb1 of=particao.img bs=1M
```

O utilitário foremost tenta recuperar e reconstruir os arquivos com base nos seus cabeçalhos e na estrutura de dados sem depender dos metadados da área de controle do sistema de arquivos.
Essa técnica forense é conhecida como "file carving".

O programa suporta vários tipos de arquivos como por exemplo:

(jpg, gif, png, bmp, avi, exe, mpg, wav, riff, wmv, mov, pdf, ole, doc, zip, rar, htm, cpp)

A forma mais básica de uso do foremost é fornecendo uma fonte de dados para procurar por arquivos apagados.\
Pode ser tanto uma partição ou uma imagem de disco gerada pelo dd.

Vamos ver um exemplo, imagine que queira procurar na partição /dev/sdb1:

Antes de começarmos uma coisa MUITO importante de lembrar é NUNCA armazenar os dados recuperados na mesma partição que está recuperando para evitar que os arquivos deletados ainda presentes no dispositivo sejam sobrescritos.

Devemos executar o comando:
```
foremost -i /dev/sdb1
```

Por padrão, o programa cria um diretório chamado *output* dentro do diretório corrente e usa esse diretório como destino.
Dentro desse diretório é criado um sub-diretório para cada tipo de arquivo suportado que estamos tentando recuperar.
Cada diretórios receberá os arquivos recuperados daquele tipo no processo de recuperação.

```
output
├── audit.txt
├── avi
├── bmp
├── dll
├── doc
├── docx
├── exe
├── gif
├── htm
├── jar
├── jpg
├── mbd
├── mov
├── mp4
├── mpg
├── ole
├── pdf
├── png
├── ppt
├── pptx
├── rar
├── rif
├── sdw
├── sx
├── sxc
├── sxi
├── sxw
├── vis
├── wav
├── wmv
├── xls
├── xlsx
└── zip
```

Quando o foremost termina, os diretórios vazios são apagados.\
Apenas os diretórios que contém arquivos são mantidos, assim é possível saber quais tipos de arquivos foram recuperados com sucesso.

Por padrão o programa tenta recuperar todos os tipos de arquivo suportados, para restringir sua procura você pode usar a opção `-t` e fornecer uma lista de tipos de arquivo que deseja recuperar, separados por vírgula. No exemplo abaixo restringimos a procurar apenas para arquivos gif e pdf:

```
foremost -t gif,pdf -i /dev/sdb1
```

## 4. Especificando outro diretório de destino

Como já dissemos, se o destino não for explicitamente declarado o foremost cria um diretório chamado *output* dentro do diretório corrente.\
E se quisermos especificar um diretório alternativo?

Tudo que devemos fazer é usar a opção `-o` e fornecer o nome do diretório.\
Se o diretório não existir, será criado, se existir e não estiver vazio o programa vai apresentar uma mensagem de erro:

```
ERROR: /home/kretcheu/output is not empty
 	Please specify another directory or run with -T.
```

Para resolver isso, como sugerido pelo programa, podemos tanto usar outro diretório como executar novamente o programa com a opção `-T`.
Se usarmos a opção `-T` o nome de diretório destino especificado na opção `-o` será acrescido de data e hora da recuperação.
Isso possibilita que o programa seja executado várias vezes com o mesmo destino.

No nosso caso o diretório de destino poderia ser:
```
foremost -i /dev/vdc2-o recuperados -T

/home/kretcheu/recuperados_Sat_Apr_18_13_49_21_2020
```

Após o término do processo que pode demorar bastante, digo BASTANTE mesmo, algo como 8 ou 9 horas para uma partição de 500 Gb, dependendo da velocidade de acesso aos dados.

Será criado nesse diretório de destino um arquivo chamado `audit.txt` que conterá um relatório da recuperação.

Nesse exemplo:
```
cat audit.txt

Foremost version 1.5.7 by Jesse Kornblum, Kris Kendall, and Nick Mikus
Audit File

Foremost started at Sat Apr 18 18:12:18 2020
Invocation: foremost -i /dev/vdc2 -o recuperados -T
Output directory: /home/kretcheu/recuperados_Sat_Apr_18_18_12_18_2020
Configuration file: /etc/foremost.conf
------------------------------------------------------------------
File: /dev/vdc2
Start: Sat Apr 18 18:12:18 2020
Length: 17 MB (18857472 bytes)

Num	 Name (bs=512)	       Size	 File Offset	 Comment

0:	00005568.png 	      41 KB 	    2850816 	  (1920 x 1080)
1:	00005784.png 	      64 KB 	    2961408 	  (1920 x 1080)
Finish: Sat Apr 18 18:12:18 2020

2 FILES EXTRACTED

png:= 2
------------------------------------------------------------------

Foremost finished at Sat Apr 18 18:12:18 2020

```

## 5. O arquivo de configuração

O arquivo de configuração do foremost pode ser usado para especificar formatos de arquivos que não são suportados nativamente pelo programa.
Dentro do arquivo podemos encontrar vários exemplos comentados, mostrando a sintaxe que deverá ser usada para cumprir essa tarefa.

Aqui um exemplo envolvendo o tipo de arquivo png, as linhas estão comentadas uma vez que o tipo é suportado por padrão.
```
# PNG   (used in web pages)
#	(NOTE THIS FORMAT HAS A BUILTIN EXTRACTION FUNCTION)
#  	png	y	200000	\x50\x4e\x47?	\xff\xfc\xfd\xfe
```

A informação a ser fornecida para adicionar suporte a um tipo de arquivo é composta do 5 campos separados por um caracter TAB:

Campo | Descrição
-- | --
png | O tipo de arquivo.
y  | Tanto cabeçalho como rodapé são "case sensitive".
200000 | Tamanho máximo do arquivo em bytes.
\x50\x4e\x47? | O cabeçalho
\xff\xfc\xfd\xfe | O rodapé.

Apenas o último é opcional e pode ser omitido.

Se o caminho do arquivo de configuração não for explicitamente fornecido com a opção `-c` e existir um arquivo chamado `foremost.conf` no diretório corrente este será usado.\
Caso não seja encontrado então o arquivo de configuração padrão `/etc/foremost.conf` será usado.

## 6. Adicioando suporte a outros tipos de arquivo

Lendo o exemplo fornecido no arquivo de configuração, nós podemos facilmente incluir suporte a um novo tipo de arquivo.
Nesse exemplo vamos adicionar suporte aos arquivos de audio flac.
Flac (Free Lossless Audio Coded) é um formato de audio não proprietário que é capaz de fornecer audio comprimido sem perda de qualidade.

Primeiro, sabemos que o cabeçalho desse tipo de arquivo na forma hexadecimal é: 66 4C 61 43 00 00 00 22 (fLaC em ASCII)
Podemos verificar isso usando um programa como *hexdump* num arquivo flac:

```
hexdump -C Adrian_Lester_bbc_radio4_front_row_25_04_2013.flac |head

00000000  66 4c 61 43 00 00 00 22  10 00 10 00 00 22 71 00  |fLaC..."....."q.|
00000010  45 3b 0b b8 03 70 00 16  89 8f 84 2b e7 2a bc fc  |E;...p.....+.*..|
00000020  3f de e0 eb c0 c8 ef 00  63 07 84 00 01 f7 20 00  |?.......c..... .|
00000030  00 00 72 65 66 65 72 65  6e 63 65 20 6c 69 62 46  |..reference libF|
00000040  4c 41 43 20 31 2e 32 2e  31 20 32 30 30 37 30 39  |LAC 1.2.1 200709|
00000050  31 37 02 00 00 00 15 01  00 00 44 65 73 63 72 69  |17........Descri|
00000060  70 74 69 6f 6e 3d 54 68  69 73 20 66 69 6c 65 20  |ption=This file |
00000070  77 61 73 20 63 72 65 61  74 65 64 20 61 73 20 70  |was created as p|
00000080  61 72 74 20 6f 66 20 74  68 65 20 4a 61 6e 75 61  |art of the Janua|
00000090  72 79 20 32 30 31 34 20  42 42 43 2f 57 69 6b 69  |ry 2014 BBC/Wiki|
```

Como você pode ver, a assinatura do arquivo é de fato o que esperávamos.

Aqui vamos assumir o tamanho máximo do arquivo de 30 Mb ou 30000000 bytes.

Vamos adicionar uma entrada no arquivo de configuração.

```
flac    y       30000000    \x66\x4c\x61\x43\x00\x00\x00\x22
```

A assinatura do rodapé é opcional e aqui não vamos fornece-la.
Agora o programa deve ser capaz de recuperar arquivos flac apagados.

Vamos verificar:

Para testar que tudo está funcionando como esperado, coloquei e apaguei um arquivo flac na partição /dev/vdc2 e então vou rodar o comando:

```
foremost -i /dev/vdc2 -o recuperados -T
```

Vamos verificar
```
tree recuperados_Sat_Apr_18_18_32_58_2020/

recuperados_Sat_Apr_18_18_32_58_2020/
├── audit.txt
└── flac
    └── 00008356.flac

1 directory, 2 files
```

Como esperado, o programa foi capaz de recuperar o arquivo flac apagado.
No entanto, o nome do arquivo está diferente.\
O nome original do arquivo não pode ser recuperado porque como sabemos, os metadados dos arquivos estão contidos no sistema de arquivos e não no arquivo em si.

O arquivo audit.txt contém o relatório da recuperação, nesse caso:

```
Foremost version 1.5.7 by Jesse Kornblum, Kris Kendall, and Nick Mikus
Audit File

Foremost started at Sat Apr 18 18:48:23 2020
Invocation: foremost -i /dev/vdc2 -o recuperados -T
Output directory: /home/kretcheu/recuperados_Sat_Apr_18_18_48_23_2020
Configuration file: /etc/foremost.conf
------------------------------------------------------------------
File: /dev/vdc2
Start: Sat Apr 18 18:48:23 2020
Length: 20 MB (20971520 bytes)

Num	 Name (bs=512)	       Size	 File Offset	 Comment

0:	00012452.flac 	      13 MB 	    6375424
Finish: Sat Apr 18 18:48:23 2020

1 FILES EXTRACTED

flac:= 1
------------------------------------------------------------------

Foremost finished at Sat Apr 18 18:48:23 2020
```

## 7. Conclusão

Nesse artigo você aprendeu como usar o foremost, um programa forense que é capaz de recuperar arquivos apagados de vários tipos.
Você aprendeu que o programa usa uma técnica chamada "data carving" que depende das assinaturas dos arquivos para alcançar seu objetivo.

É importante ressaltar que o trabalho do foremost pode ser feito para recuperar arquivos de muitos sistemas de arquvos diferentes, independente de qual sistema operacional ele é usado.

Vimos um exemplo de uso do programa e também como adicionar suporte a um tipo específico de arquivo usando a sintaxe ilustrada no arquivo de configiração.
Para mais informações sobre o uso do programa, consulte a página do manual.
```
man foremost

foremost ⁻h
```

Esse artigo foi traduzido por Paulo Kretcheu @kretcheu e recebeu algumas adpatações.

Qualquer dúvida me procure no Telegram!

- [Grupo Debian Brasil](https://t.me/debianbrasil)
- [Grupo Debian BR](https://t.me/debianbr)

Artigo traduzido e adaptado.\
Original escrito por Egidio Docile.\
https://linuxconfig.org/how-to-recover-deleted-files-with-foremost-on-linux

