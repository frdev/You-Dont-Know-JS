# You Don't Know JS Yet: Scope & Closures - 2ª Edição
# Capítulo 2: Ilustrando o Escopo Léxico

No Capítulo 1, exploramos como o escopo é determinado durante a compilação do código, um modelo chamado de "escopo léxico". O termo "léxico" se refere ao primeiro estágio da compilação (lexing/parsing).

Para *raciocinar* corretamente sobre nossos programas, é importante ter uma base conceitual sólida de como o escopo funciona. Se nos apoiarmos em palpites e intuição, podemos acidentalmente acertar as respostas algumas vezes, mas em muitas outras vamos errar feio. Essa não é uma receita de sucesso.

Como lá na aula de matemática do ensino fundamental, acertar a resposta não basta se não mostrarmos os passos corretos para chegar lá! Precisamos construir modelos mentais precisos e úteis como base para seguir em frente.

Este capítulo vai ilustrar o *escopo* com várias metáforas. O objetivo aqui é *pensar* sobre como seu programa é tratado pela engine JS de formas que se alinhem mais de perto com o modo como a engine JS realmente funciona.

## Bolinhas, e Baldes, e Bolhas... Ai, Meu Deus!

Uma metáfora que considero eficaz para entender escopo é separar bolinhas de gude coloridas em baldes da cor correspondente.

Imagine que você se depara com uma pilha de bolinhas de gude e nota que todas são vermelhas, azuis ou verdes. Vamos separar todas as bolinhas, colocando as vermelhas no balde vermelho, as verdes no balde verde e as azuis no balde azul. Depois de separar, quando você precisar de uma bolinha verde, já sabe que é no balde verde que deve buscar.

Nessa metáfora, as bolinhas são as variáveis do nosso programa. Os baldes são escopos (funções e blocos), aos quais atribuímos conceitualmente cores individuais para fins da nossa discussão. A cor de cada bolinha é, portanto, determinada por em qual escopo de qual *cor* encontramos a bolinha originalmente criada.

Vamos anotar o programa de exemplo que vem do Capítulo 1 com rótulos de cor de escopo:

```js
// escopo externo/global: VERMELHO

var students = [
    { id: 14, name: "Kyle" },
    { id: 73, name: "Suzy" },
    { id: 112, name: "Frank" },
    { id: 6, name: "Sarah" }
];

function getStudentName(studentID) {
    // escopo da função: AZUL

    for (let student of students) {
        // escopo do laço: VERDE

        if (student.id == studentID) {
            return student.name;
        }
    }
}

var nextStudent = getStudentName(73);
console.log(nextStudent);   // Suzy
```

Designamos três cores de escopo com comentários no código: VERMELHO (escopo global mais externo), AZUL (escopo da função `getStudentName(..)`) e VERDE (escopo de/dentro do laço `for`). Mas ainda pode ser difícil reconhecer os limites desses baldes de escopo olhando para uma listagem de código.

A Figura 2 ajuda a visualizar os limites dos escopos desenhando bolhas coloridas (ou seja, baldes) ao redor de cada um:

<figure>
    <img src="../../../scope-closures/images/fig2.png" width="500" alt="Bolhas de Escopo Coloridas" align="center">
    <figcaption><em>Fig. 2: Bolhas de Escopo Coloridas</em></figcaption>
</figure>

1. A **Bolha 1** (VERMELHA) abrange o escopo global, que contém três identificadores/variáveis: `students` (linha 1), `getStudentName` (linha 8) e `nextStudent` (linha 16).

2. A **Bolha 2** (AZUL) abrange o escopo da função `getStudentName(..)` (linha 8), que contém apenas um identificador/variável: o parâmetro `studentID` (linha 8).

3. A **Bolha 3** (VERDE) abrange o escopo do laço `for` (linha 9), que contém apenas um identificador/variável: `student` (linha 9).

| NOTA: |
| :--- |
| Tecnicamente, o parâmetro `studentID` não está exatamente no escopo AZUL(2). Vamos desenrolar essa confusão em "Escopos Implícitos", no Apêndice A. Por ora, é próximo o suficiente rotular `studentID` como uma bolinha AZUL(2). |

