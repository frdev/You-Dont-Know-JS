# You Don't Know JS Yet: Scope & Closures - 2ª Edição
# Apêndice A: Explorando Mais a Fundo

Vamos agora explorar uma série de nuances e bordas em torno de muitos dos tópicos cobertos no texto principal deste livro. Este apêndice é material opcional, de apoio.

Algumas pessoas acham que mergulhar fundo demais nos casos de borda sutis e nas opiniões divergentes não cria nada além de ruído e distração — supostamente, desenvolvedores são mais bem servidos ficando nos caminhos mais trilhados. Minha abordagem já foi criticada como impraticável e contraproducente. Eu entendo e respeito essa perspectiva, mesmo que não necessariamente a compartilhe.

Acredito que é melhor ser empoderado pelo conhecimento de como as coisas funcionam do que apenas passar por cima dos detalhes com suposições e falta de curiosidade. No fim das contas, você vai encontrar situações em que algo borbulha de uma parte que você não explorou. Em outras palavras, você não vai poder passar todo o seu tempo cavalgando na suave *trilha feliz*. Você não preferiria estar preparado para os inevitáveis solavancos do fora de estrada?

Estas discussões também serão mais fortemente influenciadas pelas minhas opiniões do que o texto principal, então tenha isso em mente enquanto consome e considera o que é apresentado. Este apêndice é um pouco como uma coleção de mini-posts de blog que elaboram sobre vários tópicos do livro. É longo e profundo no mato, então vá com calma e não corra por tudo aqui.

## Escopos Implícitos

Escopos às vezes são criados em lugares não óbvios. Na prática, esses escopos implícitos frequentemente não impactam o comportamento do seu programa, mas ainda assim é útil saber que eles estão acontecendo. Fique de olho nos seguintes escopos surpreendentes:

* Escopo de parâmetros
* Escopo do nome da função

### Escopo de Parâmetros

A metáfora da conversa no Capítulo 2 sugere que parâmetros de função são basicamente iguais a variáveis declaradas localmente no escopo da função. Mas isso nem sempre é verdade.

Considere:

```js
// escopo externo/global: VERMELHO(1)

function getStudentName(studentID) {
    // escopo da função: AZUL(2)

    // ..
}
```

Aqui, `studentID` é considerado um parâmetro "simples", então ele se comporta como um membro do escopo de função AZUL(2). Mas, se o mudarmos para um parâmetro não simples, tecnicamente esse deixa de ser o caso. Formas de parâmetro consideradas não simples incluem parâmetros com valores padrão, parâmetros rest (usando `...`) e parâmetros desestruturados.

Considere:

```js
// escopo externo/global: VERMELHO(1)

function getStudentName(/*AZUL(2)*/ studentID = 0) {
    // escopo da função: VERDE(3)

    // ..
}
```

Aqui, a lista de parâmetros essencialmente se torna seu próprio escopo, e o escopo da função fica então aninhado dentro *daquele* escopo.

Por quê? Que diferença isso faz? As formas de parâmetro não simples introduzem vários casos de borda, então a lista de parâmetros se torna seu próprio escopo para lidar com eles de forma mais eficaz.

Considere:

```js
function getStudentName(studentID = maxID, maxID) {
    // ..
}
```

Assumindo operações da esquerda para a direita, o padrão `= maxID` para o parâmetro `studentID` exige que `maxID` já exista (e já tenha sido inicializado). Este código produz um erro de TDZ (Capítulo 5). A razão é que `maxID` é declarado no escopo de parâmetros, mas ainda não foi inicializado por causa da ordem dos parâmetros. Se a ordem dos parâmetros for invertida, nenhum erro de TDZ ocorre:

```js
function getStudentName(maxID,studentID = maxID) {
    // ..
}
```

A complicação fica ainda mais *no mato* se introduzirmos uma expressão de função na posição de parâmetro padrão, que pode então criar sua própria closure (Capítulo 7) sobre parâmetros neste escopo implícito de parâmetros:

```js
function whatsTheDealHere(id,defaultID = () => id) {
    id = 5;
    console.log( defaultID() );
}

whatsTheDealHere(3);
// 5
```

Esse trecho provavelmente faz sentido, porque a arrow function `defaultID()` faz closure sobre o parâmetro/variável `id`, que então reatribuímos para `5`. Mas agora vamos introduzir uma definição de `id` que sombreia, no escopo da função:

```js
function whatsTheDealHere(id,defaultID = () => id) {
    var id = 5;
    console.log( defaultID() );
}

whatsTheDealHere(3);
// 3
```

Opa! O `var id = 5` está sombreando o parâmetro `id`, mas a closure da função `defaultID()` é sobre o parâmetro, não sobre a variável sombreadora no corpo da função. Isso prova que há uma bolha de escopo em torno da lista de parâmetros.

Mas fica ainda mais maluco do que isso!

```js
function whatsTheDealHere(id,defaultID = () => id) {
    var id;

    console.log(`local variable 'id': ${ id }`);
    console.log(
        `parameter 'id' (closure): ${ defaultID() }`
    );

    console.log("reassigning 'id' to 5");
    id = 5;

    console.log(`local variable 'id': ${ id }`);
    console.log(
        `parameter 'id' (closure): ${ defaultID() }`
    );
}

whatsTheDealHere(3);
// local variable 'id': 3   <--- Hã!? Estranho!
// parameter 'id' (closure): 3
// reassigning 'id' to 5
// local variable 'id': 5
// parameter 'id' (closure): 3
```

A parte estranha aqui é a primeira mensagem do console. Naquele momento, a variável local sombreadora `id` acabou de ser declarada com `var id`, o que o Capítulo 5 afirma que é tipicamente autoinicializado como `undefined` no topo do seu escopo. Por que não imprime `undefined`?

Neste caso de borda específico (por razões de compatibilidade legada), o JS não autoinicializa `id` como `undefined`, e sim com o valor do parâmetro `id` (`3`)!

Embora os dois `id` pareçam, naquele momento, ser uma única variável, eles ainda são, na verdade, separados (e em escopos separados). A atribuição `id = 5` torna a divergência observável, em que o parâmetro `id` permanece `3` e a variável local se torna `5`.

Meu conselho para evitar ser mordido por essas nuances esquisitas:

* Nunca sombreie parâmetros com variáveis locais

* Evite usar uma função de parâmetro padrão que faça closure sobre qualquer um dos parâmetros

Pelo menos agora você está ciente e pode tomar cuidado com o fato de que a lista de parâmetros é seu próprio escopo se algum dos parâmetros for não simples.

### Escopo do Nome da Função

Na seção "Escopo do Nome da Função", no Capítulo 3, afirmei que o nome de uma expressão de função é adicionado ao escopo da própria função. Lembre-se:

```js
var askQuestion = function ofTheTeacher(){
    // ..
};
```

