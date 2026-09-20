# You Don't Know JS Yet: Get Started - 2ª Edição
# Capítulo 2: Um Panorama do JS

A melhor maneira de aprender JS é começar a escrever JS.

Para isso, você precisa saber como a linguagem funciona, e é nisso que vamos focar aqui. Mesmo que você já tenha programado em outras linguagens, tire seu tempo para ficar confortável com JS e certifique-se de praticar cada parte.

Este capítulo não é uma referência exaustiva sobre cada pedacinho da sintaxe da linguagem JS. Também não pretende ser uma cartilha completa de "introdução ao JS".

Em vez disso, vamos apenas percorrer algumas das principais áreas temáticas da linguagem. Nosso objetivo é ganhar uma *sensação* melhor sobre ela, para que possamos avançar escrevendo nossos próprios programas com mais confiança. Vamos revisitar muitos desses tópicos com sucessivos níveis de detalhe ao longo do restante deste livro e do restante da série.

Por favor, não espere que este capítulo seja uma leitura rápida. Ele é longo e há bastante detalhe para mastigar. Vá com calma.

| DICA: |
| :--- |
| Se você ainda está se familiarizando com JS, sugiro que reserve bastante tempo extra para trabalhar este capítulo. Pegue cada seção e pondere e explore o tópico por um tempo. Olhe programas JS existentes e compare o que você vê neles com o código e as explicações (e opiniões!) apresentadas aqui. Você vai extrair muito mais do restante do livro e da série com uma base sólida sobre a *natureza* do JS. |

## Cada Arquivo é um Programa

Quase todo site (aplicação web) que você usa é composto por muitos arquivos JS diferentes (tipicamente com a extensão .js). É tentador pensar na coisa toda (a aplicação) como um único programa. Mas o JS enxerga isso de outro jeito.

Em JS, cada arquivo independente é seu próprio programa separado.

A razão pela qual isso importa gira principalmente em torno do tratamento de erros. Como o JS trata arquivos como programas, um arquivo pode falhar (durante o parse/compilação ou a execução) e isso não vai necessariamente impedir que o próximo arquivo seja processado. Obviamente, se sua aplicação depende de cinco arquivos .js e um deles falha, a aplicação como um todo provavelmente vai operar só parcialmente, na melhor das hipóteses. É importante garantir que cada arquivo funcione corretamente e que, na medida do possível, eles lidem com falhas em outros arquivos da forma mais elegante possível.

Pode te surpreender considerar arquivos .js separados como programas JS separados. Da perspectiva do seu uso de uma aplicação, ela certamente parece um grande programa só. Isso porque a execução da aplicação permite que esses *programas* individuais cooperem e ajam como um único programa.

| NOTA: |
| :--- |
| Muitos projetos usam ferramentas de processo de build que acabam combinando arquivos separados do projeto em um único arquivo a ser entregue para uma página web. Quando isso acontece, o JS trata esse único arquivo combinado como o programa inteiro. |

A única maneira de múltiplos arquivos .js independentes agirem como um único programa é compartilhando seu estado (e o acesso à sua funcionalidade pública) por meio do "escopo global". Eles se misturam nesse namespace de escopo global, então, em tempo de execução, agem como um todo.

Desde o ES6, o JS também suporta um formato de módulo, além do típico formato de programa JS independente. Módulos também são baseados em arquivos. Se um arquivo é carregado por um mecanismo de carregamento de módulos, como uma instrução `import` ou uma tag `<script type=module>`, todo o seu código é tratado como um único módulo.

Embora você normalmente não pense em um módulo — uma coleção de estado e métodos publicamente expostos para operar sobre esse estado — como um programa independente, o JS, de fato, ainda trata cada módulo separadamente. De forma parecida com como o "escopo global" permite que arquivos independentes se misturem em tempo de execução, importar um módulo dentro de outro permite a interoperação entre eles em tempo de execução.

Independentemente de qual padrão de organização de código (e mecanismo de carregamento) seja usado para um arquivo (independente ou módulo), você ainda deve pensar em cada arquivo como seu próprio (mini) programa, que pode então cooperar com outros (mini) programas para realizar as funções da sua aplicação como um todo.

## Valores

A unidade mais fundamental de informação em um programa é um valor. Valores são dados. São a forma como o programa mantém estado. Valores vêm em duas formas em JS: **primitivo** e **objeto**.

Valores são embutidos em programas usando *literais*:

```js
greeting("My name is Kyle.");
```

Neste programa, o valor `"My name is Kyle."` é um literal primitivo de string; strings são coleções ordenadas de caracteres, geralmente usadas para representar palavras e frases.

Usei o caractere aspas duplas `"` para *delimitar* (cercar, separar, definir) o valor da string. Mas eu poderia ter usado o caractere aspas simples `'` também. A escolha de qual caractere de aspas usar é inteiramente estilística. O importante, em nome da legibilidade e da manutenibilidade do código, é escolher um e usá-lo de forma consistente ao longo do programa.

Outra opção para delimitar um literal de string é usar o caractere crase `` ` ``. Porém, essa escolha não é meramente estilística; há também uma diferença de comportamento. Considere:

```js
console.log("My name is ${ firstName }.");
// My name is ${ firstName }.

console.log('My name is ${ firstName }.');
// My name is ${ firstName }.

console.log(`My name is ${ firstName }.`);
// My name is Kyle.
```

Presumindo que este programa já tenha definido uma variável `firstName` com o valor de string `"Kyle"`, a string delimitada por `` ` `` então resolve a expressão de variável (indicada com `${ .. }`) para seu valor atual. Isso se chama **interpolação**.

