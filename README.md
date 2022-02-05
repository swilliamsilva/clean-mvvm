# clean-mvvm

ESTRUTURA DE CLEAN e MVVM

ARQUITETURA LIMPA DE FORMA SIMPLES
_____________________________________

Nessa aplicação temos três camadas do Clean que representam
APRESENTAÇÃO (Presenters)  - DOMINIO  (Domain ) -DADOS (domain)
Dentro da camada PRESENTER temos a nossa MVVM composta pela VIEW e
 VIEW MODEL

# FLUXO NA APLICAÇÃO NA ARQUITETURA LIMPA

- O usuário tem uma iteração com a parte da VIEW

- A VIEW pega essa informação e repassa para a VIEWMODEL, para que ela
trate essa informção e ser for necessario fazer alguma operação com essa
informação ela manda isso para a camada de DOMINIO.

-Se a camada de DOMINIO necessitar de algum dado ela busca na
camada de DADOS

-A informação e retornada da camada DATA para a camada de DOMINIO
e da camada de DOMINIO para nossa VIEW MODEL na camada de apresentação.
A VIEW MODEL modifica a VIEW que por consequencia atualiza o estado
da VIEW e mostra a informação nova para o usuário.

O que representa cada uma das camadas?

DADOS

Quando a gente fala da camada de DADOS estamos falando que vamos ter
um repositório e este deve saber onde ele deve buscar as informações
que ele precisa.

* Então: no nosso caso ela vai buscar os dados dentro de uma API.

Então vai ter uma função.

Essa função vai requisitar do nosso SERVICE as informações que ela precisa.
E vai receber essa informação do nosso SERVICE através de um RESPONSE.

DOMINIO

Já a nossa camada de dominio ela vai ser composta por uma USER CASE .
Ela vai buscar a informação na nossa camada de DADOS através de uma
função e vai receber como resposta dessa chamada em uma lista de
categorias e por fim na última camada do CLEAN temos a aprensentação
 " PRESENTER."

PRESENTER
Composto pela VIEW é aqui que vamos criar um fragmento, o nosso FRAGMENT
que vai se comunicar com o nosso VIEW MODEL e o VIEWMODEL e vai fazer
 a comunicação com a camada de DOMiNIO.

# NA APLICAÇÃO ONDE USAMOS CLEAN COM MVVM

Nessa  aplicação simples no ANDROID,  aqui dentro eu criei só um modulo
a mais chamado HELPERs, dentro desse módulo eu tenho basicamente a minha
camada de rede o que eu vou usar para conseguir chamar meu servidor.

A nossa arquitetura, ela vai ser aplicada no módulo WEB e aqui a gente
vai começar pelo CLEAN.

Então ela esta separada nesses em três pacotes. O DATA, DOMAIN, PRESENTER.

Então vamos começar falando da camada de DATA.

Como sabe que ela vai ter que buscar as informações em um servidor dentro do
pacote DATA vamos  criar dois pacotes.

## DATA

(* Nesse exercicio aprenda a localizar no código os pontos de marcação do
 fluxo, aqui sermpre destacado em vermelho e entre parenteses. )

API e MODEL

MODEL

Dentro do MODEL a gente vai criar o modelo de RESPONSE

 (CategoryResponse.kt ) que existe no nosso sevidor,

e dentro da API a gente vai criar nossa API ( MealApi ) onde vai fazer
a rquisição propriamente no nosso servidor.

API
Nossa API é composta por um método GET  (getCategories() ) e a resposta o,
RESPONSE dessa API ( Response<listCategories...) é uma lista de categorias.
Com o nosso servidor montado vamos ver como ficou o repositório.
(MealRepository.kt)

