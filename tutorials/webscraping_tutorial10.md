# Tutorial 10 - Arquivos em formato .pdf no R

Boa parte da informação disponibilizada por governos ou oganizações empresariais na internet está em formato .pdf. Neste tutorial vamos ver como obter textos contidos em um documento neste formato e treinar um exemplo que combina webscraping e arquivos em .pdf com dados da Câmara dos Deputados.

Finalmente, vamos ver como converter arquivos de .pdf digitalizados, ou seja, que são imagens de texto, em objetos do R com o pacote _tesseract_ (ainda não está disponível).

## Pacote _pdftools_

Até agora trabalhamos apenas com infomações em páginas em HTML que. Dada as marcações de um texto em HTML, conseguíamos com poucas funções impotar um documento, identificar sua estrutura e extrair infomações com precisão de dentro dele. O pacote _rvest_ contém tais funções é suficiente para a tarefa.

Com arquivos em .pdf, porém, precisaremos de outra ferramenta e outra estratégia de organização de dados. Nosso primeiro passo será utilizar o pacote _pdftools_ para transformar arquivos em .pdf em objetos de texto no R.

Comecemos instalando e caregando o pacote:

```{r}
install.packages("pdftools")
library(pdftools)
```

A principal função do pacote _pdftools_ é _pdf\_text_. Esta função recebe como argumento o caminho de um arquivo em .pdf, seja em uma pasta de seu computador ou na internet. Vamos utilizar como exemplos o documento com a proposição em inteiro teor da última PEC apresentada na data em que o tutorial foi elaborado e, a seguir, minha tese de doutorado defendida em 2014 na FGV-SP. Veja o uso da função é simples:

```{r}
url_ultima_pec <- "http://www.camara.gov.br/proposicoesWeb/prop_mostrarintegra;jsessionid=500EB6A69A190D8AEE117D22C6B3DFF9.proposicoesWebExterno2?codteor=1579429&filename=PEC+350/2017"

ultima_pec <- pdf_text(url_ultima_pec)

url_tese_leo <- "http://bibliotecadigital.fgv.br/dspace/bitstream/handle/10438/11683/teseLSB.pdf?sequence=1&isAllowed=y"

tese_leo <- pdf_text(url_tese_leo)

class(ultima_pec)
class(tese_leo)
length(ultima_pec)
length(tese_leo)
```

Em primeiro lugar, note que os objetos criados são vetores de texto (classe "character").

Antes de examinar seus conteúdos, note que os documenos têm tamanhos diferentes. Por que?

Ao convertermos o arquivo em .pdf em um vetor de texto no R, cada página é alocada em uma posição do vetor. A proposição da PEC tem 5 páginas. Minha tese, 178.

Veja a primeira página da proposição da PEC:

```{r}
ultima_pec[1]
```

A conversão de pdf para texto foi fácil. O difícil, veremos, será organizar o documento. Não temos mais à nossa disposição "caminhos" dentro do documento, tal qual o "xpath" em um documento HTML. Precisaremos, artesanalmente, conhecer os documentos e limpar utilizando as funções do pacote _stringr_.

Pode ser conveniente juntarmos todas as páginas. Com a função _paste_, e preenchendo o argumento "collapse", fazemos isso rapidamente:

```{r}
texto_ultima_pec <- paste(ultima_pec, collapse = "")
```

## Capturando e transformando diversos documentos em pdf

Vamos combinar o que aprendemos até agora de webscraping com o pacote _pdftools_. Nosso objeivo será capturar o texto do documento de inteiro teor de todas as proposições de PEC na Câmara dos Deputados no mês de junho de 2017, 10 ao todo. Vamos começar carregando o pacote _rvest_ e salvando o endereço URL da busca, que pode ser feita manualmente:

```{r}
library(rvest)

url_busca <- "http://www.camara.leg.br/buscaProposicoesWeb/resultadoPesquisa?emtramitacao=Todas&noorgao=&valueOrigem=-1&siglaorigem=&orgaoorigem=&naementa=true&indexacao=true&inteiroteor=false&tipoproposicao=%5BPEC++++++++-+Proposta+de+Emenda+%C3%A0+Constitui%C3%A7%C3%A3o%5D&dataInicialApresentacao=01/07/2017&dataFinalApresentacao=31/07/2017&partidoautor=&ufautor=&tramitacaoorgao=&partidorelator=&ufrelator=&comissaorelator=&data=03/08/2017&page=true"
```
Nosso segundo passo será capturar os URLs de cada proposição. A essa altura do curso você já deve estar acostumad@ com isso:

```{r}
urls_pecs <- url_busca %>%
  read_html() %>%
  html_nodes(xpath = "//span[@class='titulo']/a") %>%
  html_attr(name = 'href')
```

Vamos aproveitar a página de busca para guardar os números das PEC em um vetor para utilizá-los futuramente em um data frame:

```{r}
nomes_pecs <- url_busca %>%
  read_html() %>%
  html_nodes(xpath = "//span[@class='titulo']/a/span") %>%
  html_text()
```

Com os URLs de cada proposição em mãos, daremos os seguintes passos: (1) entraremos em cada URL; (2) capturaremos o link do documento de inteiro teor; (3) ainda antes de cada iteração encerrar, utilizaremos o link para converter o documento de inteiro teor em um vetor de texto, e, imediatamente a seguir, em um texto único que guadaremos em um vetor com os textos das demais PECs.

Note que os URLs dos documentos de inteiro teor capturados precisam de um pequeno complemento. Leia o código com calma para entender cada etapa e, se precisar exercitar, faça anotações com comentários.

```{r}
texto_pecs <- c()

for (u in urls_pecs) {

  pagina <- read_html(u)
  
  inteiro_teor <- pagina %>%
    html_nodes(xpath = "//span[@class = 'naoVisivelNaImpressao']/a[@class='rightIconified iconDetalhe linkDownloadTeor']") %>%
    html_attr(name = 'href')
    
  inteiro_teor <- paste0("http://www.camara.gov.br/proposicoesWeb/",
           inteiro_teor)
  
  texto <- pdf_text(inteiro_teor)
    
  texto <- paste(texto, collapse = "")
  
  texto_pecs <- c(texto_pecs, texto)
}
```
Temos, assim, dois vetores de tamanho 10, um com os números e outro com os textos das 10 PECs. Vamos organizá-los em um data frame.

```{r}
df_pec <- data.frame(nomes_pecs, texto_pecs)
```

Pronto! Os próximos passos lógicos seriam extrair o que nos interessa dos textos (justificação? texto legal? cabeçalho?) com o pacote _stringr_ e produzir análises. Você pode ler como fazer isso nos tutoriais sobre mineração de texto.