É verdade que `ofTheTeacher` não é adicionado ao escopo envolvente (onde `askQuestion` é declarada), mas também não é *apenas* adicionado ao escopo da função, do jeito que você provavelmente está supondo. É outro caso de borda estranho de escopo implícito.

O identificador de nome de uma expressão de função fica em seu próprio escopo implícito, aninhado entre o escopo envolvente externo e o escopo principal interno da função.

Se `ofTheTeacher` estivesse no escopo da função, esperaríamos um erro aqui:

```js
var askQuestion = function ofTheTeacher(){
    // por que isto não é um erro de declaração duplicada?
    let ofTheTeacher = "Confused, yet?";
};
```

A forma de declaração `let` não permite redeclaração (veja o Capítulo 5). Mas isto é sombreamento perfeitamente legal, não redeclaração, porque os dois identificadores `ofTheTeacher` estão em escopos separados.

Você raramente vai se deparar com um caso em que o escopo do identificador de nome de uma função importe. Mas, de novo, é bom saber como esses mecanismos realmente funcionam. Para evitar ser mordido, nunca sombreie identificadores de nome de função.

## Funções Anônimas vs. Nomeadas

Como discutido no Capítulo 3, funções podem ser expressas em forma nomeada ou anônima. É muitíssimo mais comum usar a forma anônima, mas isso é uma boa ideia?

Enquanto você reflete sobre nomear suas funções, considere:

* A inferência de nome é incompleta
* Nomes léxicos permitem autorreferência
* Nomes são descrições úteis
* Arrow functions não têm nomes léxicos
* IIFEs também precisam de nomes

### Nomes Explícitos ou Inferidos?

Toda função no seu programa tem um propósito. Se não tem propósito, tire-a de lá, porque você está apenas desperdiçando espaço. Se ela *tem* um propósito, *existe* um nome para esse propósito.

Até aqui, muitos leitores provavelmente concordam comigo. Mas isso significa que devemos sempre colocar esse nome no código? Aqui é onde vou levantar mais do que algumas sobrancelhas. Eu digo, inequivocamente, sim!

Antes de tudo, ver "anonymous" aparecendo em stack traces simplesmente não ajuda tanto assim na depuração:

```js
btn.addEventListener("click",function(){
    setTimeout(function(){
        ["a",42].map(function(v){
            console.log(v.toUpperCase());
        });
    },100);
});
// Uncaught TypeError: v.toUpperCase is not a function
//     at myProgram.js:4
//     at Array.map (<anonymous>)
//     at myProgram.js:3
```

Eca. Compare com o que é reportado se eu der nomes às funções:

```js
btn.addEventListener("click",function onClick(){
    setTimeout(function waitAMoment(){
        ["a",42].map(function allUpper(v){
            console.log(v.toUpperCase());
        });
    },100);
});
// Uncaught TypeError: v.toUpperCase is not a function
//     at allUpper (myProgram.js:4)
//     at Array.map (<anonymous>)
//     at waitAMoment (myProgram.js:3)
```

Viu como os nomes `waitAMoment` e `allUpper` aparecem e dão ao stack trace informação/contexto mais úteis para a depuração? O programa fica mais depurável se usarmos nomes razoáveis para todas as nossas funções.

| NOTA: |
| :--- |
| O infeliz "&lt;anonymous>" que ainda aparece se refere ao fato de que a implementação de `Array.map(..)` não está presente no nosso programa, mas é embutida na engine JS. Não é por causa de nenhuma confusão que nosso programa introduz com atalhos de legibilidade. |

A propósito, vamos garantir que estamos na mesma página sobre o que é uma função nomeada:

```js
function thisIsNamed() {
    // ..
}

ajax("some.url",function thisIsAlsoNamed(){
   // ..
});

var notNamed = function(){
    // ..
};

makeRequest({
    data: 42,
    cb /* também não é um nome */: function(){
        // ..
    }
});

var stillNotNamed = function butThisIs(){
    // ..
};
```

"Mas espera!", você diz. Algumas dessas *são* nomeadas, certo!?

```js
var notNamed = function(){
    // ..
};

var config = {
    cb: function(){
        // ..
    }
};

notNamed.name;
// notNamed

config.cb.name;
// cb
```

Esses são chamados de nomes *inferidos*. Nomes inferidos são ok, mas eles não resolvem realmente toda a preocupação que estou discutindo.

### Nomes Faltando?

Sim, esses nomes inferidos podem aparecer em stack traces, o que é definitivamente melhor do que ver "anonymous". Mas...

```js
function ajax(url,cb) {
    console.log(cb.name);
}

ajax("some.url",function(){
    // ..
});
// ""
```

Ops. Expressões de `function` anônimas passadas como callbacks são incapazes de receber um nome inferido, então `cb.name` guarda apenas a string vazia `""`. A imensa maioria de todas as expressões de `function`, especialmente as anônimas, é usada como argumento de callback; nenhuma delas recebe um nome. Então depender da inferência de nome é, na melhor das hipóteses, incompleto.

E não são só os callbacks que ficam a desejar com a inferência:

```js
var config = {};

config.cb = function(){
    // ..
};

config.cb.name;
// ""

var [ noName ] = [ function(){} ];
noName.name
// ""
```

Qualquer atribuição de uma expressão de `function` que não seja uma *atribuição simples* também vai falhar na inferência de nome. Então, em outras palavras, a menos que você seja cuidadoso e intencional a respeito, essencialmente quase todas as expressões de `function` anônimas do seu programa, de fato, não vão ter nome nenhum.

A inferência de nome simplesmente... não é suficiente.

E, mesmo que uma expressão de `function` *receba* um nome inferido, isso ainda não conta como ser uma função nomeada de verdade.

### Quem sou eu?

Sem um identificador de nome léxico, a função não tem nenhuma forma interna de se referir a si mesma. A autorreferência é importante para coisas como recursão e tratamento de eventos:

```js
// quebrado
runOperation(function(num){
    if (num <= 1) return 1;
    return num * oopsNoNameToCall(num - 1);
});

// também quebrado
btn.addEventListener("click",function(){
   console.log("should only respond to one click!");
   btn.removeEventListener("click",oopsNoNameHere);
});
```

Deixar de fora o nome léxico do seu callback torna mais difícil se autorreferenciar de forma confiável. Você *poderia* declarar uma variável em um escopo envolvente que referencie a função, mas essa variável é *controlada* por aquele escopo envolvente — pode ser reatribuída etc. —, então não é tão confiável quanto a função ter sua própria autorreferência interna.

### Nomes São Descritores

Por fim, e acho que o mais importante de tudo, deixar de dar nome a uma função torna mais difícil para quem lê identificar qual é o propósito da função com uma olhada rápida. A pessoa precisa ler mais código, incluindo o código dentro da função e o código ao redor, fora dela, para descobrir.

Considere:

```js
[ 1, 2, 3, 4, 5 ].filter(function(v){
    return v % 2 == 1;
});
// [ 1, 3, 5 ]

[ 1, 2, 3, 4, 5 ].filter(function keepOnlyOdds(v){
    return v % 2 == 1;
});
// [ 1, 3, 5 ]
```

Simplesmente não há argumento razoável de que **omitir** o nome `keepOnlyOdds` do primeiro callback comunique mais eficazmente ao leitor o propósito desse callback. Você economizou 13 caracteres, mas perdeu informação importante de legibilidade. O nome `keepOnlyOdds` diz ao leitor, de forma bem clara e com uma primeira olhada rápida, o que está acontecendo.

A engine JS não se importa com o nome. Mas os leitores humanos do seu código se importam, e muito.

Um leitor consegue olhar `v % 2 == 1` e descobrir o que aquilo faz? Claro. Mas ele precisa inferir o propósito (e o nome) executando o código mentalmente. Mesmo uma breve pausa para fazer isso desacelera a leitura do código. Um bom nome descritivo torna esse processo quase sem esforço e instantâneo.

Pense assim: quantas vezes o autor deste código precisa descobrir o propósito de uma função antes de adicionar o nome ao código? Umas uma vez. Talvez duas ou três, se precisar ajustar o nome. Mas quantas vezes os leitores deste código vão ter que descobrir o nome/propósito? Toda vez que esta linha for lida. Centenas de vezes? Milhares? Mais?

Não importa o tamanho ou a complexidade da função, minha afirmação é: o autor deveria pensar em um bom nome descritivo e adicioná-lo ao código. Até funções de uma linha em instruções `map(..)` e `then(..)` deveriam ser nomeadas:

```js
lookupTheRecords(someData)
.then(function extractSalesRecords(resp){
   return resp.allSales;
})
.then(storeRecords);
```

O nome `extractSalesRecords` diz ao leitor o propósito deste manipulador de `then(..)` *melhor* do que apenas inferir esse propósito executando mentalmente `return resp.allSales`.

A única desculpa para não incluir um nome em uma função é preguiça (não querer digitar alguns caracteres a mais) ou falta de criatividade (não conseguir pensar em um bom nome). Se você não consegue pensar em um bom nome, provavelmente ainda não entende a função e seu propósito. A função talvez esteja mal projetada, ou faça coisas demais, e deveria ser retrabalhada. Uma vez que você tem uma função bem projetada e com propósito único, seu nome apropriado deve se tornar evidente.

Aqui vai um truque que uso: ao escrever uma função pela primeira vez, se não entendo totalmente seu propósito e não consigo pensar em um bom nome, simplesmente uso `TODO` como nome. Assim, mais tarde, ao revisar meu código, é provável que eu encontre esses marcadores de nome, e fico mais inclinado (e mais preparado!) a voltar e descobrir um nome melhor, em vez de simplesmente deixar como `TODO`.

Todas as funções precisam de nomes. Cada uma delas. Sem exceções. Qualquer nome que você omite torna o programa mais difícil de ler, mais difícil de depurar, mais difícil de estender e manter depois.

### Arrow Functions

Arrow functions são **sempre** anônimas, mesmo se (raramente) forem usadas de uma forma que lhes dê um nome inferido. Acabei de gastar várias páginas explicando por que funções anônimas são uma má ideia, então você provavelmente consegue adivinhar o que penso sobre arrow functions.

Não as use como substituto geral de funções regulares. Elas são mais concisas, sim, mas essa brevidade vem ao custo de omitir delimitadores visuais importantes, que ajudam nossos cérebros a analisar rapidamente o que estamos lendo. E, no que diz respeito a esta discussão, elas são anônimas, o que também as torna piores para a legibilidade por esse ângulo.

Arrow functions têm um propósito, mas esse propósito não é economizar digitação. Arrow functions têm comportamento de *this léxico*, o que está um tanto além dos limites da nossa discussão neste livro.

Resumidamente: arrow functions não definem um identificador/palavra-chave `this` de forma alguma. Se você usa um `this` dentro de uma arrow function, ele se comporta exatamente como qualquer outra referência a variável, ou seja, a cadeia de escopos é consultada para encontrar um escopo de função (função não arrow) em que ele *esteja* definido, e usar esse.

Em outras palavras, arrow functions tratam `this` como qualquer outra variável léxica.

Se você está acostumado a gambiarras como `var self = this`, ou se prefere chamar `.bind(this)` em expressões de `function` internas só para forçá-las a herdar um `this` de uma função externa, como se fosse uma variável léxica, então as arrow functions `=>` são absolutamente a melhor opção. Elas foram projetadas especificamente para corrigir esse problema.

Então, nos raros casos em que você precisa de *this léxico*, use uma arrow function. É a melhor ferramenta para esse trabalho. Mas esteja ciente de que, ao fazer isso, você está aceitando as desvantagens de uma função anônima. Você deveria empregar esforço adicional para mitigar o *custo* de legibilidade, como nomes de variáveis mais descritivos e comentários no código.

### Variações de IIFE

Todas as funções deveriam ter nomes. Eu disse isso algumas vezes, certo!? Isso inclui IIFEs.

```js
(function(){
    // não faça isso!
})();

(function doThisInstead(){
    // ..
})();
```

Como chegamos a um nome para uma IIFE? Identifique para que a IIFE está ali. Por que você precisa de um escopo naquele ponto? Está escondendo uma variável de cache para registros de alunos?

```js
var getStudents = (function StoreStudentRecords(){
    var studentRecords = [];

    return function getStudents() {
        // ..
    }
})();
```

Nomeei a IIFE como `StoreStudentRecords` porque é isso que ela faz: armazenar registros de alunos. Toda IIFE deveria ter um nome. Sem exceções.

IIFEs são tipicamente definidas colocando `( .. )` em torno da expressão de `function`, como mostrado nos trechos anteriores. Mas essa não é a única forma de definir uma IIFE. Tecnicamente, a única razão pela qual usamos aquele primeiro par de `( .. )` ao redor é justamente para que a palavra-chave `function` não esteja em posição de se qualificar como declaração de `function` para o parser do JS. Mas há outras formas sintáticas de evitar ser analisado como declaração:

```js
!function thisIsAnIIFE(){
    // ..
}();

+function soIsThisOne(){
    // ..
}();

~function andThisOneToo(){
    // ..
}();
```

O `!`, o `+`, o `~` e vários outros operadores unários (operadores com um operando) podem ser colocados na frente de `function` para transformá-la em expressão. Então a chamada final `()` é válida, o que faz dela uma IIFE.

Na verdade, eu meio que gosto de usar o operador unário `void` ao definir uma IIFE independente:

```js
void function yepItsAnIIFE() {
    // ..
}();
```

O benefício do `void` é que ele comunica claramente, no começo da função, que essa IIFE não vai retornar valor algum.

Seja como você definir suas IIFEs, mostre a elas algum carinho dando nomes a elas.

## Hoisting: Funções e Variáveis

