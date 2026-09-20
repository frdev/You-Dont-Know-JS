# You Don't Know JS Yet: Scope & Closures - 2ª Edição
# Capítulo 3: A Cadeia de Escopos

Os Capítulos 1 e 2 estabeleceram uma definição concreta de *escopo léxico* (e de suas partes) e ilustraram metáforas úteis para sua base conceitual. Antes de prosseguir com este capítulo, encontre outra pessoa para explicar (por escrito ou em voz alta), com suas próprias palavras, o que é escopo léxico e por que é útil entendê-lo.

Parece um passo que você poderia pular, mas eu descobri que realmente ajuda tirar um tempo para reformular essas ideias como explicações para outras pessoas. Isso ajuda nosso cérebro a digerir o que estamos aprendendo!

Agora é hora de entrar nos detalhes, então espere que as coisas fiquem muito mais minuciosas daqui em diante. Persista, porém, porque estas discussões realmente deixam claro o quanto todos nós *ainda não sabemos* sobre escopo. Vá com calma no texto e em todos os trechos de código fornecidos.

Para refrescar o contexto do nosso exemplo corrente, vamos relembrar a ilustração com cores das bolhas de escopo aninhadas, do Capítulo 2, Figura 2:

<figure>
    <img src="../../../scope-closures/images/fig2.png" width="500" alt="Bolhas de Escopo Coloridas" align="center">
    <figcaption><em>Fig. 2 (Cap. 2): Bolhas de Escopo Coloridas</em></figcaption>
    <br><br>
</figure>

As conexões entre escopos que estão aninhados dentro de outros escopos são chamadas de cadeia de escopos, que determina o caminho pelo qual variáveis podem ser acessadas. A cadeia é direcionada, ou seja, a busca se move apenas para cima/para fora.

## A "Busca" É (Quase Sempre) Conceitual

Na Figura 2, note a cor da referência à variável `students` no laço `for`. Como exatamente determinamos que ela é uma bolinha VERMELHA(1)?

No Capítulo 2, descrevemos o acesso a uma variável em tempo de execução como uma "busca", em que a *Engine* precisa começar perguntando ao *Gerente de Escopo* do escopo atual se ele conhece um identificador/variável, e prosseguir para cima/para fora pela cadeia de escopos aninhados (rumo ao escopo global) até encontrá-lo, se é que encontra. A busca para assim que a primeira declaração de nome correspondente é encontrada em um balde de escopo.

O processo de busca determinou, assim, que `students` é uma bolinha VERMELHA(1), porque ainda não havíamos encontrado um nome de variável correspondente enquanto percorríamos a cadeia de escopos, até chegarmos ao escopo global VERMELHO(1) final.

De forma semelhante, `studentID` na instrução `if` é determinada como uma bolinha AZUL(2).

Essa sugestão de um processo de busca em tempo de execução funciona bem para o entendimento conceitual, mas não é assim que as coisas costumam funcionar na prática.

A cor do balde de uma bolinha (ou seja, a metainformação de qual escopo uma variável se origina) é *geralmente determinada* durante o processamento inicial de compilação. Como o escopo léxico está praticamente finalizado nesse ponto, a cor de uma bolinha não vai mudar em função de nada que possa acontecer depois, em tempo de execução.

Como a cor da bolinha é conhecida desde a compilação, e é imutável, essa informação provavelmente seria armazenada junto (ou pelo menos acessível a partir) da entrada de cada variável na AST; essa informação é então usada explicitamente pelas instruções executáveis que constituem o tempo de execução do programa.

Em outras palavras, a *Engine* (do Capítulo 2) não precisa buscar através de um monte de escopos para descobrir de qual balde de escopo uma variável vem. Essa informação já é conhecida! Evitar a necessidade de uma busca em tempo de execução é um benefício-chave de otimização do escopo léxico. O tempo de execução opera com mais desempenho sem gastar tempo com todas essas buscas.

Mas eu disse "...geralmente determinada..." há pouco, com respeito a descobrir a cor de uma bolinha durante a compilação. Então, em que caso ela *não* seria conhecida durante a compilação?

