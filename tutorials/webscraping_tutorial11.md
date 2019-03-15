# Tutorial 11 - Pacote Rfacebook

Neste breve tutorial vamos ver como acessar a API do facebook com R e obter dados informaçõs sobre um conjunto limitado de postagens.

Diferentemente do twitter, o facebook restringe os dados que podem ser acessados via API. Os únicos dados que conseguimos capturar com o pacote _Rfacebook_ e que os interessam para fins de pesquisa são as postagens de páginas públicas e grupos aos quais pertencemos.

Há um bom tutorial sobre no [repositório do pacote _Rfacebook_](https://github.com/pablobarbera/Rfacebook). Sugiro sua leitura.

## O pacote _Rfacebook_

Vamos começar carregando e instalando o pacote:

```{r}
install.packages("Rfacebook")
library(Rfacebook)
```

O primeiro passo para obter dados do facebook é criar uma conta de desenvolvedor(a). Se você tem uma conta no facebook, clique [aqui](https://developers.facebook.com/), faça login e configure sua conta de desenvolvedor(a). 

O autor do pacote recomenda este [tutorial](http://thinktostart.com/analyzing-facebook-with-r/) para fazer a preparação do acesso à API. Explico rapidamente os passos abaixo e você pode recorrer a ele para ver imagens de telas (desatualizadas e em inglês) que podem ajudar.

* 1- Clique no botão "Criar aplicativo"

* 2- Dê um nome para seu aplicativo em "Nome de exibição" e clique em "Crie um número de identificação do aplicativo". Sobreviva ao "captcha" e confirma a criação.

* 3- No "Painel" do aplicativo você verá a Versão da API, o ID do Aplicativo e a Chave Secreta do Aplicativo. Clique em "Mostrar" para exibir a chave secreta e anote os números do id e da chave secreta em dois vetores diferentes (substitua o "888"): 

```{r}
app_id <- "888"
app_secret <- "888"
```

* 4- Vá em "Conigurações" e clique no botão "+ Adicionar plataforma" na parte inferior. Escolha "Site" e insira no campo "URL do site" o seguinte endereço: http://localhost:1410/ . Clique em "Salva Alterações"

* 5- Pronto! Vamos agora fazer a autenticação de dentro do R:

```{r}
fb_oauth <- fbOAuth(app_id = app_id, 
                    app_secret = ,
                    extended_permissions = TRUE)
```

* 6- Siga as instruções que aparecerem no console (a primeira já está feita, que é colar o URL do site). Clique em qualquer tecla. Você será redirecionado ao navegador para conceder algumas permissões no seu perfil. 

* 7- É interessante salvar o objeto de autenticação em uma pasta do seu computador. Você poderá reutilizá-lo sempre que quiser para evitar fazer todas as vezes o processo de autenticação.

```{r}
save(fb_oauth, file="fb_oauth")
load("fb_oauth")
```

Alternativamente à autenticação com Id e chave secreta, podemos utilizar o token de acesso para acessar a API. Ele dura por duas horas, apenas. Depois disso é preciso renová-lo.

Clique [aqui](https://developers.facebook.com/tools/explorer). Na página à qual você será redirecionad@, você poderá copiar o "Token de acesso". Durante a aula, deixarei um token válido para vocês explorarem minha conta.

```{r}
token_acesso <- 'EAACEdEose0cBAKJgbGubZCMUagDwq3ZAIlVrpuZAlCRPGCEqF7qjlSpaRbEqQOUHh8jsHyNTQxU3aj4ZCNw5r3xyrV2Jd2Qv5wvScOkLCGRBsVWgjvdoOZB7t3VZBEdrsrvDar30CIGUkV1dkNklJoC6MHSuFpBp0nWe4O7LH4q2po0ovKBxiL6sH5ZCd8cyhcZD'
```

Não se frustre se não der certo a primeira vez (não deu para mim). Recomece o processo e leia o tutorial que indiquei.

## Obtendo dados com Rfacebook

Vamos ver basicamente 3 funções do pacote _Rfacebook_: _getUsers_, _getPage_ e _getGroup_, que retornam, respectivamente, os dados sobre você (ou sobre um outro usuário vinculado ao seu aplicativo - não funciona para os demais), sobre uma página pública ou sobre um grupo ao qual você pertence.

Sobre você :

```{r}
me <- getUsers("me", token_acesso)
View(page)
```

Desinteressante. Vamos agora testar com uma página. Escolhi como exemplo a página ["I Fucking Love Science"](https://www.facebook.com/IFeakingLoveScience/). Note no URL da página que o "nome do usuário" da página (sic) é "IFeakingLoveScience".

Vamos inserir três argumentos na função _getPage_: o "nome do usuário" da página; o token de acesso, que pode ser tanto os objetos "fb_oauth" ou "token_acesso", que criamos acima; e o número de postagem que desejamos:

```{r}
page <- getPage(page = "IFeakingLoveScience", 
                token = token_acesso, 
                n = 100)
View(page)
```

O resultado da função é um data frame com as últimas 100 postagens da página. Dentre as variáveis mais interessantes estão o texto do post, usuário que postou, data, url para o post (hey, podemos tentar usar isso aqui de outra forma!), o número de reações, shares e comentários.

Há alguns outros argumntos importantes na função. Em vez do númeo de postagens, você pode utilizar os argumentos "since" e "until" para delimitar um intervalo de tempo para obter postagens. Por exemplo:

```{r}
page <- getPage(page = "IFeakingLoveScience", 
                token = token_acesso,
                n = 100,
                since='2017/08/01',
                until='2017/08/04')
View(page)
```


O argumento "feed" incluirá na busca as postagens feitas por outros usuários na página. Finalmente, "reactions" incluirá variáveis detalhadas sobre cada reação em vez da simples contagem.

O funcionamnto de _getComments_ é bastante semelhante. No exemplo abaixo, vamos pegar os comentários a uma postagem da página da qual acabamos de obter as postagens. Em primeiro lugar, vamos selecionar o id da primeira postagem usando os dados acima capturados:

```{r}
id_postagem <- page@id[1]
```

A seguir, vamos obter os dados sobre a primeira postagem

```{r}
post <- getPost(post = id_postagem,
                  token = token_acesso)
str(post)
```

Note que uma postagem é uma lista. Na primeira posição da lista, temos as informações sobre o post e na segunda os comentários ao post. Vamos separar os comentários em um objeto a parte:

```{r}
comments <- post$comments
View(comments)
```

Finalmente, se quisermos podemos usar a função _getCommentReplies_ para observar os replies a cada um dos comentários.