A string delimitada por crase `` ` `` pode ser usada sem incluir expressões interpoladas, mas isso anula todo o propósito dessa sintaxe alternativa de literal de string:

```js
console.log(
    `Am I confusing you by omitting interpolation?`
);
// Am I confusing you by omitting interpolation?
```

A melhor abordagem é usar `"` ou `'` (de novo, escolha um e mantenha!) para strings *a menos que você precise* de interpolação; reserve `` ` `` apenas para strings que vão incluir expressões interpoladas.

Além de strings, programas JS frequentemente contêm outros valores literais primitivos, como booleanos e números:

```js
while (false) {
    console.log(3.141592);
}
```

`while` representa um tipo de laço, uma forma de repetir operações *enquanto* (*while*) sua condição for verdadeira.

Neste caso, o laço nunca vai rodar (e nada será impresso), porque usamos o valor booleano `false` como condição do laço. `true` teria resultado em um laço que continua para sempre, então cuidado!

O número `3.141592` é, como você talvez saiba, uma aproximação do PI matemático até o sexto dígito. Em vez de embutir tal valor, porém, você normalmente usaria o valor predefinido `Math.PI` para esse propósito. Outra variação sobre números é o tipo primitivo `bigint` (big-integer, ou inteiro grande), usado para armazenar números arbitrariamente grandes.

Números são usados com mais frequência em programas para contar passos, como iterações de laço, e para acessar informação em posições numéricas (ou seja, um índice de array). Vamos cobrir arrays/objetos daqui a pouco, mas, como exemplo, se houvesse um array chamado `names`, poderíamos acessar o elemento na sua segunda posição assim:

```js
console.log(`My name is ${ names[1] }.`);
// My name is Kyle.
```

Usamos `1` para o elemento na segunda posição, em vez de `2`, porque, como na maioria das linguagens de programação, os índices de array em JS são baseados em 0 (`0` é a primeira posição).

Além de strings, números e booleanos, outros dois valores *primitivos* em programas JS são `null` e `undefined`. Embora existam diferenças entre eles (algumas históricas e outras contemporâneas), em grande parte ambos os valores servem ao propósito de indicar *vazio* (ou ausência) de um valor.

Muitos desenvolvedores preferem tratar os dois de forma consistente dessa maneira, ou seja, presumindo que os valores sejam indistinguíveis. Com cuidado, isso muitas vezes é possível. Porém, o mais seguro e melhor é usar apenas `undefined` como o único valor vazio, mesmo que `null` pareça atraente por ser mais curto de digitar!

```js
while (value != undefined) {
    console.log("Still got something!");
}
```

O último valor primitivo a se conhecer é o symbol, um valor de propósito especial que se comporta como um valor oculto e não adivinhável. Symbols são quase exclusivamente usados como chaves especiais em objetos:

```js
hitchhikersGuide[ Symbol("meaning of life") ];
// 42
```

Você não vai encontrar o uso direto de symbols com muita frequência em programas JS típicos. Eles são usados sobretudo em código de baixo nível, como em bibliotecas e frameworks.

### Arrays e Objetos

Além dos primitivos, o outro tipo de valor em JS é o valor de objeto.

Como mencionado antes, arrays são um tipo especial de objeto, composto por uma lista de dados ordenada e indexada numericamente:

```js
var names = [ "Frank", "Kyle", "Peter", "Susan" ];

names.length;
// 4

names[0];
// Frank

names[1];
// Kyle
```

Arrays em JS podem conter qualquer tipo de valor, primitivo ou objeto (incluindo outros arrays). Como veremos perto do fim do Capítulo 3, até funções são valores que podem ser guardados em arrays ou objetos.

| NOTA: |
| :--- |
| Funções, assim como arrays, são um tipo especial (ou seja, um subtipo) de objeto. Vamos cobrir funções com mais detalhes em breve. |

Objetos são mais gerais: uma coleção não ordenada, com chaves, de quaisquer valores diversos. Em outras palavras, você acessa o elemento por um nome de localização em string (também chamado de "chave" ou "propriedade") em vez de por sua posição numérica (como nos arrays). Por exemplo:

```js
var me = {
    first: "Kyle",
    last: "Simpson",
    age: 39,
    specialties: [ "JS", "Table Tennis" ]
};

console.log(`My name is ${ me.first }.`);
```

Aqui, `me` representa um objeto, e `first` representa o nome de uma localização de informação nesse objeto (coleção de valores). Outra opção de sintaxe que acessa informação em um objeto pela sua propriedade/chave usa os colchetes `[ ]`, como em `me["first"]`.

### Determinação do Tipo de Valor

Para distinguir valores, o operador `typeof` informa seu tipo embutido, se for primitivo, ou `"object"` caso contrário:

```js
typeof 42;                  // "number"
typeof "abc";               // "string"
typeof true;                // "boolean"
typeof undefined;           // "undefined"
typeof null;                // "object" -- ops, bug!
typeof { "a": 1 };          // "object"
typeof [1,2,3];             // "object"
typeof function hello(){};  // "function"
```

| AVISO: |
| :--- |
| `typeof null`, infelizmente, retorna `"object"` em vez do esperado `"null"`. Além disso, `typeof` retorna o específico `"function"` para funções, mas não o esperado `"array"` para arrays. |

Converter de um tipo de valor para outro, como de string para número, é chamado em JS de "coerção". Vamos cobrir isso com mais detalhes ainda neste capítulo.