Considere uma referência a uma variável que não é declarada em nenhum escopo lexicamente disponível no arquivo atual — veja *Get Started*, Capítulo 1, que afirma que cada arquivo é seu próprio programa separado, da perspectiva da compilação JS. Se nenhuma declaração é encontrada, isso não é *necessariamente* um erro. Outro arquivo (programa) no ambiente de execução pode, de fato, declarar aquela variável no escopo global compartilhado.

Então a determinação final sobre se a variável foi apropriadamente declarada em algum balde acessível pode precisar ser adiada para o tempo de execução.

Qualquer referência a uma variável que esteja inicialmente *não declarada* é deixada como uma bolinha sem cor durante a compilação daquele arquivo; essa cor não pode ser determinada até que outro(s) arquivo(s) relevante(s) tenham sido compilados e o tempo de execução da aplicação comece. Essa busca adiada vai eventualmente resolver a cor para qualquer que seja o escopo em que a variável for encontrada (provavelmente o escopo global).

Porém, essa busca só seria necessária, no máximo, uma vez por variável, já que nada mais durante o tempo de execução poderia depois mudar a cor daquela bolinha.

A seção "Falhas de Busca", no Capítulo 2, cobre o que acontece se uma bolinha continua, afinal, sem cor no momento em que sua referência é executada em tempo de execução.

## Sombreamento

"Sombreamento" (*shadowing*) pode soar misterioso e um pouco suspeito. Mas não se preocupe, é completamente legítimo!

Nosso exemplo corrente nestes capítulos usa nomes de variáveis diferentes através das fronteiras de escopo. Como todos têm nomes únicos, de certa forma não faria diferença se todos fossem guardados em um único balde (como o VERMELHO(1)).

Onde ter baldes de escopo léxico distintos começa a importar mais é quando você tem duas ou mais variáveis, cada uma em escopos diferentes, com os mesmos nomes léxicos. Um único escopo não pode ter duas ou mais variáveis com o mesmo nome; múltiplas referências assim seriam assumidas como apenas uma variável.

Então, se você precisa manter duas ou mais variáveis de mesmo nome, precisa usar escopos separados (geralmente aninhados). E, nesse caso, é muito relevante como os diferentes baldes de escopo estão dispostos.

Considere:

```js
var studentName = "Suzy";

function printStudent(studentName) {
    studentName = studentName.toUpperCase();
    console.log(studentName);
}

printStudent("Frank");
// FRANK

printStudent(studentName);
// SUZY

console.log(studentName);
// Suzy
```

| DICA: |
| :--- |
| Antes de seguir adiante, tire um tempo para analisar este código usando as várias técnicas/metáforas que cobrimos no livro. Em particular, certifique-se de identificar as cores das bolinhas/bolhas neste trecho. É um bom exercício! |

A variável `studentName` na linha 1 (a instrução `var studentName = ..`) cria uma bolinha VERMELHA(1). Uma variável de mesmo nome é declarada como uma bolinha AZUL(2) na linha 3, o parâmetro na definição da função `printStudent(..)`.

De que cor será a bolinha `studentName` na instrução de atribuição `studentName = studentName.toUpperCase()` e na instrução `console.log(studentName)`? Todas as três referências a `studentName` serão AZUIS(2).

Com a noção conceitual da "busca", afirmamos que ela começa no escopo atual e vai se movendo para fora/para cima, parando assim que uma variável correspondente é encontrada. O `studentName` AZUL(2) é encontrado imediatamente. O `studentName` VERMELHO(1) nem sequer é considerado.

Este é um aspecto central do comportamento do escopo léxico, chamado *sombreamento*. A variável `studentName` AZUL(2) (parâmetro) sombreia o `studentName` VERMELHO(1). Então, o parâmetro está sombreando a variável global (sombreada). Repita essa frase para si mesmo algumas vezes, para garantir que a terminologia esteja clara!

É por isso que a reatribuição de `studentName` afeta apenas a variável interna (o parâmetro): o `studentName` AZUL(2), e não o `studentName` global VERMELHO(1).

Quando você escolhe sombrear uma variável de um escopo externo, um impacto direto é que, daquele escopo para dentro/para baixo (através de quaisquer escopos aninhados), passa a ser impossível que qualquer bolinha seja colorida como a variável sombreada — (VERMELHA(1), neste caso). Em outras palavras, qualquer referência ao identificador `studentName` vai corresponder àquela variável de parâmetro, nunca à variável global `studentName`. É lexicamente impossível referenciar o `studentName` global em qualquer lugar dentro da função `printStudent(..)` (ou a partir de quaisquer escopos aninhados).