O Capítulo 5 articulou tanto o *function hoisting* quanto o *hoisting de variáveis*. Como o hoisting é frequentemente citado como um erro no design do JS, quis explorar brevemente por que ambas as formas de hoisting *podem* ser benéficas e ainda devem ser consideradas.

Dê ao hoisting um nível mais profundo de consideração, avaliando os méritos de:

* Código executável primeiro, declarações de função por último
* Colocação semântica de declarações de variáveis

### Function Hoisting

Para revisar, este programa funciona por causa do *function hoisting*:

```js
getStudents();

// ..

function getStudents() {
    // ..
}
```

A declaração de `function` sofre hoisting durante a compilação, o que significa que `getStudents` é um identificador declarado para todo o escopo. Além disso, o identificador `getStudents` é autoinicializado com a referência da função, também no início do escopo.

Por que isso é útil? A razão pela qual prefiro aproveitar o *function hoisting* é que ele coloca o código *executável* de qualquer escopo no topo, e quaisquer declarações adicionais (funções) abaixo. Isso significa que é mais fácil encontrar o código que vai rodar em determinada área, em vez de ter que rolar e rolar, torcendo para achar um `}` final marcando o fim de um escopo/função em algum lugar.

Eu aproveito esse posicionamento invertido em todos os níveis de escopo:

```js
getStudents();

// *************

function getStudents() {
    var whatever = doSomething();

    // outras coisas

    return whatever;

    // *************

    function doSomething() {
        // ..
    }
}
```

Quando abro um arquivo assim pela primeira vez, a primeiríssima linha é código executável que dispara seu comportamento. Isso é muito fácil de identificar! Então, se algum dia eu precisar ir encontrar e inspecionar `getStudents()`, gosto que sua primeira linha também seja código executável. Só se eu precisar ver os detalhes de `doSomething()` é que vou encontrar sua definição lá embaixo.

Em outras palavras, acho que o *function hoisting* torna o código mais legível por meio de uma ordem de leitura fluida e progressiva, de cima para baixo.

### Hoisting de Variáveis

E quanto ao *hoisting de variáveis*?

Embora `let` e `const` sofram hoisting, você não pode usar essas variáveis na sua TDZ (veja o Capítulo 5). Então a discussão a seguir se aplica apenas a declarações `var`. Antes de continuar, vou admitir: em quase todos os casos, concordo completamente que *hoisting de variáveis* é uma má ideia:

```js
pleaseDontDoThis = "bad idea";

// muito depois
var pleaseDontDoThis;
```

Embora esse tipo de ordenação invertida fosse útil para o *function hoisting*, aqui eu acho que ela geralmente torna o código mais difícil de raciocinar.

Mas há uma exceção que encontrei, de forma um tanto rara, na minha própria codificação. Tem a ver com onde coloco minhas declarações `var` dentro de uma definição de módulo CommonJS.

Veja como eu tipicamente estruturo minhas definições de módulo no Node:

```js
// dependências
var aModuleINeed = require("very-helpful");
var anotherModule = require("kinda-helpful");

// API pública
var publicAPI = Object.assign(module.exports,{
    getStudents,
    addStudents,
    // ..
});

// ********************************
// implementação privada

var cache = { };
var otherData = [ ];

function getStudents() {
    // ..
}

function addStudents() {
    // ..
}
```

Note como as variáveis `cache` e `otherData` estão na seção "privada" do layout do módulo. Isso porque não planejo expô-las publicamente. Então organizo o módulo de modo que fiquem junto aos outros detalhes ocultos de implementação do módulo.

Mas tive alguns casos raros em que precisei que as atribuições desses valores acontecessem *acima*, antes de eu declarar a API pública exportada do módulo. Por exemplo:

```js
// API pública
var publicAPI = Object.assign(module.exports,{
    getStudents,
    addStudents,
    refreshData: refreshData.bind(null,cache)
});
```

Preciso que a variável `cache` já tenha recebido um valor, porque esse valor é usado na inicialização da API pública (a aplicação parcial com `.bind(..)`).

Eu deveria simplesmente mover o `var cache = { .. }` para o topo, acima dessa inicialização da API pública? Bem, talvez. Mas então fica menos óbvio que `var cache` é um detalhe *privado* de implementação. Eis o meio-termo que eu (raramente) usei:

```js
cache = {};   // usado aqui, mas declarado abaixo

// API pública
var publicAPI = Object.assign(module.exports,{
    getStudents,
    addStudents,
    refreshData: refreshData.bind(null,cache)
});

// ********************************
// implementação privada

var cache /* = {}*/;
```

Viu o *hoisting de variáveis*? Declarei o `cache` lá embaixo, onde ele pertence logicamente, mas, neste caso raro, eu o usei antes, mais acima, na área em que sua inicialização é necessária. Até deixei uma dica do valor que é atribuído a `cache` em um comentário de código.

Esse é literalmente o único caso que já encontrei para aproveitar o *hoisting de variáveis* e atribuir uma variável mais cedo em um escopo do que sua declaração. Mas acho que é uma exceção razoável de se empregar com cautela.

## Em Defesa do `var`

Falando em *hoisting de variáveis*, vamos ter uma conversa franca por um instante sobre `var`, um vilão favorito que os devs adoram culpar por muitos dos males do desenvolvimento em JS. No Capítulo 5, exploramos `let`/`const` e prometemos revisitar onde `var` se encaixa em toda essa mistura.

Enquanto exponho o caso, não perca:

* `var` nunca foi quebrado
* `let` é seu amigo
* `const` tem utilidade limitada
* O melhor dos dois mundos: `var` *e* `let`

### Não Jogue o `var` Fora

`var` está bem e funciona muito bem. Está por aí há 25 anos, e vai continuar por aí, útil e funcional, por mais 25 anos ou mais. Afirmações de que `var` é quebrado, obsoleto, ultrapassado, perigoso ou mal projetado são modismo sem fundamento.

Isso significa que `var` é o declarador certo para toda e qualquer declaração no seu programa? Certamente não. Mas ele ainda tem seu lugar nos seus programas. Recusar-se a usá-lo porque alguém da equipe escolheu uma opinião agressiva de linter que engasga com `var` é cortar o próprio nariz para irritar o rosto.

Ok, agora que já te deixei realmente irritado, deixe-me tentar explicar minha posição.

Para constar, sou fã de `let`, para declarações com escopo de bloco. Eu realmente não gosto da TDZ e acho que aquilo foi um erro. Mas o `let` em si é ótimo. Eu o uso com frequência. De fato, provavelmente o uso tanto quanto, ou mais do que, uso `var`.

### `const`-antemente Confuso

`const`, por outro lado, eu não uso com tanta frequência. Não vou cavar todas as razões, mas se resume ao fato de `const` não *carregar seu próprio peso*. Ou seja, embora haja um pequenino benefício em `const` em alguns casos, esse benefício é superado pela longa história de problemas em torno da confusão com `const` em uma variedade de linguagens, muito antes de ele sequer aparecer no JS.