Valores primitivos e valores de objeto se comportam de forma diferente quando são atribuídos ou passados adiante. Vamos cobrir esses detalhes no Apêndice A, "Valores vs. Referências".

## Declarando e Usando Variáveis

Para deixar explícito algo que pode não ter ficado óbvio na seção anterior: em programas JS, valores podem tanto aparecer como valores literais (como muitos dos exemplos anteriores ilustram) quanto ser guardados em variáveis; pense em variáveis apenas como contêineres para valores.

Variáveis precisam ser declaradas (criadas) para serem usadas. Existem várias formas de sintaxe que declaram variáveis (também chamadas de "identificadores"), e cada forma tem comportamentos implícitos diferentes.

Por exemplo, considere a instrução `var`:

```js
var myName = "Kyle";
var age;
```

A palavra-chave `var` declara uma variável a ser usada naquela parte do programa e, opcionalmente, permite uma atribuição inicial de um valor.

Outra palavra-chave parecida é `let`:

```js
let myName = "Kyle";
let age;
```

A palavra-chave `let` tem algumas diferenças em relação a `var`, sendo a mais óbvia que `let` permite um acesso mais limitado à variável do que `var`. Isso se chama "escopo de bloco" (*block scoping*), em oposição ao escopo regular ou de função.

Considere:

```js
var adult = true;

if (adult) {
    var myName = "Kyle";
    let age = 39;
    console.log("Shhh, this is a secret!");
}

console.log(myName);
// Kyle

console.log(age);
// Error!
```

A tentativa de acessar `age` fora da instrução `if` resulta em erro, porque `age` teve escopo de bloco limitado ao `if`, enquanto `myName` não.

Escopo de bloco é muito útil para limitar o alcance das declarações de variáveis nos nossos programas, o que ajuda a evitar sobreposição acidental de seus nomes.

Mas `var` continua útil no sentido de comunicar "esta variável será vista por um escopo mais amplo (o da função inteira)". Ambas as formas de declaração podem ser apropriadas em qualquer parte de um programa, dependendo das circunstâncias.

| NOTA: |
| :--- |
| É muito comum sugerir que `var` deva ser evitado em favor de `let` (ou `const`!), geralmente por causa de uma confusão percebida sobre como o comportamento de escopo do `var` funciona desde o início do JS. Acredito que esse seja um conselho excessivamente restritivo e, no fim das contas, pouco útil. Ele presume que você é incapaz de aprender e usar um recurso corretamente em combinação com outros recursos. Acredito que você *pode* e *deve* aprender todos os recursos disponíveis e usá-los onde for apropriado! |

Uma terceira forma de declaração é `const`. É como `let`, mas tem a limitação adicional de que deve receber um valor no momento em que é declarada e não pode ter um valor diferente reatribuído depois.

Considere:

```js
const myBirthday = true;
let age = 39;

if (myBirthday) {
    age = age + 1;    // OK!
    myBirthday = false;  // Error!
}
```

A constante `myBirthday` não pode ser reatribuída.

Variáveis declaradas com `const` não são "imutáveis"; elas apenas não podem ser reatribuídas. Não é aconselhável usar `const` com valores de objeto, porque esses valores ainda podem ser alterados, mesmo que a variável não possa ser reatribuída. Isso leva a uma confusão em potencial mais adiante, então acho sábio evitar situações como:

```js
const actors = [
    "Morgan Freeman", "Jennifer Aniston"
];

actors[2] = "Tom Cruise";   // OK :(
actors = [];                // Error!
```

O melhor uso semântico de um `const` é quando você tem um valor primitivo simples ao qual quer dar um nome útil, como usar `myBirthday` em vez de `true`. Isso torna os programas mais fáceis de ler.

| DICA: |
| :--- |
| Se você se limitar a usar `const` apenas com valores primitivos, evita qualquer confusão entre reatribuição (não permitida) e mutação (permitida)! Essa é a forma mais segura e melhor de usar `const`. |

Além de `var` / `let` / `const`, existem outras formas sintáticas que declaram identificadores (variáveis) em diversos escopos. Por exemplo:

```js
function hello(myName) {
    console.log(`Hello, ${ myName }.`);
}

hello("Kyle");
// Hello, Kyle.
```

O identificador `hello` é criado no escopo externo, e também é automaticamente associado de modo a referenciar a função. Mas o parâmetro nomeado `myName` é criado apenas dentro da função e, portanto, só é acessível dentro do escopo dessa função. `hello` e `myName` geralmente se comportam como se fossem declarados com `var`.

Outra sintaxe que declara uma variável é a cláusula `catch`:

```js
try {
    someError();
}
catch (err) {
    console.log(err);
}
```

O `err` é uma variável com escopo de bloco que existe apenas dentro da cláusula `catch`, como se tivesse sido declarada com `let`.

## Funções

A palavra "função" tem diversos significados em programação. Por exemplo, no mundo da Programação Funcional, "função" tem uma definição matemática precisa e implica um conjunto estrito de regras a serem seguidas.

Em JS, devemos considerar que "função" assume o significado mais amplo de outro termo relacionado: "procedimento". Um procedimento é uma coleção de instruções que pode ser invocada uma ou mais vezes, pode receber algumas entradas e pode devolver uma ou mais saídas.

Desde os primeiros dias do JS, a definição de função se parecia com isto:

```js
function awesomeFunction(coolThings) {
    // ..
    return amazingStuff;
}
```

