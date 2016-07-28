---
title: Tratamento e Análise de Bases de Dados Disponibilizadas pela Prefeitura
        de São Paulo
author: Diego Rabatone Oliveira (@diraol) - diraol@diraol.eng.br
date: "1º sem / 2016"
output: github_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

```{r bibliotecas, echo=FALSE, message=FALSE, results='hide', render=FALSE, warning=FALSE}
  # Carregando bibliotecas que serão utilizadas
  library(dplyr)
  library(data.table) # Biblioteca utilizada para leitura de arquivos
  library(png)
  library(grid)
  library(ggplot2)
```

```{r configGGPLOT, echo=FALSE}
  # Setando tema default do ggplot como sendo o theme_light
  theme_set(theme_light())
  
  # Alterando algumas características:
  theme_update(
    plot.title = element_text(lineheight=1.6,
                              face="bold",
                              size="16"),
    axis.title = element_text(color="#333333",
                              size="12"),
    axis.text = element_text(color="#333333",
                             size="10"),
    axis.text.y = element_text(margin=margin(l=6)),
  
    legend.title = element_text(color="#333333",
                                size="10"),
    legend.text = element_text(color="#333333",
                               size="8"),
    # Para facets
    strip.text = element_text(color="#333333",
                              size="8",
                              face="bold"),
    strip.background = element_rect(fill="#999999", colour = "#999999")
  )

```

Este documento é o guia de referência para a aula sobre ***Tratamento e análise
de Bases de Dados Disponibilizadas pela Prefeitura de São Paulo*** oferecida no
contexto do curso de formação dos auditores aprovados em concurso recente da
Prefeitura de São Paulo (PMSP). O objetivo desta aula é apresentar um *case* de
trabalho com alguns conjuntos de dados fornecidos pela PMSP, apresentando não
apenas o fluxo de trabalho, mas também desafios correntes de serme enfrentados
e algumas técnicas e tecnologias que podem contribuir para um melhor desempenho
na execução das atividades da carreira.

A PMSP disponibiliza hoje um conjunto considerável de bases de dados, apesar de
algumas não estarem disponíveis em formato aberto. Para este trabalho, optamos
por escolher uma base em formato aberto que apresente alguns desafios comuns de
serem encontrados.

Escolheu-se trabalhar com as Folhas de Pagamento das autarquias municipais.
Estas bases também poderiam ser cruzadas com outras bases como, por exemplo, a
do IPTU - que só está disponível para consultas pontuais no site
GeoSampa[^1], mas,
infelizmente, não está disponível para *download*. Além disso, supondo que os
auditores tenham acesso à base de declarações de imposto de renda dos
funcionários, seria possível realizar uma análise comparativa entre evolução
patrimonial e salário, por exemplo.

[^1]: http://geosampa.prefeitura.sp.gov.br

Vale destacar também que as bases publicadas não contêm o CPF dos servidores,
o que dificulta certos cruzamentos entre bases, visto que o tratamento por "nome"
pode incorrer em problemas de homônimos ou de erros de digitação.