`const` finge criar valores que não podem ser mutados — um equívoco extremamente comum em comunidades de desenvolvedores de muitas linguagens —, ao passo que o que ele realmente faz é impedir a reatribuição.

```js
const studentIDs = [ 14, 73, 112 ];

// depois

studentIDs.push(6);   // opa, espera... o quê!?
```

Usar um `const` com um valor mutável (como um array ou objeto) é pedir para que um desenvolvedor futuro (ou leitor do seu código) caia na armadilha que você armou, que foi ele não saber, ou meio que esquecer, que *imutabilidade de valor* não é, de forma alguma, a mesma coisa que *imutabilidade de atribuição*.

Eu simplesmente acho que não deveríamos armar essas armadilhas. A única vez em que uso `const` é quando estou atribuindo um valor já imutável (como `42` ou `"Hello, friends!"`) e quando ele é claramente uma "constante" no sentido de ser um marcador nomeado para um valor literal, com fins semânticos. É para isso que `const` é mais bem usado. Isso é bem raro no meu código, porém.

Se a reatribuição de variáveis fosse um grande problema, então `const` seria mais útil. Mas reatribuição de variáveis simplesmente não é um problema tão grande em termos de causar bugs. Há uma longa lista de coisas que levam a bugs em programas, mas "reatribuição acidental" está bem, bem lá embaixo nessa lista.

Combine isso com o fato de que `const` (e `let`) deveriam ser usados em blocos, e blocos deveriam ser curtos, e você tem uma área realmente pequena do seu código em que uma declaração `const` sequer é aplicável. Um `const` na linha 1 do seu bloco de dez linhas só te diz algo sobre as próximas nove linhas. E o que ele te diz já é óbvio ao dar uma olhada nessas nove linhas: a variável nunca está do lado esquerdo de um `=`; ela não é reatribuída.

É isso, é tudo o que `const` realmente faz. Fora isso, não é muito útil. Colocado contra a confusão significativa entre imutabilidade de valor e de atribuição, `const` perde muito do seu brilho.

Um `let` (ou `var`!) que nunca é reatribuído já é, comportamentalmente, uma "constante", mesmo sem ter a garantia do compilador. Isso é bom o bastante na maioria dos casos.

### `var` *e* `let`

Na minha cabeça, `const` é bem raramente útil, então esta é apenas uma corrida de dois cavalos entre `let` e `var`. Mas também não é bem uma corrida, porque não precisa haver apenas um vencedor. Ambos podem vencer... corridas diferentes.

O fato é que você deveria usar tanto `var` quanto `let` nos seus programas. Eles não são intercambiáveis: você não deveria usar `var` onde um `let` é pedido, mas também não deveria usar `let` onde `var` é mais apropriado.

Então, onde ainda devemos usar `var`? Sob quais circunstâncias ele é uma escolha melhor do que `let`?

Primeiro, eu sempre uso `var` no escopo de nível superior de qualquer função, independentemente de ser no começo, no meio ou no fim da função. Também uso `var` no escopo global, embora eu tente minimizar o uso do escopo global.

Por que usar `var` para escopo de função? Porque é exatamente isso que `var` faz. Literalmente não há ferramenta melhor para o trabalho de dar escopo de função a uma declaração do que um declarador que, por 25 anos, faz exatamente isso.

Você *poderia* usar `let` nesse escopo de nível superior, mas ele não é a melhor ferramenta para esse trabalho. Também acho que, se você usa `let` em todo lugar, fica menos óbvio quais declarações foram projetadas para ser localizadas e quais se destinam a ser usadas em toda a função.

Em contraste, eu raramente uso um `var` dentro de um bloco. É para isso que `let` serve. Use a melhor ferramenta para o trabalho. Se você vê um `let`, isso te diz que está lidando com uma declaração localizada. Se você vê `var`, isso te diz que está lidando com uma declaração de função inteira. Simples assim.

```js
function getStudents(data) {
    var studentRecords = [];

    for (let record of data.records) {
        let id = `student-${ record.id }`;
        studentRecords.push({
            id,
            record.name
        });
    }

    return studentRecords;
}
```

A variável `studentRecords` se destina a uso em toda a função. `var` é o melhor declarador para dizer isso ao leitor. Em contraste, `record` e `id` se destinam a uso apenas no escopo mais estreito da iteração do laço, então `let` é a melhor ferramenta para esse trabalho.

Além desse argumento semântico da *melhor ferramenta*, `var` tem algumas outras características que, em certas circunstâncias limitadas, o tornam mais poderoso.

Um exemplo é quando um laço está usando exclusivamente uma variável, mas sua cláusula condicional não consegue ver declarações com escopo de bloco dentro da iteração:

```js
function commitAction() {
    do {
        let result = commit();
        var done = result && result.code == 1;
    } while (!done);
}
```

Aqui, `result` é claramente usada apenas dentro do bloco, então usamos `let`. Mas `done` é um pouco diferente. Ela só é útil para o laço, mas a cláusula `while` não consegue ver declarações `let` que aparecem dentro do laço. Então fazemos um meio-termo e usamos `var`, para que `done` sofra hoisting até o escopo externo, onde possa ser vista.

A alternativa — declarar `done` fora do laço — a separa de onde ela é usada pela primeira vez e exige ou escolher um valor padrão para atribuir ou, pior, deixá-la sem atribuição, parecendo ambígua para o leitor. Acho que `var` dentro do laço é preferível aqui.

Outra característica útil do `var` é vista com declarações dentro de blocos não intencionais. Blocos não intencionais são blocos criados porque a sintaxe exige um bloco, mas em que a intenção do desenvolvedor não é realmente criar um escopo localizado. A melhor ilustração de escopo não intencional é a instrução `try..catch`:

```js
function getStudents() {
    try {
        // não é realmente um escopo de bloco
        var records = fromCache("students");
    }
    catch (err) {
        // ops, recorre a um padrão
        var records = [];
    }
    // ..
}
```

Há outras formas de estruturar este código, sim. Mas eu acho que esta é a *melhor* forma, considerando os vários trade-offs.

Não quero declarar `records` (com `var` ou `let`) fora do bloco `try` e então atribuir a ela em um ou nos dois blocos. Prefiro que declarações iniciais estejam sempre o mais próximo possível (idealmente, na mesma linha) do primeiro uso da variável. Neste exemplo simples, seria uma distância de apenas algumas linhas, mas em código real isso pode crescer para muitas linhas. Quanto maior o intervalo, mais difícil é descobrir a qual variável de qual escopo você está atribuindo. `var` usado na própria atribuição torna isso menos ambíguo.