Isso é chamado de declaração de função, porque aparece como uma instrução por si só, e não como uma expressão dentro de outra instrução. A associação entre o identificador `awesomeFunction` e o valor da função acontece durante a fase de compilação do código, antes de esse código ser executado.

Em contraste com a instrução de declaração de função, uma expressão de função pode ser definida e atribuída assim:

```js
// let awesomeFunction = ..
// const awesomeFunction = ..
var awesomeFunction = function(coolThings) {
    // ..
    return amazingStuff;
};
```

Esta função é uma expressão que é atribuída à variável `awesomeFunction`. Diferente da forma de declaração de função, uma expressão de função não é associada ao seu identificador até aquela instrução ser executada em tempo de execução.

É extremamente importante notar que, em JS, funções são valores que podem ser atribuídos (como mostrado neste trecho) e passados adiante. De fato, funções em JS são um tipo especial do tipo de valor objeto. Nem todas as linguagens tratam funções como valores, mas isso é essencial para que uma linguagem suporte o padrão de programação funcional, como o JS faz.

Funções JS podem receber parâmetros de entrada:

```js
function greeting(myName) {
    console.log(`Hello, ${ myName }!`);
}

greeting("Kyle");   // Hello, Kyle!
```

Neste trecho, `myName` é chamado de parâmetro, e atua como uma variável local dentro da função. Funções podem ser definidas para receber qualquer quantidade de parâmetros, de nenhum em diante, conforme você achar adequado. Cada parâmetro recebe o valor do argumento que você passa naquela posição (`"Kyle"`, aqui) da chamada.

Funções também podem retornar valores usando a palavra-chave `return`:

```js
function greeting(myName) {
    return `Hello, ${ myName }!`;
}

var msg = greeting("Kyle");

console.log(msg);   // Hello, Kyle!
```

Você só pode fazer `return` de um único valor, mas, se tiver mais valores a retornar, pode embrulhá-los em um único objeto/array.

Como funções são valores, elas podem ser atribuídas como propriedades em objetos:

```js
var whatToSay = {
    greeting() {
        console.log("Hello!");
    },
    question() {
        console.log("What's your name?");
    },
    answer() {
        console.log("My name is Kyle.");
    }
};

whatToSay.greeting();
// Hello!
```

Neste trecho, referências a três funções (`greeting()`, `question()` e `answer()`) estão incluídas no objeto guardado em `whatToSay`. Cada função pode ser chamada acessando a propriedade para recuperar o valor de referência da função. Compare esse estilo direto de definir funções em um objeto com a sintaxe mais sofisticada de `class`, discutida mais adiante neste capítulo.

Existem muitas formas variadas que as `function`s assumem em JS. Vamos nos aprofundar nessas variações no Apêndice A, "Tantas Formas de Função".

## Comparações

Tomar decisões em programas exige comparar valores para determinar sua identidade e sua relação uns com os outros. O JS tem vários mecanismos que permitem comparar valores, então vamos olhá-los mais de perto.

### Igual... mais ou menos

A comparação mais comum em programas JS faz a pergunta: "Este valor X é *o mesmo que* aquele valor Y?" Mas o que exatamente "o mesmo que" significa de verdade para o JS?

Por razões ergonômicas e históricas, o significado é mais complicado do que a óbvia correspondência de *identidade exata*. Às vezes uma comparação de igualdade pretende uma correspondência *exata*, mas outras vezes a comparação desejada é um pouco mais ampla, permitindo correspondência *bastante similar* ou *intercambiável*. Em outras palavras, precisamos estar cientes das diferenças sutis entre uma comparação de **igualdade** (*equality*) e uma comparação de **equivalência** (*equivalence*).

Se você já passou algum tempo trabalhando com JS e lendo sobre ele, certamente já viu o chamado operador "igual triplo" `===`, também descrito como o operador de "igualdade estrita". Isso parece bem direto, não é? Certamente "estrito" significa estrito, no sentido de restrito e *exato*.

Não *exata*mente.

Sim, a maior parte dos valores que participam de uma comparação de igualdade `===` vai se encaixar naquela intuição de *exatamente o mesmo*. Considere alguns exemplos:

```js
3 === 3.0;              // true
"yes" === "yes";        // true
null === null;          // true
false === false;        // true

42 === "42";            // false
"hello" === "Hello";    // false
true === 1;             // false
0 === null;             // false
"" === null;            // false
null === undefined;     // false
```

| NOTA: |
| :--- |
| Outra forma comum de descrever a comparação de igualdade do `===` é: "verifica tanto o valor quanto o tipo". Em vários dos exemplos que vimos até aqui, como `42 === "42"`, o *tipo* de ambos os valores (number, string etc.) parece mesmo ser o fator distintivo. Mas há mais do que isso. **Todas** as comparações de valores em JS consideram o tipo dos valores comparados, não *apenas* o operador `===`. Especificamente, `===` proíbe qualquer tipo de conversão de tipo (também chamada de "coerção") em sua comparação, enquanto outras comparações do JS *permitem* coerção. |

Mas o operador `===` tem, sim, suas sutilezas, um fato que muitos desenvolvedores JS passam por cima, para seu próprio prejuízo. O operador `===` foi projetado para *mentir* em dois casos de valores especiais: `NaN` e `-0`. Considere:

```js
NaN === NaN;            // false
0 === -0;               // true
```

No caso de `NaN`, o operador `===` *mente* e diz que uma ocorrência de `NaN` não é igual a outro `NaN`. No caso de `-0` (sim, este é um valor real e distinto, que você pode usar intencionalmente nos seus programas!), o operador `===` *mente* e diz que ele é igual ao valor `0` normal.