As bolhas de escopo são determinadas durante a compilação com base em onde as funções/blocos de escopo são escritos, no aninhamento de uns dentro dos outros e assim por diante. Cada bolha de escopo está inteiramente contida dentro da sua bolha de escopo pai — um escopo nunca está parcialmente em dois escopos externos diferentes.

Cada bolinha (variável/identificador) é colorida com base em qual bolha (balde) ela é declarada, e não na cor do escopo de onde ela pode ser acessada (por exemplo, `students` na linha 9 e `studentID` na linha 10).

| NOTA: |
| :--- |
| Lembre-se de que afirmamos no Capítulo 1 que `id`, `name` e `log` são todos propriedades, não variáveis; em outras palavras, não são bolinhas em baldes, então não recebem cor com base em nenhuma das regras que estamos discutindo neste livro. Para entender como esses acessos a propriedades são tratados, veja o terceiro livro da série, *Objects & Classes*. |

Conforme a engine JS processa um programa (durante a compilação) e encontra uma declaração de variável, ela essencialmente pergunta: "Em qual escopo (bolha ou balde) de qual *cor* eu estou agora?" A variável é designada com essa mesma *cor*, o que significa que ela pertence àquele balde/bolha.

O balde VERDE(3) está totalmente aninhado dentro do balde AZUL(2) e, de forma semelhante, o balde AZUL(2) está totalmente aninhado dentro do balde VERMELHO(1). Escopos podem se aninhar uns dentro dos outros como mostrado, em qualquer profundidade de aninhamento de que seu programa precise.

Referências (não declarações) a variáveis/identificadores são permitidas se houver uma declaração correspondente no escopo atual ou em qualquer escopo acima/fora do escopo atual, mas não com declarações de escopos inferiores/aninhados.

Uma expressão no balde VERMELHO(1) só tem acesso a bolinhas VERMELHAS(1), **não** a AZUIS(2) nem VERDES(3). Uma expressão no balde AZUL(2) pode referenciar bolinhas AZUIS(2) ou VERMELHAS(1), **não** VERDES(3). E uma expressão no balde VERDE(3) tem acesso a bolinhas VERMELHAS(1), AZUIS(2) e VERDES(3).

Podemos conceituar o processo de determinar as cores dessas bolinhas que não são declarações, em tempo de execução, como uma busca. Como a referência à variável `students` na instrução do laço `for` na linha 9 não é uma declaração, ela não tem cor. Então perguntamos ao balde de escopo AZUL(2) atual se ele tem uma bolinha com esse nome. Como não tem, a busca continua com o próximo escopo externo/contêiner: VERMELHO(1). O balde VERMELHO(1) tem uma bolinha de nome `students`, então a referência à variável `students` da instrução do laço é determinada como uma bolinha VERMELHA(1).

A instrução `if (student.id == studentID)` na linha 10 é, de forma semelhante, determinada como referenciando uma bolinha VERDE(3) chamada `student` e uma bolinha AZUL(2) chamada `studentID`.

| NOTA: |
| :--- |
| A engine JS geralmente não determina essas cores de bolinhas em tempo de execução; a "busca" aqui é um recurso retórico para te ajudar a entender os conceitos. Durante a compilação, a maioria ou todas as referências a variáveis vão corresponder a baldes de escopo já conhecidos, então sua cor já está determinada e armazenada junto a cada referência de bolinha, para evitar buscas desnecessárias enquanto o programa roda. Mais sobre essa sutileza no Capítulo 3. |

As principais conclusões sobre bolinhas e baldes (e bolhas!):

* Variáveis são declaradas em escopos específicos, que podem ser pensados como bolinhas coloridas de baldes de cor correspondente.

* Qualquer referência a variável que apareça no escopo em que ela foi declarada, ou em quaisquer escopos aninhados mais profundos, será rotulada como uma bolinha daquela mesma cor — a menos que um escopo intermediário "sombreie" a declaração da variável; veja "Sombreamento" no Capítulo 3.