Note também que usei `var` tanto no bloco `try` quanto no `catch`. Isso porque quero sinalizar ao leitor que, não importa qual caminho seja tomado, `records` sempre é declarada. Tecnicamente, isso funciona porque `var` sofre hoisting uma vez para o escopo da função. Mas ainda é um bom sinal semântico para lembrar ao leitor o que qualquer um dos `var` garante. Se `var` fosse usado apenas em um dos blocos, e você estivesse lendo apenas o outro bloco, não descobriria tão facilmente de onde `records` estava vindo.

Este é, na minha opinião, um pequeno superpoder do `var`. Ele não só pode escapar dos blocos não intencionais de `try..catch`, como também pode aparecer múltiplas vezes no escopo de uma função. Você não pode fazer isso com `let`. Não é ruim; é, na verdade, um recurso um pouco útil. Pense no `var` mais como uma anotação declarativa que te lembra, a cada uso, de onde a variável vem. "Ah, é mesmo, ela pertence à função inteira."

Esse superpoder de anotação repetida é útil em outros casos:

```js
function getStudents() {
    var data = [];

    // faz algo com data
    // .. mais 50 linhas de código ..

    // puramente uma anotação para nos lembrar
    var data;

    // usa data de novo
    // ..
}
```

O segundo `var data` não está redeclarando `data`; está apenas anotando, para benefício dos leitores, que `data` é uma declaração de função inteira. Assim, o leitor não precisa rolar mais de 50 linhas de código para encontrar a declaração inicial.

Eu fico perfeitamente confortável em reutilizar variáveis para múltiplos propósitos ao longo do escopo de uma função. Também fico perfeitamente confortável com dois usos de uma variável separados por muitas linhas de código. Nos dois casos, a capacidade de "redeclarar" (anotar) com segurança usando `var` ajuda a garantir que eu consiga dizer de onde meu `data` vem, não importa onde eu esteja na função.

De novo, tristemente, `let` não pode fazer isso.

Há outras nuances e cenários em que `var` acaba oferecendo alguma ajuda, mas não vou me alongar mais no ponto. A lição é que `var` pode ser útil nos nossos programas ao lado de `let` (e do eventual `const`). Você está disposto a usar criativamente as ferramentas que a linguagem JS fornece para contar uma história mais rica aos seus leitores?

Não jogue fora uma ferramenta útil como `var` só porque alguém te envergonhou fazendo você pensar que ela não era mais legal. Não evite `var` porque você se confundiu uma vez, anos atrás. Aprenda essas ferramentas e use cada uma naquilo em que ela é melhor.

## Qual é a Dessa TDZ?

A TDZ (zona morta temporal) foi explicada no Capítulo 5. Ilustramos como ela ocorre, mas passamos por cima de qualquer explicação sobre *por que* foi necessário introduzi-la, para começo de conversa. Vamos olhar brevemente as motivações da TDZ.

Algumas migalhas de pão na história de origem da TDZ:

* `const`s nunca deveriam mudar
* É tudo uma questão de tempo
* `let` deveria se comportar mais como `const` ou como `var`?

### Onde Tudo Começou

A TDZ vem do `const`, na verdade.

Durante os primeiros trabalhos de desenvolvimento do ES6, o TC39 teve que decidir se `const` (e `let`) sofreriam hoisting para o topo de seus blocos. Eles decidiram que essas declarações sofreriam hoisting, de forma parecida com `var`. Se não fosse assim, acho que parte do medo era a confusão com sombreamento no meio do escopo, como:

```js
let greeting = "Hi!";

{
    // o que deveria ser impresso aqui?
    console.log(greeting);

    // .. um monte de linhas de código ..

    // agora sombreando a variável `greeting`
    let greeting = "Hello, friends!";

    // ..
}
```

O que deveríamos fazer com aquela instrução `console.log(..)`? Faria algum sentido para os devs JS ela imprimir "Hi!"? Parece que isso poderia ser uma pegadinha, o sombreamento valer apenas para a segunda metade do bloco, mas não para a primeira. Isso não é um comportamento muito intuitivo nem "JS-like". Então `let` e `const` têm que sofrer hoisting para o topo do bloco, visíveis em toda a sua extensão.

Mas, se `let` e `const` sofrem hoisting para o topo do bloco (como `var` sofre hoisting para o topo de uma função), por que `let` e `const` não se autoinicializam (como `undefined`) do jeito que `var` faz? Eis a principal preocupação:

```js
{
    // o que deveria ser impresso aqui?
    console.log(studentName);

    // depois

    const studentName = "Frank";

    // ..
}
```

Vamos imaginar que `studentName` não só sofresse hoisting para o topo deste bloco, como também fosse autoinicializada como `undefined`. Na primeira metade do bloco, `studentName` poderia ser observada com o valor `undefined`, como na nossa instrução `console.log(..)`. Uma vez alcançada a instrução `const studentName = ..`, agora `studentName` recebe `"Frank"`. Daquele ponto em diante, `studentName` nunca mais pode ser reatribuída.

Mas é estranho ou surpreendente que uma constante tenha, observavelmente, dois valores diferentes, primeiro `undefined` e depois `"Frank"`? Isso parece ir contra o que achamos que uma `const`ante significa; ela só deveria ser observável com um valor.

Então... agora temos um problema. Não podemos autoinicializar `studentName` como `undefined` (nem com qualquer outro valor, aliás). Mas a variável precisa existir em todo o escopo. O que fazemos com o período de tempo entre o momento em que ela passa a existir (início do escopo) e o momento em que recebe seu valor?

Chamamos esse período de tempo de "zona morta", como em "zona morta temporal" (TDZ). Para evitar confusão, determinou-se que qualquer tipo de acesso a uma variável enquanto ela está na sua TDZ é ilegal e deve resultar no erro de TDZ.

Ok, essa linha de raciocínio faz algum sentido, devo admitir.

### Quem `let` a TDZ Escapar?

Mas isso é só o `const`. E quanto ao `let`?

Bem, o TC39 tomou a decisão: já que precisamos de uma TDZ para `const`, podemos muito bem ter uma TDZ para `let` também. *De fato, se fizermos let ter uma TDZ, então desencorajamos todo aquele feio hoisting de variáveis que as pessoas fazem.* Então havia uma perspectiva de consistência e, talvez, um pouco de engenharia social para mudar o comportamento dos desenvolvedores.

Meu contra-argumento seria: se você está favorecendo consistência, seja consistente com `var` em vez de `const`; `let` é definitivamente mais parecido com `var` do que com `const`. Isso é especialmente verdade porque eles já haviam escolhido consistência com `var` em toda aquela questão do hoisting-para-o-topo-do-escopo. Deixe `const` ser seu próprio caso único, com uma TDZ, e que a resposta para a TDZ seja puramente: apenas evite a TDZ declarando sempre suas constantes no topo do escopo. Acho que isso teria sido mais razoável.

Mas, ai de nós, não foi assim que ficou. `let` tem uma TDZ porque `const` precisa de uma TDZ, porque `let` e `const` imitam `var` no seu hoisting para o topo do escopo (de bloco). Aí está. Circular demais? Leia de novo algumas vezes.