Como essas *mentiras* em tais comparações podem ser incômodas, é melhor evitar usar `===` para elas. Para comparações com `NaN`, use o utilitário `Number.isNaN(..)`, que não *mente*. Para comparação com `-0`, use o utilitário `Object.is(..)`, que também não *mente*. `Object.is(..)` também pode ser usado para checagens de `NaN` sem *mentiras*, se você preferir. Humoristicamente, você poderia pensar em `Object.is(..)` como o "igual quádruplo" `====`, a comparação realmente-realmente-estrita!

Existem razões históricas e técnicas mais profundas para essas *mentiras*, mas isso não muda o fato de que `===` não é, de fato, uma comparação de *igualdade estritamente exata*, no sentido mais *estrito*.

A história fica ainda mais complicada quando consideramos comparações de valores de objeto (não primitivos). Considere:

```js
[ 1, 2, 3 ] === [ 1, 2, 3 ];    // false
{ a: 42 } === { a: 42 }         // false
(x => x * 2) === (x => x * 2)   // false
```

O que está acontecendo aqui?

Pode parecer razoável presumir que uma checagem de igualdade considere a *natureza* ou o *conteúdo* do valor; afinal, `42 === 42` considera o valor `42` em si e o compara. Mas, quando se trata de objetos, uma comparação que leva o conteúdo em conta é geralmente chamada de "igualdade estrutural".

O JS não define `===` como *igualdade estrutural* para valores de objeto. Em vez disso, `===` usa *igualdade de identidade* para valores de objeto.

Em JS, todos os valores de objeto são mantidos por referência (veja "Valores vs. Referências" no Apêndice A), são atribuídos e passados por cópia de referência **e**, para a nossa discussão atual, são comparados por igualdade de referência (identidade). Considere:

```js
var x = [ 1, 2, 3 ];

// a atribuição é por cópia de referência, então
// y referencia o *mesmo* array que x,
// e não outra cópia dele.
var y = x;

y === x;              // true
y === [ 1, 2, 3 ];    // false
x === [ 1, 2, 3 ];    // false
```

Neste trecho, `y === x` é verdadeiro porque ambas as variáveis guardam uma referência ao mesmo array inicial. Mas as comparações `=== [1,2,3]` falham, porque `y` e `x`, respectivamente, estão sendo comparados a novos arrays *diferentes* `[1,2,3]`. A estrutura e o conteúdo do array não importam nessa comparação, apenas a **identidade da referência**.

O JS não fornece um mecanismo para comparação de igualdade estrutural de valores de objeto, apenas comparação de identidade de referência. Para fazer comparação de igualdade estrutural, você mesmo precisará implementar as verificações.

Mas cuidado, é mais complicado do que você vai supor. Por exemplo, como você determinaria se duas referências de função são "estruturalmente equivalentes"? Mesmo transformá-las em string para comparar o texto do código-fonte não levaria em conta coisas como closure. O JS não fornece comparação de igualdade estrutural porque é quase intratável lidar com todos os casos de borda!

### Comparações Coercitivas

Coerção significa um valor de um tipo sendo convertido para sua respectiva representação em outro tipo (na medida do possível). Como discutiremos no Capítulo 4, coerção é um pilar central da linguagem JS, e não algum recurso opcional que se possa razoavelmente evitar.

Mas, onde a coerção encontra os operadores de comparação (como o de igualdade), confusão e frustração infelizmente aparecem com mais frequência do que não.

Poucos recursos do JS atraem mais fúria na comunidade JS em geral do que o operador `==`, geralmente chamado de operador de "igualdade frouxa". A maior parte de tudo que se escreve e se discute publicamente sobre JS condena esse operador como mal projetado e perigoso/cheio de bugs quando usado em programas JS. Até o próprio criador da linguagem, Brendan Eich, já lamentou que ele tenha sido projetado como um grande erro.

Pelo que consigo perceber, a maior parte dessa frustração vem de uma lista bem curta de casos de borda confusos, mas um problema mais profundo é o equívoco extremamente difundido de que ele faz suas comparações sem considerar os tipos dos valores comparados.

O operador `==` realiza uma comparação de igualdade de forma semelhante ao `===`. De fato, ambos os operadores consideram o tipo dos valores comparados. E, se a comparação é entre valores do mesmo tipo, tanto `==` quanto `===` **fazem exatamente a mesma coisa, sem nenhuma diferença.**

Se os tipos dos valores comparados são diferentes, o `==` difere do `===` por permitir coerção antes da comparação. Em outras palavras, ambos querem comparar valores de tipos iguais, mas `==` permite conversões de tipo *primeiro* e, uma vez que os tipos tenham sido convertidos para serem iguais dos dois lados, então `==` faz a mesma coisa que `===`. Em vez de "igualdade frouxa", o operador `==` deveria ser descrito como "igualdade coercitiva".

Considere:

```js
42 == "42";             // true
1 == true;              // true
```

Em ambas as comparações, os tipos dos valores são diferentes, então o `==` faz com que os valores que não são números (`"42"` e `true`) sejam convertidos para números (`42` e `1`, respectivamente) antes de as comparações serem feitas.

Só de estar ciente dessa natureza do `==` — de que ele prefere comparações numéricas primitivas — você já evita a maior parte dos casos de borda problemáticos, como manter distância de pegadinhas como `"" == 0` ou `0 == false`.