# Ambiente
Este relatório foi produzido numa distribuição GNU/Linux
\href{http://debian.org}{Debian} 8.3 64bits, utilizando o
software estatístico
\href{https://www.r-project.org}{R} (versão 3.2.3), com os
recursos de
\href{http://rmarkdown.rstudio.com}{Rmarkdown}, utilizando
como IDE (*Integrated Development Environment* ou
**Ambiente de Desenvolvimento Integrado**) a aplicação
\href{http://rstudio.com}{Rstudio} (versão 0.99.878). Ainda
foram utilizados o 
\href{http://pt-br.libreoffice.org/descubra/calc/}{LibreOffice Calc}
(versão 5.0.4.2),

# Passo a passo
A seguir serão apresentados os passos realizados para a análise presente neste
documento. Tanto os códigos para geração deste documento quanto os arquivos
utilizados nas análises podem ser encontrados em:

\url{https://github.com/diraol/AulaAnaliseDeDados}

## Passo 01 - *Download* das folhas de pagamento
Os *downloads* foram realizados do
Portal de Dados Abertos da PMSP[^2]
em 18/02/2016.

[^2]: http://dados.prefeitura.sp.gov.br

No portal de Dados Abertos foram buscados os registros de "Folha de Pagamentos"
dentro do grupo "Orçamento e Gestão". Em seguida os arquivos no formato CSV[^3]
foram baixados um a um e salvos na pasta *dados* para serem analisados.

[^3]: *Comma-separated values* ou **Valores Separados Por Vírgula** - https://pt.wikipedia.org/wiki/Comma-separated_values

## Passo 02 - Codificações dos arquivos
Um dos primeiros, e mais comuns, problemas com o qual nos deparamos quando
estamos trabalhando com dados é a codificação[^4] de caracteres dos arquivos.
Idealmente, todos arquivos deveriam usar um mesmo padrão, sendo que por motivos
de interoperabilidade[^5] recomenda-se utilizar a codificação UTF-8[^6]
(a codificação padrão utilizada no Sistema Operacional Windows é a
*ISO-8859-1*[^7], não recomendado).

[^4]: https://pt.wikipedia.org/wiki/Codifica%C3%A7%C3%A3o_de_caracteres
[^5]: http://htmlpurifier.org/docs/enduser-utf8.html
[^6]: https://en.wikipedia.org/wiki/UTF-8
[^7]: https://en.wikipedia.org/wiki/ISO/IEC_8859-1

A importância de se saber qual é a codificação que está sendo utilizada e utilizar
uma mesma codificação para todos os arquivos reside no fato de que se a leitura
dos dados não for realizada na codificação correta, teremos problemas com a
acentuação das palavras, o que dificulta a leitura, pode impedir a compatibilização
de diversos arquivos e dificulta o tratamento e a análise dos dados.

Assim, o primeiro passo é verificar qual é a codificação dos arquivos que foram
baixados antes de qualquer outro processamento. Para realizar tal verificação,
com uma quantidade reduzida de arquivos, podemos fazê-la utilizando de
planilhas eletrônicas, como o LibreOffice Calc.

Mas se temos muitos arquivos para verificar, é mais fácil/prático utilizar
ferramentas de linha de comando que processam os arquivos em lote. Por exemplo,
podemos utilizar o comando **file**[^8] presente em todas as distribuições
GNU/Linux, passando como parâmetro a expressão regular que identifica todos os
arquivos que terminam em *.csv* e estão dentro da pasta dados.

[^8]: http://pubs.opengroup.org/onlinepubs/9699919799/utilities/file.html

```{r fileEncodings, eval=FALSE}
  file dados/*.csv
```
```{r fileEncodingsResults, results='hold', echo=FALSE, cache=TRUE, cache.rebuild=FALSE}
  cat(system('file dados/*.csv', intern = TRUE), sep='\n')
```

O comando *file* nos retorna duas informações para cada arquivo:

1. a codificação utilizada; e
2. o caractere especial que indica *quebra de linha*[^9] do arquivo.

[^9]: Leia mais sobre caracteres de quebra de linha em: http://stackoverflow.com/a/1552775

Conforme pode ser observado, os arquivos apresentam duas codificações diferentes:

   * *Non-ISO extended-ASCII text*
   * *ISO-8859 text*
\newpage
O comando *file* conseguiu identificar uma das codificações (ISO-8859-1), mas
não as duas. Assim, tentaremos utilizar um método um pouco mais manual para
identificar o segundo formato.
Para tanto, tentamos abrir o arquivo *ahmsp.csv* no *Calc*. Ao tentar abri-lo,
será apresentada a tela de importação, conforme a Figura \ref{fig:fig1}.

\begin{figure}[h]
  \centering
  \caption{Início do processo de importação de arquivo CSV\label{fig:fig1}}
  \includegraphics[width=0.85\textwidth]{./imgs/fig1.png}
\end{figure}

Podemos notar que no quadro que apresenta uma amostra dos dados temos diversos
caracteres desconhecidos. Isto se deve ao fato de a codificação selecionada (campo
*Conjunto de caracteres*) não estar condizente com a codificação usada na geração
do arquivo. Assim, para descobrir a codificação correta iremos trocar a opção do
campo *Conjunto de caracteres*' e, simultaneamente, observar os dados no quadro
na parte inferior da janela, até que todos os caracteres apareçam corretamente.
Assim, descobrimos que a codificação destes "Non-ISO" arquivos é a
"*Europa Ocidental (DOS/OS2-850/Internacional)*".

Para realizar a conversão do arquivo *ahmsp.csv* abriremos o arquivo no *Calc* e,
em seguida, iremos salvá-lo com a nova codificação. Agora que já descobrimos a
codificação do arquivo, precisamos indicar qual é o "separador" utilizado. O
padrão do formato *csv* é utilizar a vírgula. Mas, como no Brasil nós utilizamos
a vírgula como separador de decimais, é muito comum encontrar o caractere 
"ponto-e-vírgula" ("**;**"") como separador.
E este é o caso do presente exemplo. Ao alterar a opção de separador para
"ponto-e-vírgula", e com a codificação correta escolhida, podemos perceber que o
software passa a identificar os diversos campos do arquivo e com todas as
acentuações corretas, conforme pode ser visto na Figura \ref{fig:fig2}.
Agora basta clicar em "OK".

\begin{figure}[h!]
  \centering
  \caption{Selecionando opções para importação de arquivo CSV\label{fig:fig2}}
  \includegraphics[width=0.83\textwidth]{./imgs/fig2.png}
\end{figure}

Antes de salvar o arquivo no novo formato, podemos observar que a primeira linha
do mesmo contém uma descrição sobre o que se refere o arquivo
"*Autarquia Hospitalar Municipal - AHM 10/2015*". Esta primeira linha, antes da
linha de cabeçalho com o nome das colunas, pode atrapalhar quando tentamos
importar o CSV em oturos softwares. Assim, é recomendado remover esta linha e
deixar como primeira linha a linha que contém o nome de cada uma das colunas.

Após remover esta primeira linha descritiva, vamos agora salvar o arquivo.

Seguiremos pelo menu "*Arquivo*" > "*Salvar como*", que abrirá a tela para salvar
arquivos, ver Figura \ref{fig:fig3}:

\begin{figure}[h!]
  \centering
  \caption{Salvando arquivo CSV\label{fig:fig3}}
  \includegraphics[width=0.77\textwidth]{./imgs/fig3.png}
\end{figure}

Nesta tela, iremos deixar marcada a opção "*Editar definições do filtro*", que se
encontra do lado inferior esquerdo da tela, e no campo em que consta
"*Todos os formatos*", selecionaremos a opção "*Texto CSV (.csv)*". Ao clicar em
*Salvar*, será perguntado se você deseja substir o arquivo existente[^10]. Confirmando,
será perguntado se você deseja realmente salvar no formato CSV. Ao confirmar 
novamente, aparecerão as opções para salvar o arquivo. Nela podemos definir a
codificação, o caractere delimitador de campo e outras informações.

[^10]: É sempre uma boa prática guardar a versão original dos arquivos que serão
trabalhado, então o que pode ser feito é criar uma pasta na qual você irá salvar
uma cópia da versão original de todos os arquivos.

No caso presente, utilizaremos como "*Conjunto de caracteres*" **Unicode (UTF-8)**,
como "*Delimitador de campo*" utilizaremos o "ponto-e-vírgula" (**;**), no campo
"*Delimitador de texto*" utilizaremos aspas duplas (**\"**)[^11], e marcaremos também
a opção *Aspas em todas as células de texto*, vide a Figura \ref{fig:fig4}.

[^11]: Utilizaremos aspas duplas pois é comum encontrar o uso de aspas simples em
nomes, como, por exemplo, *Joana D'Arc*, o que pode atrapalhar na importação.

\begin{figure}[h!]
  \centering
  \caption{Tela de opções para exportação de arquivo CSV\label{fig:fig4}}
  \includegraphics[width=0.6\textwidth]{./imgs/fig4.png}
\end{figure}

Com este procedimento, será necessário recodificar cada um dos arquivos
individualmente, abrindo cada um dos arquivos e salvando na nova codificação.
Como teremos que abrir todos os arquivos, aproveitaremos para já verificar e
garantir que todos os arquivos possuam as mesmas colunas.

Observando alguns dos arquivos, podemos perceber que as colunas comuns são:
'*NOME*', '*CARGO*', '*LOTACAO*', '*ADMISSAO*', '*NASCIMENTO*', '*VENCIMENTOS*',
'*ENCARGOS*', '*BENEFÍCIOS*', '*OUTRAS*', '*VÍNCULO*', '*LIMINAR*'.

Para o caso de haverem poucos arquivos, esta é uma opção bem prática. Porém, se
houverem muitos arquivos (dezenas, centenas) se torna uma tarefa quase
impraticável desa forma.

Então, uma alternativa possível seria a utilização de ferramentas de linha de
comando que permitem a execução desta tarefa de conversão em lote. No caso, o
comando recomendado seria o **iconv**[^12], e um exemplo de como seria a
utilização do comando para converter um arquivo do formato  *ISO-8859-1* para
*UTF-8* seria:

[^12]: https://en.wikipedia.org/wiki/Iconv

```{r, eval=FALSE}
  iconv -f 'ISO-8859-1' -t 'UTF-8' arquivo_entrada.csv > arquivo_saida.csv
```

Vale destacar que, neste caso (**iconv**), aquela primeira linha que não contém
os cabeçalhos não será removida. Isso deverá ser tratado no momento da leitura
do arquivo.

## Passo 03 - Lendo e unificando os arquivos
Após padronizadas as codificações e formatos dos arquivos, vamos carregá-los no
R e unificá-los.

Primeiro vamos carregar cada um dos arquivos numa variável diferente. Vamos
utilizar o comando *read.csv2* pois ele assume como padrão de separador de
campos o caractere ponto-e-vírgula ("**;**") e separador de decimais a vírgula
("**,**"), enquanto o comando *read.csv* utiliza como padrões a vírgula e o ponto,
respectivamente. Passamos também o parâmetro *stringAsFactors=FALSE* para forçar
que a importação dos dados textuais seja feita como texto, e não como fatores
(*factors*)[^13]:

[^13]: Fator é o tipo de dado que é utilizado quando a variável possui categorias
pré-definidas e todos os dados devem se enquadrar nestas categorias. Os **cargos**
se enquadram nesta definição, visto que temos uma lista finita e conhecida de
cargos nas quais todos os funcionários devem se enquadrar.

```{r}
  ahmsp <- read.csv2("dados/ahmsp.csv", stringsAsFactors = FALSE)
  amlurb <- read.csv2("dados/amlurb.csv", stringsAsFactors = FALSE)
  cet <- read.csv2("dados/cet.csv", stringsAsFactors = FALSE)
  cohab <- read.csv2("dados/cohab.csv", stringsAsFactors = FALSE)
  fundatec <- read.csv2("dados/fundatec.csv", stringsAsFactors = FALSE)
  hspm <- read.csv2("dados/hspm.csv", stringsAsFactors = FALSE)
  iprem <- read.csv2("dados/iprem.csv", stringsAsFactors = FALSE)
  prodam <- read.csv2("dados/prodam.csv", stringsAsFactors = FALSE)
  sfmsp <- read.csv2("dados/sfmsp.csv", stringsAsFactors = FALSE)
  spcine <- read.csv2("dados/spcine.csv", stringsAsFactors = FALSE)
  spda <- read.csv2("dados/spda.csv", stringsAsFactors = FALSE)
  spnegocios <- read.csv2("dados/spnegocios.csv", stringsAsFactors = FALSE)
  spobras <- read.csv2("dados/spobras.csv", stringsAsFactors = FALSE)
  spsec <- read.csv2("dados/spsec.csv", stringsAsFactors = FALSE)
  sptrans <- read.csv2("dados/sptrans.csv", stringsAsFactors = FALSE)
  spturis <- read.csv2("dados/spturis.csv", stringsAsFactors = FALSE)
  spurbanismo <- read.csv2("dados/spurbanismo.csv", stringsAsFactors = FALSE)
  tmsp <- read.csv2("dados/tmsp.csv", stringsAsFactors = FALSE)
```

Nas bases escolhidas, cada registro (linha) representa uma pessoa ou pagamento.
É interessante não perdermos a informação de qual é a autarquia (base) de origem
de cada pessoa. Assim, antes de unificarmos todas as bases, adicionaremos uma
nova coluna (*ORIGEM*) com o nome da autarquia em questão:

```{r}
  ahmsp$ORIGEM <- 'ahmsp'
  amlurb$ORIGEM <- 'amlurb'
  cet$ORIGEM <- 'cet'
  cohab$ORIGEM <- 'cohab'
  fundatec$ORIGEM <- 'fundatec'
  hspm$ORIGEM <- 'hspm'
  iprem$ORIGEM <- 'iprem'
  prodam$ORIGEM <- 'prodam'
  sfmsp$ORIGEM <- 'sfmsp'
  spcine$ORIGEM <- 'spcine'
  spda$ORIGEM <- 'spda'
  spnegocios$ORIGEM <- 'spnegocios'
  spobras$ORIGEM <- 'spobras'
  spsec$ORIGEM <- 'spsec'
  sptrans$ORIGEM <- 'sptrans'
  spturis$ORIGEM <- 'spturis'
  spurbanismo$ORIGEM <- 'spurbanismo'
  tmsp$ORIGEM <- 'tmsp'
```

E agora unificaremos todas as bases numa variável chamada *dados*, 'empilhando'
as bases que temos uma abaixo da outra, utilizando o comando *rbind*:
```{r, tidy=TRUE}
  # Unificando as bases numa única variável:
  dados <- rbind(  ahmsp, amlurb, cet, cohab, fundatec, hspm, iprem, prodam,
                   sfmsp, spcine, spda, spnegocios, spobras, spsec, sptrans,
                   spturis, spurbanismo, tmsp )
  
  # Verificando o resultado final:
  str(dados)
```

Como podemos observar, temos 26689 observações (registros) e 12 variáveis, sendo
que 4 delas são numéricas e 8 são textuais. Mas, destas textuais duas são campos
de data e 4 são campos que podemos definir como categóricos, nos quais temos
categorias fixas. Precisamos então informar estes detalhes:
```{r}
  # Definindo os campos de Data, passando qual é o formato de dada a ser lido
  dados$ADMISSAO <- as.Date(dados$ADMISSAO, '%d/%m/%Y')
  dados$NASCIMENTO <- as.Date(dados$NASCIMENTO, '%d/%m/%Y')
  # Definindo os campos de categoria:
  dados$CARGO <- as.factor(dados$CARGO)
  dados$LOTACAO <- as.factor(dados$LOTACAO)
  dados$VÍNCULO <- as.factor(dados$VÍNCULO)
  dados$ORIGEM <- as.factor(dados$ORIGEM)
```

E verificando novamente nossos dados:
```{r}
  str(dados)
```

É possível observar que no campo *CARGO* tempos 944 "níveis" e no campo *LOTAÇÃO*
temos 926 níveis. Dificilmente temos tantos cargos/níveis distintos, ou, pelo menos,
para uma primeira análise é recomendável realizarmos um agrupamento com menos
categorias. Assim, vamos fazer uma investigação sobre quais são estes níveis,
mostrando os 10 primeiros e os 10 últimos níveis:
```{r}
  head(levels(dados$CARGO), n=10)
  tail(levels(dados$CARGO), n=10)
```

Como podemos observar, por falta de uniformidade e padronização nas informações,
devemos ter muitas ocorrênicas que poderiam ser agrupadas. Essa diferenças se dão
em abreviações, acentuação, etc. Assim, para unificar estes dados eles serão
exportados num arquivo externo e tratados à parte:
```{r}
  # salvando os cargos existentes no arquivo 'cargos.csv'
  write.csv(levels(dados$CARGO), 'cargos.csv')
  # lendo os 'novos cargos' de volta
  cargos <- read.csv2('cargos_resumidos.csv')
  str(cargos)
```

Agora vamos criar uma nova coluna em *dados* com a categorização agregada dos
cargos que criamos:
```{r}
  # criando uma nova coluna (CARGOS_RESUMIDOS)
  dados <- merge(dados, cargos, by='CARGO', all.x = TRUE)
```

## Análises exploratórias

### Primeiras Análises

Agora vamos fazer nossas primeiras análises dos dados presentes nas bases que
coletamos e tratamos.

Existem muitas formas de fazer tais análises, mas, em geral, as formas gráficas
são as mais práticas para uma primeira visão geral da situação.

Nós utilizaremos a biblioteca *ggplot2* para fazer algumas análises iniciais.
Muitos são os modelos de gráficos que podemos utilizar, e suas utilidades variam
bastante de acordo com o tipo de dados que estamos observando. No caso dos salários,
que são dados numéricos contínuos, uma boa maneira de começar a análise é observando
a distribuição geral de VENCIMENTOS:
```{r, fig.height=3, fig.cap='Distribuição dos Salários'}
ggplot(dados) +
  geom_density(aes(VENCIMENTOS, fill='VENCIMENTOS'), alpha=0.4, show.legend = FALSE) +
  scale_y_continuous('Frequência relativa', labels=scales::percent)
```

Do gráfico acima podemos perceber que os valores dos vencimentos vão além de
R$200.000,00. Daí já cabe investigarmos valores tão altos. Assim, vamos observar
valores de vencimento acima de R$30.000,00

```{r}
  # Filtrando os registros com VENCIMENTOS acima de R$30.000,00
  registros30k <- filter(dados, VENCIMENTOS>30000)
```

O número total de registros com vencimentos acima de R$30.000,00 é de: `r nrow(registros30k)`.

Como temos poucos registros, vamos análisá-los, selecionando alguns campos:
```{r, tidy=TRUE, size='tiny'}
  select(registros30k, ORIGEM, CARGO.RESUMIDO, VENCIMENTOS, ENCARGOS, BENEFÍCIOS)
```

E vamos salvar esses registros num arquivo à parte para que cada um desses casos
possa ser averiguado individualmente.

Seguindo com a análise dos demais dados:
```{r}
  # Salvando os registros com vencimentos de mais de 30k 
  write.table(registros30k, 'salarios_acima_de_30_mil.csv', row.names = FALSE, sep=';')
  # removendo os registros que já foram separados
  dados <- filter(dados, VENCIMENTOS<30000)
```

Gerando novamente o gráfico:
```{r, fig.height=3, fig.cap='Distribuição dos Salários'}
ggplot(dados) +
  geom_density(aes(VENCIMENTOS, fill='VENCIMENTOS'), alpha=0.4, show.legend = FALSE) +
  scale_y_continuous('Frequência relativa', labels=scales::percent)
```

Vamos para uma segunda análise, com base nos cargos ocupados, seguindo a nossa
agregação. Para esta análise, utilizaremos um estilo de gráfico chamado
**boxplot**:

```{r, echo=FALSE, fig.height=2.5}
ggplot(filter(dados, CARGO.RESUMIDO=="MOTORISTA" | CARGO.RESUMIDO=="COPEIRO"),
       aes(x = CARGO.RESUMIDO, y = VENCIMENTOS, colour = CARGO.RESUMIDO ) ) +
  geom_boxplot(alpha=0.5, show.legend = F) + xlab('Cargo') + coord_flip()
```

```{r}
q1 <- quantile(filter(dados, CARGO.RESUMIDO=="MOTORISTA")$VENCIMENTOS, 1/4)
q3 <- quantile(filter(dados, CARGO.RESUMIDO=="MOTORISTA")$VENCIMENTOS, 3/4)
media <- mean(filter(dados, CARGO.RESUMIDO=="MOTORISTA")$VENCIMENTOS)
mediana <- median(filter(dados, CARGO.RESUMIDO=="MOTORISTA")$VENCIMENTOS)
cat(q1, q3, media, mediana)
```

Agora vamos ao gráfico com todos os registros, de todos os cargos:

```{r, fig.height=10}
ggplot(dados, aes(x = CARGO.RESUMIDO, y = VENCIMENTOS, colour = CARGO.RESUMIDO ) ) +
  geom_boxplot(alpha=0.5, show.legend = F) + xlab('Cargo') + coord_flip() +
  theme(axis.text.y = element_text(size='8'))
```

O padrão no mundo científico é tratar os conjuntos de dados descartando os 
*outliers*, mas no presente contexto (Auditores de Controle Interno), a tendência
é buscar justamente os elementos que fogem ao padrão e às regras.
Assim, para um primeiro passo investigativo, buscaríamos avaliar cada uma das
categorias plotadas (*CARGO.RESUMIDO*) e investigar os *outliers* de cada uma delas.

Além disso, nós só avaliamos os *VENCIMENTOS*, e não tratamos dos *ENCARGOS* ou
dos *BENEFÍCIOS*, que podem render uma outra sorte de investigações.

### Tratando nomes
Iremos agora buscar pessoas que se encontram em mais de uma organização/origem.
Isso poderia caracterizar pessoas que recebem mais de um salário, o que pode vir
a configurar irregularidade. Claro que a busca por nomes contém várias limitações,
como, por exemplo, existência de homônimos, erros de digitação, etc.

Assim, iremos agora buscar nomes que aparecem mais de uma vez:

```{r}
nomes.pessoas.repetidas <- dados %>% group_by(NOME) %>%
                                      summarise(repeticoes = n()) %>%
                                      as.data.frame() %>%
                                      filter(repeticoes>1)
```

Agora vamos recuperar os registros completos das pessoas identificadas anteriormente

```{r}
registros.selecionados <- dados %>%
                      filter(NOME %in% nomes.pessoas.repetidas$NOME) %>%
                      arrange(NOME) # Ordenando pelo NOME
```

Vejamos os primeiros registros, e salvemo-os num arquivo externo:
```{r, tidy=TRUE, size='tiny'}
head(select(registros.selecionados, NOME, CARGO),n=15)
write.table(registros.selecionados, 'pessoas_duplicadas.csv', sep=';', dec=',', row.names=FALSE)
```


## Próximos passos
Os passos acima descritos servem como um exemplo de caso de uso de como utilizar
análise de dados para averiguar informações sob a ótica de auditoria.
Foram utilizadas técnicas exploratórias básicas, que, partindo de uma grande
quantidade de dados, indicam alguns caminhos a serem melhor investigados.
Além disso, também vale destacar que existem diversas outras técnicas, muito mais
elaboradas e complexas, como a utilização de técnicas estatísticas (Regressões,
Análise Fatorial, Clusterização, etc) ou técnicas específicas de "*Ciência de Dados*",
como *Machine Learning*, que podem nos apresentar outros resultados com grau de
aprofundamento muito maior. Mas, independente de técnicas e tecnologias,em algum
momento neste processo é necessária a análise crítica humana de olhar para os dados
individualizados e extrair deles informações, realizar verificações (por vezes
utilizando o telefone, ou mesmo visitando locais), averiguar legislações, etc.

Hoje existe uma grande diversidade de ambientes *online* nos quais se pode
aprender tais técnicas, das mais básicas às mais avançadas, até mesmo de forma
gratuita.

Outro ponto importante a ser destacado é a importância de buscar sempre utilizar
padrões quando se for elaborar um conjunto de dados, partindo do princípio de que
o conjunto de dados provavelmente será utilizado por alguém e que ele deve ser
possível de ser vinculado a outros conjuntos de dados. Isso facilita a vida de
quem for trabalhar com a base: não será preciso ter um grande trabalho inicial
para tratá-la. Além disso, também é fundamental sempre divulgar junto aos dados
um **Dicionário de Dados**, que consiste num documento explicando cada uma das
informações presentes no conjunto de dados (Colunas/variáveis, quais as possíveis
respostas em cada campo, etc).

Trabalhar com Dados Públicos, hoje, significa interagir e trocar, seja com outros
órgãos públicos, seja com a Sociedade Civil. Neste sentido, é fundamental conhecer,
entender e praticar os Princípios dos Dados Abertos[^14] de forma que todos consigam
estabelecer um real diálogo informacional que só favorece o desenvolvimento de uma
uma sociedade mais transparente, colaborativa e participativa.

[^14]: http://dados.gov.br/dados-abertos/

# Referências e Indicações
## Cursos e Outros links

 * Big Data em Saúde no Brasil: https://www.coursera.org/course/bigdatabrasil
 * TryR: http://tryr.codeschool.com/
 * The Data Scientist’s Toolbox: https://www.coursera.org/learn/data-scientists-tools
 * RProgramming: https://www.coursera.org/learn/r-programming
 * Exploratory Data Analysis: https://www.coursera.org/learn/exploratory-data-analysis
 * Statistics One: https://www.coursera.org/course/stats1
 * https://lagunita.stanford.edu/courses/HumanitiesSciences/StatLearning/Winter2016/about
 * http://online.stanford.edu/course/statistical-learning-winter-2014
 * \url{http://ocw.mit.edu/resources/res-9-0002-statistics-and-visualization-for-data-analysis-and-inference-january-iap-2009/}
 * https://www.edx.org/course
 * An introduction to ggplot2 - http://seananderson.ca/ggplot2-FISH554/
 * R Online Learning: https://www.rstudio.com/resources/training/online-learning/
 * Resources to help you learn and use R: http://statistics.ats.ucla.edu/stat/r/
 * Bons exemplos de uso: http://www.r-bloggers.com/

# Conteúdo e Licença
Este material e os arquivos e códigos utilizados para gerá-lo estão licenciados
sob uma licença Creative Commons CC-By-SA (https://creativecommons.org/licenses/by-sa/3.0/br/).