Repare que esta sendo criado uma INTERFACE (interface MealRepository )
 o REPOSITORY e também a implementação dela que possui um construtor.

 (Class MealRepositoryImpl(
private val service: MealAPI ..)

estamos fazendo isso basicamente por duas coisas:
para facilitar os testes e a minha injeção de dependência.
Depois aprofrundaremos nesse assunto.

Olhando a implementação da função GET CATEGORY o que acontece?

O REPOSITORY sabe exatamente onde ele tem que buscar as informações.

(set result =service.getCategories()....)

Então nesse caso ele sabe que para pegar nossa categorias ele tem que
 acessar o nosso método GET CATEGORY
(return when (result) ) e após a chamada de serviço nós temos o nosso
 RESULT.
O nosso RESULT (return when (result) ) ele pode assumir dois estados um
e sucesso e outro de erro.
Em caso de erro nós vamos apenas retornar uma lista vazia e isso
apenas para simplificar o exemplo.
Porém em caso de sucesso ( val categoryResponseList = result...)
o que nós temos dentro do nosso RESULT é uma lista de CATEGORY RESPONSE,
um objeto que a minha camada de dados conhece, porém
eu não posso retornar isso para a minha camada de DOMINIO.

Então é aqui que tenho que fazer é o mapeamento do objeto
(categoryResponseList.map ) ou uma conversão ( it.toCategory() )
 de CATEGORY RESPONSE para CATEGORY e para fazer a conversão desse objeto
 nesse exemplo eu  utilizao uma Extension functions,
 veja: https://kotlinlang.org/docs/extensions.html

( fun CategoryResponse.toCategory() = Category( )
 para fazer o mapeamento.

Você pode fazer isso de outra maneira.. dá que você achar melhor.
O importante mesmo  é que sua camada de DOMINIO não conheça o objeto
de RESPONSE.

E o que rola dessa camada de objetos da camada de dados para a camada de
dominio?

Imagina que o seu objeto o CATEGORY RESPONSE
(MealRepositorie.kt|)
(..
it.toCategory()
)
e ele  é utilizado em vários "End Point" do seu servidor o que é algo
bem comum de acontecer, imagina também que por algum motivo vai ocorrer
uma mudança no seu servidor e se você não possui esse mapeamento o que
vai acontecer, você vai ter que fazer além da mudança desse objeto na
sua camada de dados.

Você vai precisar também ter que procurar esse objeto em outras pontas
da sua aplicação, lá na camada de dominio, todo que esse objeto  tem  que
ser alterado, agora se existir o mapeamento essas alterações irão  ocorrer
apenas na sua camada de dados e o restante da sua aplicação vai continuar
funcionando normalmente porque o objeto vai ter sido convertido no seu
mapeamento.

( data )
Com a camada de DADOS concluida vamos para a camada de DOMINIO.
(domain )

## DOMAIN
Onde ficam as nossas regras de negócio.
(Category.kt)

Na camada de dominio nós temos o modelo o CATEGORY
(data class Category(
            val id: String...)

 e também a nossa USER CASE.
( GatCategories.kt )
( override suspend fun...
   mealRepository.getCategories()
)

e  USER CASE repare que também temos a INTERFACE
(Interface GetCategoriesCase )

e a sua implementação usando Koin, saiba mais em: https://insert-koin.io/
(
) : GetCategoriesUseCase {    )
como nesse caso nós só precisamos buscar a nossa lista de CATEGORIAS,
( mealRepository.getCategories()  )
aqui só esta acontecendo a nossa chamada para o nosso repositório que
esta sendo injetado no construtor da implementação.

Mas digamos que por algum motivo nossa lista tenha que ser modificada
( mealRepository.getCategories().filter { )...depois apague filter )
seguindo algum padrão,  tipo um filtro ou alguma coisa parecida e nossa
camada que iremos manipular a nossa lista,  já que isso, essa manipulações
fazem parte da nossa regra de negócio; e passando para a ultima camada o
PRESENTER.

## PRESENTER

Onde fica o nosso MVVM e começando pela VIEW o nosso FRAGMENT
( PRESENTER
Model
Class ModelMealFragment
..
vielModel.getCategories()
)

Lembra que na MVVM a VIEW conhece a VIEWMODEL e que ela notifica a VIEW.
( private val viewModel: MealViewModel by sharedViewModel () )

Vamos passo a passo aqui.
A VIEW possui uma instância da nossa VIEW MODEL,  que é feita aqui
e a VIEW também observa uma variável do nosso VIEW MODEL de que é feita a
nessa linha e nessa parte
(viewModel.categories.abserve( viesLifeCyclerOwner )
que ocorre uma mudança no estado da nossa variável lá no VIEW MODEL
( populateMealCategories(listCategory:Model )
e cada mudança de estado o PopulateMealCategories é chamado com um
novo valor e aqui ouve  o viewModel.getCategories() é a inicialização
da nossa aplicação.
( viewModel.getCategories () )
É aqui que ocorre a inicialização da aplicação.
( class MealViewModel
...
fun getCategories ()  {
..)

Então quando na VIEWMODEL. GETCATEGORY é chamado o nosso VIEWMODEL entra
em ação
( val categoryList = getCategoriesUserCase )

e executa a nossa USER CASE e o VIEW MODEL que também converte o objeto
( categrory.toUiModel() )
CATEGORY  que vem lá do dominio, mas agora para um objeto, que a nossa
VIEW conhece e também através de um mapeamento.

Nesse exemplo eu chamo o nosso mapeamento de ToUIModel ()  e o nosso modelo
( category.toUiModel() )
de VIEW o CATEGORY.toUIMODEL() e a gente faz isso seguindo a mesma logica,
se o CATEGORY ele é usado e vários pontoS da nossa VIEW
e por algum motivo o  CATEGORY sofre alguma alteração, nos temos apenas que
alterar o nosso mapeamento e a camada de dominio e as nossas VIEW vão
continuar funcionando normalmente.