Você pode estar pensando: "Ah, bom, então eu vou simplesmente sempre evitar qualquer comparação de igualdade coercitiva (usando `===` no lugar) para fugir desses casos de borda"! Ih, desculpe, isso não é tão provável quanto você gostaria.

Há uma boa chance de você usar operadores de comparação relacional como `<`, `>` (e até `<=` e `>=`).

Assim como `==`, esses operadores vão se comportar como se fossem "estritos" se os tipos sendo comparados relacionalmente já coincidirem, mas vão permitir coerção antes (geralmente, para números) se os tipos diferirem.

Considere:

```js
var arr = [ "1", "10", "100", "1000" ];
for (let i = 0; i < arr.length && arr[i] < 500; i++) {
    // vai rodar 3 vezes
}
```

A comparação `i < arr.length` está "a salvo" de coerção, porque `i` e `arr.length` são sempre números. Já `arr[i] < 500` invoca coerção, porque os valores de `arr[i]` são todos strings. Essas comparações, portanto, se tornam `1 < 500`, `10 < 500`, `100 < 500` e `1000 < 500`. Como essa quarta é falsa, o laço para depois da terceira iteração.

Esses operadores relacionais tipicamente usam comparações numéricas, exceto no caso em que **ambos** os valores comparados já são strings; nesse caso, eles usam comparação alfabética (tipo dicionário) das strings:

```js
var x = "10";
var y = "9";

x < y;      // true, cuidado!
```

Não há como fazer esses operadores relacionais evitarem coerção, a não ser nunca usar tipos incompatíveis nas comparações. Isso talvez seja admirável como meta, mas ainda é bem provável que você acabe esbarrando em um caso em que os tipos *possam* diferir.

A abordagem mais sábia não é evitar comparações coercitivas, mas abraçá-las e aprender seus detalhes.

Comparações coercitivas aparecem em outros lugares no JS, como em condicionais (`if` etc.), o que vamos revisitar no Apêndice A, "Comparação Condicional Coercitiva".

## Como Nos Organizamos em JS

Dois grandes padrões para organizar código (dados e comportamento) são amplamente usados em todo o ecossistema JS: classes e módulos. Esses padrões não são mutuamente exclusivos; muitos programas podem usar — e usam — ambos. Outros programas ficam com apenas um padrão, ou até com nenhum!

Em alguns aspectos, esses padrões são muito diferentes. Mas, curiosamente, de outras formas, são apenas lados diferentes da mesma moeda. Ser proficiente em JS exige entender ambos os padrões e onde eles são apropriados (e onde não são!).

### Classes

Os termos "orientado a objetos", "orientado a classes" e "classes" são todos bastante carregados de detalhes e nuances; não têm definição universal.

Vamos usar aqui uma definição comum e um tanto tradicional, aquela mais provavelmente familiar a quem tem bagagem em linguagens "orientadas a objetos" como C++ e Java.

Uma classe em um programa é a definição de um "tipo" de estrutura de dados customizada que inclui tanto dados quanto comportamentos que operam sobre esses dados. Classes definem como essa estrutura de dados funciona, mas classes não são, elas próprias, valores concretos. Para obter um valor concreto que você possa usar no programa, uma classe precisa ser *instanciada* (com a palavra-chave `new`) uma ou mais vezes.

Considere:

```js
class Page {
    constructor(text) {
        this.text = text;
    }

    print() {
        console.log(this.text);
    }
}

class Notebook {
    constructor() {
        this.pages = [];
    }

    addPage(text) {
        var page = new Page(text);
        this.pages.push(page);
    }

    print() {
        for (let page of this.pages) {
            page.print();
        }
    }
}

var mathNotes = new Notebook();
mathNotes.addPage("Arithmetic: + - * / ...");
mathNotes.addPage("Trigonometry: sin cos tan ...");

mathNotes.print();
// ..
```

Na classe `Page`, os dados são uma string de texto armazenada em uma propriedade membro `this.text`. O comportamento é `print()`, um método que despeja o texto no console.

Para a classe `Notebook`, os dados são um array de instâncias de `Page`. O comportamento é `addPage(..)`, um método que instancia novas páginas `Page` e as adiciona à lista, assim como `print()` (que imprime todas as páginas do caderno).

A instrução `mathNotes = new Notebook()` cria uma instância da classe `Notebook`, e `page = new Page(text)` é onde instâncias da classe `Page` são criadas.

Comportamento (métodos) só pode ser chamado em instâncias (não nas próprias classes), como em `mathNotes.addPage(..)` e `page.print()`.

O mecanismo `class` permite empacotar dados (`text` e `pages`) organizados junto com seus comportamentos (por exemplo, `addPage(..)` e `print()`). O mesmo programa poderia ter sido construído sem nenhuma definição de `class`, mas provavelmente seria muito menos organizado, mais difícil de ler e raciocinar sobre e mais suscetível a bugs e a uma manutenção ruim.

#### Herança de Classes

Outro aspecto inerente ao design "orientado a classes" tradicional, embora um pouco menos usado em JS, é a "herança" (e o "polimorfismo"). Considere:

```js
class Publication {
    constructor(title,author,pubDate) {
        this.title = title;
        this.author = author;
        this.pubDate = pubDate;
    }

    print() {
        console.log(`
            Title: ${ this.title }
            By: ${ this.author }
            ${ this.pubDate }
        `);
    }
}
```

Esta classe `Publication` define um conjunto de comportamentos comuns de que qualquer publicação possa precisar.