* A determinação dos baldes coloridos, e das bolinhas que eles contêm, acontece durante a compilação. Essa informação é usada para as "buscas" de variáveis (cor da bolinha) durante a execução do código.

## Uma Conversa Entre Amigos

Outra metáfora útil para o processo de analisar variáveis e os escopos de onde elas vêm é imaginar várias conversas que ocorrem dentro da engine enquanto o código é processado e depois executado. Podemos "escutar" essas conversas para ter uma base conceitual melhor de como escopos funcionam.

Vamos agora conhecer os membros da engine JS que vão conversar enquanto processam nosso programa:

* *Engine*: responsável pela compilação e execução do nosso programa JavaScript, do início ao fim.

* *Compilador*: um dos amigos da *Engine*; cuida de todo o trabalho sujo de parsing e geração de código (veja a seção anterior).

* *Gerente de Escopo*: outro amigo da *Engine*; coleta e mantém uma lista de consulta de todas as variáveis/identificadores declarados, e faz valer um conjunto de regras sobre como eles são acessíveis ao código em execução.

Para você *entender plenamente* como o JavaScript funciona, precisa começar a *pensar* como a *Engine* (e seus amigos) pensa, fazer as perguntas que eles fazem e responder às perguntas deles da mesma forma.

Para explorar essas conversas, lembre-se novamente do nosso programa de exemplo:

```js
var students = [
    { id: 14, name: "Kyle" },
    { id: 73, name: "Suzy" },
    { id: 112, name: "Frank" },
    { id: 6, name: "Sarah" }
];

function getStudentName(studentID) {
    for (let student of students) {
        if (student.id == studentID) {
            return student.name;
        }
    }
}

var nextStudent = getStudentName(73);

console.log(nextStudent);
// Suzy
```

Vamos examinar como o JS vai processar esse programa, começando especificamente pela primeira instrução. O array e seu conteúdo são apenas literais básicos de valores JS (e, portanto, não afetados por questões de escopo), então nosso foco aqui será nas partes de declaração e de atribuição-inicialização de `var students = [ .. ]`.

Tipicamente pensamos nisso como uma única instrução, mas não é assim que nosso amigo *Engine* enxerga. De fato, o JS trata isso como duas operações distintas: uma que o *Compilador* vai tratar durante a compilação, e outra que a *Engine* vai tratar durante a execução.

A primeira coisa que o *Compilador* vai fazer com este programa é realizar o lexing para quebrá-lo em tokens, que ele então vai transformar, via parsing, em uma árvore (AST).

Quando o *Compilador* chega à geração de código, há mais detalhes a considerar do que pode parecer. Uma suposição razoável seria que o *Compilador* vai produzir, para a primeira instrução, um código do tipo: "Aloque memória para uma variável, rotule-a como `students`, depois enfie uma referência ao array nessa variável." Mas essa não é a história toda.

Aqui estão os passos que o *Compilador* vai seguir para tratar essa instrução:

1. Ao encontrar `var students`, o *Compilador* vai perguntar ao *Gerente de Escopo* se já existe uma variável chamada `students` naquele balde de escopo específico. Se existir, o *Compilador* ignoraria essa declaração e seguiria em frente. Caso contrário, o *Compilador* vai produzir código que (em tempo de execução) pede ao *Gerente de Escopo* que crie uma nova variável chamada `students` naquele balde de escopo.

2. O *Compilador* então produz código para a *Engine* executar depois, para tratar a atribuição `students = []`. O código que a *Engine* roda vai primeiro perguntar ao *Gerente de Escopo* se existe uma variável chamada `students` acessível no balde de escopo atual. Se não, a *Engine* continua procurando em outro lugar (veja "Escopo Aninhado", abaixo). Assim que a *Engine* encontra uma variável, ela atribui a referência do array `[ .. ]` a ela.

Em forma de conversa, a primeira fase da compilação do programa poderia se desenrolar entre o *Compilador* e o *Gerente de Escopo* assim:

> ***Compilador***: Ei, *Gerente de Escopo* (do escopo global), encontrei uma declaração formal para um identificador chamado `students`, já ouviu falar dele?

> ***Gerente de Escopo (Global)***: Não, nunca ouvi, então acabei de criá-lo para você.

> ***Compilador***: Ei, *Gerente de Escopo*, encontrei uma declaração formal para um identificador chamado `getStudentName`, já ouviu falar dele?

> ***Gerente de Escopo (Global)***: Não, mas acabei de criá-lo para você.

> ***Compilador***: Ei, *Gerente de Escopo*, `getStudentName` aponta para uma função, então precisamos de um novo balde de escopo.

> ***Gerente de Escopo (da Função)***: Entendido, aqui está o balde de escopo.

> ***Compilador***: Ei, *Gerente de Escopo* (da função), encontrei uma declaração de parâmetro formal para `studentID`, já ouviu falar dele?

> ***Gerente de Escopo (da Função)***: Não, mas agora ele foi criado neste escopo.

> ***Compilador***: Ei, *Gerente de Escopo* (da função), encontrei um laço `for` que vai precisar do seu próprio balde de escopo.

> ...

A conversa é uma troca de perguntas e respostas, em que o **Compilador** pergunta ao *Gerente de Escopo* atual se uma declaração de identificador encontrada já foi vista antes. Se "não", o *Gerente de Escopo* cria essa variável naquele escopo. Se a resposta for "sim", então ela é efetivamente ignorada, já que não há mais nada para aquele *Gerente de Escopo* fazer.

O *Compilador* também sinaliza quando encontra funções ou escopos de bloco, para que um novo balde de escopo e um novo *Gerente de Escopo* possam ser instanciados.

Depois, quando chega a hora de executar o programa, a conversa muda para a *Engine* e o *Gerente de Escopo*, e poderia se desenrolar assim:

> ***Engine***: Ei, *Gerente de Escopo* (do escopo global), antes de começarmos, você pode procurar o identificador `getStudentName` para eu atribuir esta função a ele?

> ***Gerente de Escopo (Global)***: Sim, aqui está a variável.

> ***Engine***: Ei, *Gerente de Escopo*, encontrei uma referência de *alvo* para `students`, já ouviu falar dela?

> ***Gerente de Escopo (Global)***: Sim, ela foi formalmente declarada para este escopo, então aqui está.

> ***Engine***: Obrigado, estou inicializando `students` como `undefined`, então está pronta para uso.

> Ei, *Gerente de Escopo* (do escopo global), encontrei uma referência de *alvo* para `nextStudent`, já ouviu falar dela?

> ***Gerente de Escopo (Global)***: Sim, ela foi formalmente declarada para este escopo, então aqui está.

> ***Engine***: Obrigado, estou inicializando `nextStudent` como `undefined`, então está pronta para uso.

> Ei, *Gerente de Escopo* (do escopo global), encontrei uma referência de *origem* para `getStudentName`, já ouviu falar dela?

> ***Gerente de Escopo (Global)***: Sim, ela foi formalmente declarada para este escopo. Aqui está.

> ***Engine***: Ótimo, o valor em `getStudentName` é uma função, então vou executá-la.

> ***Engine***: Ei, *Gerente de Escopo*, agora precisamos instanciar o escopo da função.

> ...

Essa conversa é outra troca de perguntas e respostas, em que a *Engine* primeiro pede ao *Gerente de Escopo* atual que procure o identificador `getStudentName` (que sofreu hoisting), para associar a função a ele. A *Engine* então prossegue perguntando ao *Gerente de Escopo* sobre a referência de *alvo* para `students`, e assim por diante.

Para revisar e resumir como uma instrução como `var students = [ .. ]` é processada, em dois passos distintos:

1. O *Compilador* monta a declaração da variável de escopo (já que ela não havia sido declarada antes no escopo atual).

2. Enquanto a *Engine* está executando, para processar a parte de atribuição da instrução, a *Engine* pede ao *Gerente de Escopo* que procure a variável, inicializa-a como `undefined` para que fique pronta para uso e então atribui o valor do array a ela.