### O Truque de Dessombrear o Global

Atenção: usar a técnica que vou descrever não é uma prática muito boa, já que ela é limitada em utilidade, confusa para quem lê seu código e propensa a convidar bugs para o seu programa. Estou cobrindo isso apenas porque você pode se deparar com esse comportamento em programas existentes, e entender o que está acontecendo é crítico para não tropeçar.

É *possível*, sim, acessar uma variável global a partir de um escopo em que essa variável foi sombreada, mas não através de uma referência a identificador léxico típica.

No escopo global (VERMELHO(1)), declarações `var` e declarações `function` também se expõem como propriedades (de mesmo nome que o identificador) no *objeto global* — essencialmente uma representação em objeto do escopo global. Se você já escreveu JS para um ambiente de navegador, provavelmente reconhece o objeto global como `window`. Isso não é *totalmente* preciso, mas é suficiente para nossa discussão. No próximo capítulo, vamos explorar mais o tópico escopo/objeto global.

Considere este programa, especificamente executado como um arquivo .js independente em um ambiente de navegador:

```js
var studentName = "Suzy";

function printStudent(studentName) {
    console.log(studentName);
    console.log(window.studentName);
}

printStudent("Frank");
// "Frank"
// "Suzy"
```

Notou a referência `window.studentName`? Essa expressão está acessando a variável global `studentName` como uma propriedade de `window` (que, por ora, estamos fingindo ser sinônimo do objeto global). Essa é a única forma de acessar uma variável sombreada de dentro de um escopo onde a variável sombreadora está presente.

O `window.studentName` é um espelho da variável global `studentName`, não uma cópia instantânea separada. Mudanças em um ainda são vistas a partir do outro, em ambas as direções. Você pode pensar em `window.studentName` como um getter/setter que acessa a variável `studentName` de verdade. Aliás, você pode até *adicionar* uma variável ao escopo global criando/definindo uma propriedade no objeto global.

| AVISO: |
| :--- |
| Lembre-se: só porque você *pode* não significa que *deva*. Não sombreie uma variável global que você precise acessar e, por outro lado, evite usar esse truque para acessar uma variável global que você sombreou. E definitivamente não confunda quem lê seu código criando variáveis globais como propriedades de `window` em vez de com declarações formais! |

Esse pequeno "truque" só funciona para acessar uma variável do escopo global (não uma variável sombreada de um escopo aninhado) e, mesmo assim, apenas uma que tenha sido declarada com `var` ou `function`.

Outras formas de declaração no escopo global não criam propriedades espelhadas no objeto global:

```js
var one = 1;
let notOne = 2;
const notTwo = 3;
class notThree {}

console.log(window.one);       // 1
console.log(window.notOne);    // undefined
console.log(window.notTwo);    // undefined
console.log(window.notThree);  // undefined
```

Variáveis (não importa como sejam declaradas!) que existem em qualquer escopo que não seja o global são completamente inacessíveis a partir de um escopo em que foram sombreadas:

```js
var special = 42;

function lookingFor(special) {
    // O identificador `special` (parâmetro) neste
    // escopo é sombreado dentro de keepLooking(), e
    // é, portanto, inacessível a partir daquele escopo.

    function keepLooking() {
        var special = 3.141592;
        console.log(special);
        console.log(window.special);
    }

    keepLooking();
}

lookingFor(112358132134);
// 3.141592
// 42
```

O `special` global VERMELHO(1) é sombreado pelo `special` AZUL(2) (parâmetro), e o `special` AZUL(2) é ele próprio sombreado pelo `special` VERDE(3) dentro de `keepLooking()`. Ainda podemos acessar o `special` VERMELHO(1) usando a referência indireta `window.special`. Mas não há como `keepLooking()` acessar o `special` AZUL(2), que guarda o número `112358132134`.

### Copiar Não é Acessar

Já me fizeram a seguinte pergunta do tipo "Mas e quanto a...?" dezenas de vezes. Considere:

```js
var special = 42;

function lookingFor(special) {
    var another = {
        special: special
    };

    function keepLooking() {
        var special = 3.141592;
        console.log(special);
        console.log(another.special);  // Opa, esperto!
        console.log(window.special);
    }

    keepLooking();
}

lookingFor(112358132134);
// 3.141592
// 112358132134
// 42
```