Agora vamos considerar tipos mais específicos de publicação, como `Book` e `BlogPost`:

```js
class Book extends Publication {
    constructor(bookDetails) {
        super(
            bookDetails.title,
            bookDetails.author,
            bookDetails.pubDate
        );
        this.publisher = bookDetails.publisher;
        this.ISBN = bookDetails.ISBN;
    }

    print() {
        super.print();
        console.log(`
            Publisher: ${ this.publisher }
            ISBN: ${ this.ISBN }
        `);
    }
}

class BlogPost extends Publication {
    constructor(title,author,pubDate,URL) {
        super(title,author,pubDate);
        this.URL = URL;
    }

    print() {
        super.print();
        console.log(this.URL);
    }
}
```

Tanto `Book` quanto `BlogPost` usam a cláusula `extends` para *estender* a definição geral de `Publication`, incluindo comportamento adicional. A chamada `super(..)` em cada construtor delega ao construtor da classe pai `Publication` o trabalho de inicialização, e então cada uma faz coisas mais específicas de acordo com seu respectivo tipo de publicação (ou seja, "subclasse" ou "classe filha").

Agora considere usar essas classes filhas:

```js
var YDKJS = new Book({
    title: "You Don't Know JS",
    author: "Kyle Simpson",
    pubDate: "June 2014",
    publisher: "O'Reilly",
    ISBN: "123456-789"
});

YDKJS.print();
// Title: You Don't Know JS
// By: Kyle Simpson
// June 2014
// Publisher: O'Reilly
// ISBN: 123456-789

var forAgainstLet = new BlogPost(
    "For and against let",
    "Kyle Simpson",
    "October 27, 2014",
    "https://davidwalsh.name/for-and-against-let"
);

forAgainstLet.print();
// Title: For and against let
// By: Kyle Simpson
// October 27, 2014
// https://davidwalsh.name/for-and-against-let
```

Note que ambas as instâncias das classes filhas têm um método `print()`, que foi uma sobrescrita do método `print()` *herdado* da classe pai `Publication`. Cada um desses métodos `print()` sobrescritos nas classes filhas chama `super.print()` para invocar a versão herdada do método `print()`.

O fato de que tanto o método herdado quanto o sobrescrito podem ter o mesmo nome e coexistir é chamado de *polimorfismo*.

Herança é uma ferramenta poderosa para organizar dados/comportamento em unidades lógicas separadas (classes), permitindo ao mesmo tempo que a classe filha coopere com a pai, acessando/usando seu comportamento e seus dados.

### Módulos

O padrão de módulo tem essencialmente o mesmo objetivo do padrão de classe: agrupar dados e comportamento em unidades lógicas. Também como as classes, módulos podem "incluir" ou "acessar" os dados e comportamentos de outros módulos, em nome da cooperação.

Mas módulos têm algumas diferenças importantes em relação a classes. Mais notavelmente, a sintaxe é completamente diferente.

#### Módulos Clássicos

O ES6 adicionou uma forma de sintaxe de módulo à sintaxe nativa do JS, que veremos daqui a pouco. Mas, desde os primeiros dias do JS, módulos eram um padrão importante e comum, aproveitado em incontáveis programas JS, mesmo sem uma sintaxe dedicada.

As marcas registradas de um *módulo clássico* são uma função externa (que roda pelo menos uma vez), que retorna uma "instância" do módulo com uma ou mais funções expostas, capazes de operar sobre os dados internos (ocultos) da instância do módulo.

Como um módulo dessa forma é *apenas uma função*, e chamá-la produz uma "instância" do módulo, outra descrição para essas funções é "fábricas de módulos" (*module factories*).

Considere a forma de módulo clássico das classes `Publication`, `Book` e `BlogPost` vistas antes:

```js
function Publication(title,author,pubDate) {
    var publicAPI = {
        print() {
            console.log(`
                Title: ${ title }
                By: ${ author }
                ${ pubDate }
            `);
        }
    };

    return publicAPI;
}

function Book(bookDetails) {
    var pub = Publication(
        bookDetails.title,
        bookDetails.author,
        bookDetails.publishedOn
    );

    var publicAPI = {
        print() {
            pub.print();
            console.log(`
                Publisher: ${ bookDetails.publisher }
                ISBN: ${ bookDetails.ISBN }
            `);
        }
    };

    return publicAPI;
}

function BlogPost(title,author,pubDate,URL) {
    var pub = Publication(title,author,pubDate);

    var publicAPI = {
        print() {
            pub.print();
            console.log(URL);
        }
    };

    return publicAPI;
}
```

Comparando essas formas com as formas de `class`, há mais semelhanças do que diferenças.

A forma `class` armazena métodos e dados em uma instância de objeto, que precisa ser acessada com o prefixo `this.`. Com módulos, os métodos e dados são acessados como variáveis identificadoras no escopo, sem nenhum prefixo `this.`.

Com `class`, a "API" de uma instância é implícita na definição da classe — além disso, todos os dados e métodos são públicos. Com a função fábrica de módulo, você cria e retorna explicitamente um objeto com quaisquer métodos publicamente expostos, e quaisquer dados ou outros métodos não referenciados permanecem privados dentro da função fábrica.

Existem outras variações dessa forma de função fábrica que são bastante comuns em todo o JS, mesmo em 2020; você pode se deparar com essas formas em diferentes programas JS: AMD (Asynchronous Module Definition), UMD (Universal Module Definition) e CommonJS (os módulos clássicos no estilo Node.js). As variações são pequenas (não totalmente compatíveis). Porém, todas essas formas se apoiam nos mesmos princípios básicos.