## Escopo Aninhado

Quando chega a hora de executar a função `getStudentName()`, a *Engine* pede uma instância de *Gerente de Escopo* para o escopo daquela função, e então vai procurar o parâmetro (`studentID`) para atribuir a ele o valor do argumento `73`, e assim por diante.

O escopo de função de `getStudentName(..)` está aninhado dentro do escopo global. O escopo de bloco do laço `for` está, de forma semelhante, aninhado dentro daquele escopo de função. Escopos podem ser lexicamente aninhados em qualquer profundidade arbitrária que o programa definir.

Cada escopo ganha sua própria instância de *Gerente de Escopo* cada vez que esse escopo é executado (uma ou mais vezes). Cada escopo tem automaticamente todos os seus identificadores registrados no início da execução do escopo (isso é chamado de "hoisting de variáveis"; veja o Capítulo 5).

No início de um escopo, se algum identificador veio de uma declaração de `function`, essa variável é automaticamente inicializada com sua referência de função associada. E, se algum identificador veio de uma declaração `var` (em oposição a `let`/`const`), essa variável é automaticamente inicializada como `undefined`, para que possa ser usada; caso contrário, a variável permanece não inicializada (ou seja, na sua "TDZ"; veja o Capítulo 5) e não pode ser usada até que sua declaração-e-inicialização completa seja executada.

Na instrução `for (let student of students) {`, `students` é uma referência de *origem* que precisa ser buscada. Mas como essa busca será tratada, já que o escopo da função não vai encontrar tal identificador?

Para explicar, vamos imaginar esse pedaço de conversa se desenrolando assim:

> ***Engine***: Ei, *Gerente de Escopo* (da função), tenho uma referência de *origem* para `students`, já ouviu falar dela?

> ***Gerente de Escopo (da Função)***: Não, nunca ouvi. Tente o próximo escopo externo.

> ***Engine***: Ei, *Gerente de Escopo* (do escopo global), tenho uma referência de *origem* para `students`, já ouviu falar dela?

> ***Gerente de Escopo (Global)***: Sim, foi formalmente declarada, aqui está.

> ...

Um dos aspectos centrais do escopo léxico é que, sempre que uma referência a identificador não pode ser encontrada no escopo atual, o próximo escopo externo do aninhamento é consultado; esse processo é repetido até que uma resposta seja encontrada ou até não haver mais escopos a consultar.

### Falhas de Busca

Quando a *Engine* esgota todos os escopos *lexicamente disponíveis* (movendo-se para fora) e ainda assim não consegue resolver a busca de um identificador, existe então uma condição de erro. Porém, dependendo do modo do programa (modo estrito ou não) e do papel da variável (ou seja, *alvo* vs. *origem*; veja o Capítulo 1), essa condição de erro será tratada de forma diferente.

#### A Bagunça do Undefined

Se a variável é uma *origem*, uma busca de identificador não resolvida é considerada uma variável não declarada (desconhecida, ausente), o que sempre resulta em um `ReferenceError` sendo lançado. Além disso, se a variável é um *alvo* e o código naquele momento está rodando em modo estrito, a variável é considerada não declarada e lança igualmente um `ReferenceError`.

A mensagem de erro para uma condição de variável não declarada, na maioria dos ambientes JS, vai ser algo como "Reference Error: XYZ is not defined." A expressão "not defined" (não definida) parece quase idêntica à palavra "undefined" (indefinida), no que diz respeito à língua inglesa. Mas as duas são muito diferentes em JS, e essa mensagem de erro, infelizmente, cria uma confusão persistente.

"Not defined" realmente significa "não declarada" — ou, melhor, "undeclared", como em uma variável que não tem declaração formal correspondente em nenhum escopo *lexicamente disponível*. Em contraste, "undefined" realmente significa que uma variável foi encontrada (declarada), mas que ela não tem nenhum outro valor no momento, então assume por padrão o valor `undefined`.

Para perpetuar ainda mais a confusão, o operador `typeof` do JS retorna a string `"undefined"` para referências de variáveis em qualquer um dos dois estados:

```js
var studentName;
typeof studentName;     // "undefined"

typeof doesntExist;     // "undefined"
```

Essas duas referências de variáveis estão em condições bem diferentes, mas o JS certamente turva as águas. A bagunça terminológica é confusa e terrivelmente infeliz. Infelizmente, desenvolvedores JS simplesmente têm que prestar muita atenção para não confundir *com qual tipo* de "undefined" estão lidando!

#### Global... O Quê!?

Se a variável é um *alvo* e o modo estrito não está em vigor, entra em cena um comportamento legado confuso e surpreendente. O resultado problemático é que o *Gerente de Escopo* do escopo global vai simplesmente criar uma **variável global acidental** para satisfazer aquela atribuição de alvo!

Considere:

```js
function getStudentName() {
    // atribuição a uma variável não declarada :(
    nextStudent = "Suzy";
}

getStudentName();

console.log(nextStudent);
// "Suzy" -- ops, uma variável global acidental!
```

Veja como essa *conversa* vai se desenrolar:

> ***Engine***: Ei, *Gerente de Escopo* (da função), tenho uma referência de *alvo* para `nextStudent`, já ouviu falar dela?

> ***Gerente de Escopo (da Função)***: Não, nunca ouvi. Tente o próximo escopo externo.

> ***Engine***: Ei, *Gerente de Escopo* (do escopo global), tenho uma referência de *alvo* para `nextStudent`, já ouviu falar dela?

> ***Gerente de Escopo (Global)***: Não, mas, como estamos em modo não estrito, dei uma força e criei uma variável global para você, aqui está!

Eca.

Esse tipo de acidente (quase certamente levando a bugs em algum momento) é um ótimo exemplo das proteções benéficas oferecidas pelo modo estrito, e de por que é uma péssima ideia *não* estar usando modo estrito. Em modo estrito, o ***Gerente de Escopo Global*** teria respondido:

> ***Gerente de Escopo (Global)***: Não, nunca ouvi falar. Desculpe, vou ter que lançar um `ReferenceError`.

Atribuir a uma variável nunca declarada *é* um erro, então é justo que recebamos um `ReferenceError` aqui.

Nunca dependa de variáveis globais acidentais. Sempre use modo estrito e sempre declare formalmente suas variáveis. Assim, você vai receber um `ReferenceError` útil se algum dia tentar, por engano, atribuir a uma variável não declarada.

### Construindo Sobre Metáforas

Para visualizar a resolução de escopo aninhado, prefiro ainda outra metáfora, a de um edifício de escritórios, como na Figura 3:

<figure>
    <img src="../../../scope-closures/images/fig3.png" width="250" alt="&quot;Edifício&quot; de Escopos" align="center">
    <figcaption><em>Fig. 3: "Edifício" de Escopos</em></figcaption>
    <br><br>
</figure>

O edifício representa a coleção de escopos aninhados do nosso programa. O primeiro andar do edifício representa o escopo em execução no momento. O último andar do edifício é o escopo global.

Você resolve uma referência de variável de *alvo* ou de *origem* procurando primeiro no andar atual e, se não encontrar, pegando o elevador para o próximo andar (ou seja, um escopo externo), procurando lá, depois no próximo, e assim por diante. Quando você chega ao último andar (o escopo global), ou você encontra o que procura, ou não encontra. Mas tem que parar de qualquer jeito.

## Continue a Conversa

A esta altura, você deve estar desenvolvendo modelos mentais mais ricos sobre o que é escopo e como a engine JS o determina e o usa a partir do seu código.

Antes de *continuar*, vá pegar algum código em um dos seus projetos e percorra essas conversas. Sério, fale em voz alta de verdade. Encontre um amigo e pratique cada papel com ele. Se algum de vocês se vir confuso ou tropeçando, gaste mais tempo revisando este material.

Enquanto avançamos (para cima) rumo ao próximo capítulo (mais externo), vamos explorar como os escopos léxicos de um programa são conectados em uma cadeia.