## Callbacks Síncronos Ainda São Closures?

O Capítulo 7 apresentou dois modelos diferentes para encarar a closure:

* Closure é uma instância de função lembrando suas variáveis externas mesmo enquanto essa função é passada adiante e **invocada em** outros escopos.

* Closure é uma instância de função e seu ambiente de escopo sendo preservados no lugar, enquanto quaisquer referências a ela são passadas adiante e **invocadas a partir de** outros escopos.

Esses modelos não são absurdamente divergentes, mas abordam a partir de uma perspectiva diferente. E essa perspectiva diferente muda o que identificamos como closure.

Não se perca seguindo esta trilha de coelho por closures e callbacks:

* Chamar de volta para o quê (ou para onde)?
* Talvez "callback síncrono" não seja o melhor rótulo
* Funções ***IIF*** não se movem, por que precisariam de closure?
* Adiar no tempo é a chave da closure

### O que é um Callback?

Antes de revisitarmos a closure, deixe-me gastar um breve momento tratando da palavra "callback". É uma norma geralmente aceita que dizer "callback" é sinônimo tanto de *callbacks assíncronos* quanto de *callbacks síncronos*. Não acho que eu concorde que isso seja uma boa ideia, então quero explicar por quê e propor que nos afastemos disso rumo a outro termo.

Vamos primeiro considerar um *callback assíncrono*, uma referência de função que será invocada em algum ponto futuro, *mais tarde*. O que "callback" significa, nesse caso?

Significa que o código atual terminou ou pausou, suspendeu a si mesmo, e que, quando a função em questão for invocada depois, a execução vai entrar de volta no programa suspenso, retomando-o. Especificamente, o ponto de reentrada é o código que foi embrulhado na referência de função:

```js
setTimeout(function waitForASecond(){
    // é aqui que o JS deveria chamar de volta
    // para dentro do programa quando o timer terminar
},1000);

// é aqui que o programa atual termina
// ou é suspenso
```

Nesse contexto, "chamar de volta" faz muito sentido. A engine JS está retomando nosso programa suspenso ao *chamar de volta para dentro* dele em um local específico. Ok, então um callback é assíncrono.

### Callback Síncrono?

Mas e quanto aos *callbacks síncronos*? Considere:

```js
function getLabels(studentIDs) {
    return studentIDs.map(
        function formatIDLabel(id){
            return `Student ID: ${
               String(id).padStart(6)
            }`;
        }
    );
}

getLabels([ 14, 73, 112, 6 ]);
// [
//    "Student ID: 000014",
//    "Student ID: 000073",
//    "Student ID: 000112",
//    "Student ID: 000006"
// ]
```

Deveríamos nos referir a `formatIDLabel(..)` como um callback? O utilitário `map(..)` está realmente *chamando de volta* para dentro do nosso programa ao invocar a função que fornecemos?

Não há nada *para onde chamar de volta*, por assim dizer, porque o programa não pausou nem saiu. Estamos passando uma função (referência) de uma parte do programa para outra parte do programa, e então ela é imediatamente invocada.

Há outros termos estabelecidos que podem corresponder ao que estamos fazendo — passar uma função (referência) para que outra parte do programa possa invocá-la em nosso nome. Você pode pensar nisso como *Injeção de Dependência* (DI) ou *Inversão de Controle* (IoC).

DI pode ser resumida como passar parte(s) necessária(s) de funcionalidade a outra parte do programa, para que ela possa invocá-las e completar seu trabalho. Essa é uma descrição decente para a chamada de `map(..)` acima, não é? O utilitário `map(..)` sabe iterar sobre os valores da lista, mas não sabe o que *fazer* com esses valores. É por isso que passamos a ele a função `formatIDLabel(..)`. Passamos a dependência.

IoC é um conceito bem parecido e relacionado. Inversão de controle significa que, em vez de a área atual do seu programa controlar o que está acontecendo, você entrega o controle a outra parte do programa. Embrulhamos a lógica de computar uma string de rótulo na função `formatIDLabel(..)` e então entregamos o controle de invocação ao utilitário `map(..)`.

Notavelmente, Martin Fowler cita IoC como a diferença entre um framework e uma biblioteca: com uma biblioteca, você chama as funções dela; com um framework, ele chama as suas funções. [^fowlerIOC]

No contexto da nossa discussão, tanto DI quanto IoC poderiam funcionar como rótulo alternativo para um *callback síncrono*.

Mas eu tenho uma sugestão diferente. Vamos nos referir a (as funções antes conhecidas como) *callbacks síncronos* como *funções interinvocadas* (IIFs, de *inter-invoked functions*). Sim, exatamente, estou brincando com as IIFEs. Esse tipo de função é *interinvocado*, ou seja: outra entidade as invoca, em oposição às IIFEs, que invocam a si mesmas imediatamente.

Qual é a relação entre um *callback assíncrono* e uma IIF? Um *callback assíncrono* é uma IIF invocada de forma assíncrona em vez de síncrona.

### Closure Síncrona?

Agora que rerrotulamos os *callbacks síncronos* como IIFs, podemos voltar à nossa pergunta principal: IIFs são um exemplo de closure? Obviamente, a IIF teria que referenciar variável(is) de um escopo externo para ter qualquer chance de ser uma closure. A IIF `formatIDLabel(..)` de antes não referencia nenhuma variável fora do seu próprio escopo, então definitivamente não é uma closure.

E quanto a uma IIF que tem, sim, referências externas? Isso é closure?

```js
function printLabels(labels) {
    var list = document.getElementById("labelsList");

    labels.forEach(
        function renderLabel(label){
            var li = document.createElement("li");
            li.innerText = label;
            list.appendChild(li);
        }
    );
}
```

A IIF interna `renderLabel(..)` referencia `list` do escopo envolvente, então é uma IIF que *poderia* ter closure. Mas é aqui que a definição/modelo que escolhemos para closure importa:

* Se `renderLabel(..)` é uma **função que é passada para outro lugar**, e essa função é então invocada, então sim, `renderLabel(..)` está exercitando uma closure, porque a closure é o que preservou seu acesso à sua cadeia de escopos original.

* Mas se, como no modelo conceitual alternativo do Capítulo 7, `renderLabel(..)` fica no lugar, e apenas uma referência a ela é passada para `forEach(..)`, há alguma necessidade de closure para preservar a cadeia de escopos de `renderLabel(..)`, enquanto ela executa sincronamente bem dentro do seu próprio escopo?

Não. Isso é apenas escopo léxico normal.

Para entender por quê, considere esta forma alternativa de `printLabels(..)`:

```js
function printLabels(labels) {
    var list = document.getElementById("labelsList");

    for (let label of labels) {
        // apenas uma chamada de função normal no seu
        // próprio escopo, certo? Isso não é closure!
        renderLabel(label);
    }

    // **************

    function renderLabel(label) {
        var li = document.createElement("li");
        li.innerText = label;
        list.appendChild(li);
    }
}
```