Considere também o uso (ou seja, a "instanciação") dessas funções fábrica de módulos:

```js
var YDKJS = Book({
    title: "You Don't Know JS",
    author: "Kyle Simpson",
    publishedOn: "June 2014",
    publisher: "O'Reilly",
    ISBN: "123456-789"
});

YDKJS.print();
// Title: You Don't Know JS
// By: Kyle Simpson
// June 2014
// Publisher: O'Reilly
// ISBN: 123456-789

var forAgainstLet = BlogPost(
    "For and against let",
    "Kyle Simpson",
    "October 27, 2014",
    "https://davidwalsh.name/for-and-against-let"
);

forAgainstLet.print();
// Title: For and against let
// By: Kyle Simpson
// October 27, 2014
// https://davidwalsh.name/for-and-against-let
```

A única diferença observável aqui é a ausência do uso de `new`, chamando as fábricas de módulo como funções normais.

#### Módulos ES

Os módulos ES (ESM), introduzidos na linguagem JS no ES6, têm a intenção de servir a praticamente o mesmo espírito e propósito dos *módulos clássicos* que acabamos de descrever, levando em conta especialmente variações e casos de uso importantes de AMD, UMD e CommonJS.

A abordagem de implementação, porém, difere significativamente.

Primeiro, não há função envolvente para *definir* um módulo. O contexto envolvente é um arquivo. ESMs são sempre baseados em arquivo; um arquivo, um módulo.

Segundo, você não interage explicitamente com a "API" de um módulo, e sim usa a palavra-chave `export` para adicionar uma variável ou método à definição da sua API pública. Se algo é definido em um módulo mas não é exportado com `export`, então permanece oculto (assim como nos *módulos clássicos*).

Terceiro, e talvez a diferença mais perceptível em relação aos padrões discutidos anteriormente, você não "instancia" um módulo ES; você simplesmente o importa com `import` para usar sua única instância. ESMs são, na prática, "singletons", no sentido de que só uma instância é criada, no primeiro `import` do seu programa, e todos os outros `import`s apenas recebem uma referência para essa mesma instância única. Se seu módulo precisa suportar múltiplas instanciações, você precisa fornecer uma função fábrica no *estilo de módulo clássico* dentro da sua definição ESM para esse fim.

No nosso exemplo corrente, estamos presumindo múltiplas instanciações, então os trechos a seguir vão misturar ESM e *módulos clássicos*.

Considere o arquivo `publication.js`:

```js
function printDetails(title,author,pubDate) {
    console.log(`
        Title: ${ title }
        By: ${ author }
        ${ pubDate }
    `);
}

export function create(title,author,pubDate) {
    var publicAPI = {
        print() {
            printDetails(title,author,pubDate);
        }
    };

    return publicAPI;
}
```

Para importar e usar esse módulo, a partir de outro módulo ES como `blogpost.js`:

```js
import { create as createPub } from "publication.js";

function printDetails(pub,URL) {
    pub.print();
    console.log(URL);
}

export function create(title,author,pubDate,URL) {
    var pub = createPub(title,author,pubDate);

    var publicAPI = {
        print() {
            printDetails(pub,URL);
        }
    };

    return publicAPI;
}
```

E, por fim, para usar esse módulo, importamos dentro de outro módulo ES como `main.js`:

```js
import { create as newBlogPost } from "blogpost.js";

var forAgainstLet = newBlogPost(
    "For and against let",
    "Kyle Simpson",
    "October 27, 2014",
    "https://davidwalsh.name/for-and-against-let"
);

forAgainstLet.print();
// Title: For and against let
// By: Kyle Simpson
// October 27, 2014
// https://davidwalsh.name/for-and-against-let
```

| NOTA: |
| :--- |
| A cláusula `as newBlogPost` na instrução `import` é opcional; se omitida, seria importada uma função de nível superior chamada apenas `create(..)`. Neste caso, estou renomeando em nome da legibilidade; seu nome de fábrica mais genérico, `create(..)`, se torna semanticamente mais descritivo do seu propósito como `newBlogPost(..)`. |

Como mostrado, módulos ES podem usar *módulos clássicos* internamente se precisarem suportar múltiplas instanciações. Alternativamente, poderíamos ter exposto uma `class` a partir do nosso módulo, em vez de uma função fábrica `create(..)`, com praticamente o mesmo resultado. Porém, já que a essa altura você já está usando ESM, eu recomendaria ficar com os *módulos clássicos* em vez de `class`.

Se seu módulo só precisa de uma única instância, você pode pular as camadas extras de complexidade: exporte seus métodos públicos diretamente com `export`.

## A Toca do Coelho Fica Mais Funda

Como prometido no início deste capítulo, acabamos de passar os olhos por uma ampla área da superfície das principais partes da linguagem JS. Sua cabeça pode ainda estar girando, mas isso é totalmente natural depois de tal mangueira de informação!

Mesmo com este panorama "breve" do JS, cobrimos ou sinalizamos uma tonelada de detalhes que você deve considerar com cuidado e garantir que está confortável com eles. Estou falando sério quando sugiro: releia este capítulo, talvez várias vezes.

No próximo capítulo, vamos nos aprofundar muito mais em alguns aspectos importantes de como o JS funciona em sua essência. Mas, antes de seguir essa toca do coelho mais fundo, certifique-se de ter dedicado tempo adequado para digerir plenamente o que acabamos de cobrir aqui.