Ah! Então essa técnica do objeto `another` desmente minha afirmação de que o parâmetro `special` é "completamente inacessível" de dentro de `keepLooking()`? Não, a afirmação continua correta.

`special: special` está copiando o valor da variável de parâmetro `special` para outro contêiner (uma propriedade de mesmo nome). Claro, se você coloca um valor em outro contêiner, o sombreamento não se aplica mais (a menos que `another` também estivesse sombreado!). Mas isso não significa que estamos acessando o parâmetro `special`; significa que estamos acessando a cópia do valor que ele tinha naquele momento, por meio de *outro* contêiner (propriedade de objeto). Não podemos reatribuir o parâmetro `special` AZUL(2) a um valor diferente de dentro de `keepLooking()`.

Outro "Mas...!?" que você pode estar prestes a levantar: e se eu tivesse usado objetos ou arrays como valores, em vez dos números (`112358132134` etc.)? Ter referências a objetos, em vez de cópias de valores primitivos, "consertaria" a inacessibilidade?

Não. Mutar o conteúdo do valor do objeto por meio de uma cópia de referência **não** é a mesma coisa que acessar lexicamente a própria variável. Ainda não podemos reatribuir o parâmetro `special` AZUL(2).

### Sombreamento Ilegal

Nem todas as combinações de sombreamento de declaração são permitidas. `let` pode sombrear `var`, mas `var` não pode sombrear `let`:

```js
function something() {
    var special = "JavaScript";

    {
        let special = 42;   // sombreamento totalmente ok

        // ..
    }
}

function another() {
    // ..

    {
        let special = "JavaScript";

        {
            var special = "JavaScript";
            // ^^^ Syntax Error

            // ..
        }
    }
}
```

Note que, na função `another()`, a declaração interna `var special` está tentando declarar um `special` de escopo de função inteira, o que em si é ok (como mostra a função `something()`).

A descrição do erro de sintaxe nesse caso indica que `special` já foi definido, mas essa mensagem de erro é um pouco enganosa — de novo, nenhum erro desse tipo acontece em `something()`, já que o sombreamento geralmente é permitido sem problemas.

A verdadeira razão pela qual ele é lançado como um `SyntaxError` é que o `var` está basicamente tentando "cruzar a fronteira" (ou saltar por cima) da declaração `let` de mesmo nome, o que não é permitido.

Essa proibição de cruzar fronteiras efetivamente para em cada fronteira de função, então esta variante não levanta exceção:

```js
function another() {
    // ..

    {
        let special = "JavaScript";

        ajax("https://some.url",function callback(){
            // sombreamento totalmente ok
            var special = "JavaScript";

            // ..
        });
    }
}
```

Resumo: `let` (em um escopo interno) sempre pode sombrear um `var` de escopo externo. `var` (em um escopo interno) só pode sombrear um `let` de escopo externo se houver uma fronteira de função entre eles.

## Escopo do Nome da Função

Como você já viu, uma declaração de `function` se parece com isto:

```js
function askQuestion() {
    // ..
}
```

E, como discutido nos Capítulos 1 e 2, essa declaração de `function` vai criar um identificador no escopo envolvente (neste caso, o escopo global) chamado `askQuestion`.

E quanto a este programa?

```js
var askQuestion = function(){
    // ..
};
```

O mesmo vale para a variável `askQuestion` sendo criada. Mas, como se trata de uma expressão de `function` — uma definição de função usada como valor, em vez de uma declaração independente —, a própria função não sofre "hoisting" (veja o Capítulo 5).

Uma grande diferença entre declarações de `function` e expressões de `function` é o que acontece com o identificador de nome da função. Considere uma expressão de `function` nomeada:

```js
var askQuestion = function ofTheTeacher(){
    // ..
};
```

Sabemos que `askQuestion` acaba no escopo externo. Mas e o identificador `ofTheTeacher`? Para declarações formais de `function`, o identificador de nome acaba no escopo externo/envolvente, então pode parecer razoável supor que seja o caso aqui. Mas `ofTheTeacher` é declarado como um identificador **dentro da própria função**:

```js
var askQuestion = function ofTheTeacher() {
    console.log(ofTheTeacher);
};

askQuestion();
// function ofTheTeacher()...

console.log(ofTheTeacher);
// ReferenceError: ofTheTeacher is not defined
```