Estas duas versões de `printLabels(..)` são essencialmente a mesma coisa.

A última definitivamente não é um exemplo de closure, ao menos não em nenhum sentido útil ou observável. É apenas escopo léxico. A primeira versão, com `forEach(..)` chamando nossa referência de função, é essencialmente a mesma coisa. Também não é closure, mas sim uma boa e velha chamada de função com escopo léxico.

### Adiando para a Closure

A propósito, o Capítulo 7 mencionou brevemente a aplicação parcial e o currying (que *dependem*, sim, de closure!). Este é um cenário interessante em que o currying manual pode ser usado:

```js
function printLabels(labels) {
    var list = document.getElementById("labelsList");
    var renderLabel = renderTo(list);

    // definitivamente closure desta vez!
    labels.forEach( renderLabel );

    // **************

    function renderTo(list) {
        return function createLabel(label){
            var li = document.createElement("li");
            li.innerText = label;
            list.appendChild(li);
        };
    }
}
```

A função interna `createLabel(..)`, que atribuímos a `renderLabel`, faz closure sobre `list`, então a closure está definitivamente sendo utilizada.

A closure nos permite lembrar `list` para depois, enquanto adiamos a execução da lógica real de criação de rótulos, da chamada a `renderTo(..)` para as invocações subsequentes da IIF `createLabel(..)` por `forEach(..)`. Isso pode ser apenas um breve momento aqui, mas qualquer quantidade de tempo poderia se passar, já que a closure faz a ponte de chamada em chamada.

## Variações do Módulo Clássico

O Capítulo 8 explicou o padrão de módulo clássico, que pode se parecer com isto:

```js
var StudentList = (function defineModule(Student){
    var elems = [];

    var publicAPI = {
        renderList() {
            // ..
        }
    };

    return publicAPI;

})(Student);
```

Note que estamos passando `Student` (outra instância de módulo) como dependência. Mas há muitas variações úteis dessa forma de módulo que você pode encontrar. Algumas dicas para reconhecer essas variações:

* O módulo conhece sua própria API?
* Mesmo que usemos um carregador de módulos sofisticado, ainda é só um módulo clássico
* Alguns módulos precisam funcionar universalmente

### Cadê Minha API?

Primeiro, a maioria dos módulos clássicos não define nem usa um `publicAPI` do jeito que mostrei neste código. Em vez disso, eles tipicamente se parecem com:

```js
var StudentList = (function defineModule(Student){
    var elems = [];

    return {
        renderList() {
            // ..
        }
    };

})(Student);
```

A única diferença aqui é retornar diretamente o objeto que serve como API pública do módulo, em vez de primeiro salvá-lo em uma variável interna `publicAPI`. É de longe assim que a maioria dos módulos clássicos é definida.

Mas eu prefiro fortemente, e sempre uso, a forma anterior com `publicAPI`. Duas razões:

* `publicAPI` é um descritor semântico que ajuda na legibilidade, tornando mais óbvio qual é o propósito do objeto.

* Armazenar uma variável interna `publicAPI` que referencia o mesmo objeto de API pública externa retornado pode ser útil se você precisar acessar ou modificar a API durante a vida do módulo.

    Por exemplo, você pode querer chamar uma das funções publicamente expostas de dentro do módulo. Ou pode querer adicionar ou remover métodos dependendo de certas condições, ou atualizar o valor de uma propriedade exposta.

    Seja qual for o caso, parece meio bobo para mim que *não* mantivéssemos uma referência para acessar nossa própria API. Não é?

### Asynchronous Module Definition (AMD)

Outra variação da forma de módulo clássico são os módulos em estilo AMD (populares alguns anos atrás), como os suportados pelo utilitário RequireJS:

```js
define([ "./Student" ],function StudentList(Student){
    var elems = [];

    return {
        renderList() {
            // ..
        }
    };
});
```

Se você olhar de perto `StudentList(..)`, é uma clássica função fábrica de módulo. Dentro da maquinaria de `define(..)` (fornecida pelo RequireJS), a função `StudentList(..)` é executada, recebendo quaisquer outras instâncias de módulo declaradas como dependências. O valor de retorno é um objeto representando a API pública do módulo.

Isso se baseia exatamente nos mesmos princípios (incluindo como a closure funciona!) que exploramos com os módulos clássicos.

### Módulos Universais (UMD)

A última variação que veremos é a UMD, que é menos um formato específico e exato e mais uma coleção de formatos bem parecidos. Ela foi projetada para criar melhor interoperabilidade (sem nenhuma conversão por ferramenta de build) para módulos que podem ser carregados em navegadores, por carregadores em estilo AMD ou no Node. Eu pessoalmente ainda publico muitas das minhas bibliotecas utilitárias usando uma forma de UMD.

Aqui está a estrutura típica de uma UMD:

```js
(function UMD(name,context,definition){
    // carregado por um carregador em estilo AMD?
    if (
        typeof define === "function" &&
        define.amd
    ) {
        define(definition);
    }
    // no Node?
    else if (
        typeof module !== "undefined" &&
        module.exports
    ) {
        module.exports = definition(name,context);
    }
    // presume script independente de navegador
    else {
        context[name] = definition(name,context);
    }
})("StudentList",this,function DEF(name,context){

    var elems = [];

    return {
        renderList() {
            // ..
        }
    };

});
```

Embora possa parecer um pouco incomum, a UMD é, na verdade, apenas uma IIFE.

O que é diferente é que a parte principal da expressão de `function` (no topo) da IIFE contém uma série de instruções `if..else if` para detectar em qual dos três ambientes suportados o módulo está sendo carregado.

O `()` final, que normalmente invoca uma IIFE, está recebendo três argumentos: `"StudentsList"`, `this` e outra expressão de `function`. Se você associar esses argumentos aos seus parâmetros, verá que eles são: `name`, `context` e `definition`, respectivamente. `"StudentList"` (`name`) é o rótulo de nome do módulo, principalmente para o caso de ele ser definido como variável global. `this` (`context`) é geralmente o `window` (ou seja, o objeto global; veja o Capítulo 4), para definir o módulo pelo seu nome.

`definition(..)` é invocada para de fato recuperar a definição do módulo e, você vai notar que, com certeza, isso é apenas uma forma de módulo clássico!

Não há dúvida de que, no momento em que escrevo, os ESM (Módulos ES) estão se tornando populares e difundidos rapidamente. Mas, com milhões e milhões de módulos escritos ao longo dos últimos 20 anos, todos usando alguma variação pré-ESM de módulos clássicos, eles ainda são muito importantes de se conseguir ler e entender quando você se deparar com eles.

[^fowlerIOC]: *Inversion of Control*, Martin Fowler, https://martinfowler.com/bliki/InversionOfControl.html, 26 de junho de 2005.