| NOTA: |
| :--- |
| Na verdade, `ofTheTeacher` não está exatamente *no escopo da função*. O Apêndice A, "Escopos Implícitos", vai explicar melhor. |

`ofTheTeacher` não só é declarado dentro da função, em vez de fora, como também é definido como somente leitura:

```js
var askQuestion = function ofTheTeacher() {
    "use strict";
    ofTheTeacher = 42;   // TypeError

    //..
};

askQuestion();
// TypeError
```

Como usamos modo estrito, a falha de atribuição é reportada como um `TypeError`; em modo não estrito, tal atribuição falha silenciosamente, sem exceção.

E quando uma expressão de `function` não tem identificador de nome?

```js
var askQuestion = function(){
   // ..
};
```

Uma expressão de `function` com identificador de nome é chamada de "expressão de função nomeada", mas uma sem identificador de nome é chamada de "expressão de função anônima". Expressões de função anônimas claramente não têm identificador de nome que afete qualquer escopo.

| NOTA: |
| :--- |
| Vamos discutir expressões de `function` nomeadas vs. anônimas com muito mais detalhe, incluindo quais fatores afetam a decisão de usar uma ou outra, no Apêndice A. |

## Arrow Functions

O ES6 adicionou à linguagem uma forma adicional de expressão de `function`, chamada "arrow functions" (funções de seta):

```js
var askQuestion = () => {
    // ..
};
```

A arrow function `=>` não exige a palavra `function` para ser definida. Além disso, os `( .. )` em torno da lista de parâmetros são opcionais em alguns casos simples. Da mesma forma, as `{ .. }` em torno do corpo da função são opcionais em alguns casos. E, quando as `{ .. }` são omitidas, um valor de retorno é enviado sem usar a palavra-chave `return`.

| NOTA: |
| :--- |
| A atratividade das arrow functions `=>` é frequentemente vendida como "sintaxe mais curta", e se alega que isso equivale a um código objetivamente mais legível. Essa alegação é, na melhor das hipóteses, duvidosa e, na minha visão, francamente equivocada. Vamos cavar a "legibilidade" das várias formas de função no Apêndice A. |

Arrow functions são lexicamente anônimas, ou seja, não têm um identificador diretamente relacionado que referencie a função. A atribuição a `askQuestion` cria um nome inferido de "askQuestion", mas isso **não é a mesma coisa que ser não anônima**:

```js
var askQuestion = () => {
    // ..
};

askQuestion.name;   // askQuestion
```

Arrow functions alcançam sua brevidade sintática ao custo de ter que malabarizar mentalmente um monte de variações para diferentes formas/condições. Apenas algumas, por exemplo:

```js
() => 42;

id => id.toUpperCase();

(id,name) => ({ id, name });

(...args) => {
    return args[args.length - 1];
};
```

A verdadeira razão pela qual trago as arrow functions à tona é por causa da alegação comum, porém incorreta, de que elas de alguma forma se comportam de modo diferente em relação ao escopo léxico, em comparação com as funções `function` padrão.

Isso é incorreto.

Além de serem anônimas (e não terem forma declarativa), as arrow functions `=>` têm as mesmas regras de escopo léxico que as funções `function`. Uma arrow function, com ou sem `{ .. }` ao redor do corpo, ainda cria um balde de escopo separado, interno e aninhado. Declarações de variáveis dentro desse balde de escopo aninhado se comportam da mesma forma que em um escopo de `function`.

## Recuando

Quando uma função (declaração ou expressão) é definida, um novo escopo é criado. O posicionamento de escopos aninhados uns dentro dos outros cria uma hierarquia natural de escopos ao longo do programa, chamada cadeia de escopos. A cadeia de escopos controla o acesso a variáveis, orientada direcionalmente para cima e para fora.

Cada novo escopo oferece uma folha em branco, um espaço para guardar seu próprio conjunto de variáveis. Quando um nome de variável se repete em níveis diferentes da cadeia de escopos, ocorre o sombreamento, que impede o acesso à variável externa daquele ponto para dentro.

Enquanto recuamos desses detalhes mais finos, o próximo capítulo desloca o foco para o escopo primário que todo programa JS inclui: o escopo global.
