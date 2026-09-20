# You Don't Know JS Yet: Get Started - 2ª Edição
# Apêndice A: Explorando Mais a Fundo

Neste apêndice, vamos explorar alguns tópicos do texto principal dos capítulos com um pouco mais de detalhe. Pense neste conteúdo como uma prévia opcional de alguns dos detalhes mais sutis cobertos ao longo do restante da série de livros.

## Valores vs. Referências

No Capítulo 2, apresentamos os dois principais tipos de valores: primitivos e objetos. Mas ainda não discutimos uma diferença fundamental entre eles: como esses valores são atribuídos e passados adiante.

Em muitas linguagens, o desenvolvedor pode escolher entre atribuir/passar um valor como o próprio valor ou como uma referência ao valor. Em JS, porém, essa decisão é inteiramente determinada pelo tipo do valor. Isso surpreende muitos desenvolvedores vindos de outras linguagens quando começam a usar JS.

Se você atribui/passa o valor em si, o valor é copiado. Por exemplo:

```js
var myName = "Kyle";

var yourName = myName;
```

Aqui, a variável `yourName` tem uma cópia separada da string `"Kyle"` em relação ao valor armazenado em `myName`. Isso porque o valor é um primitivo, e valores primitivos são sempre atribuídos/passados como **cópias de valor**.

Veja como você pode provar que há dois valores separados envolvidos:

```js
var myName = "Kyle";

var yourName = myName;

myName = "Frank";

console.log(myName);
// Frank

console.log(yourName);
// Kyle
```

Percebeu como `yourName` não foi afetada pela reatribuição de `myName` para `"Frank"`? Isso porque cada variável guarda sua própria cópia do valor.

Em contraste, referências são a ideia de que duas ou mais variáveis estão apontando para o mesmo valor, de modo que modificar esse valor compartilhado se reflete no acesso por qualquer uma dessas referências. Em JS, apenas valores de objeto (arrays, objetos, funções etc.) são tratados como referências.

Considere:

```js
var myAddress = {
    street: "123 JS Blvd",
    city: "Austin",
    state: "TX"
};

var yourAddress = myAddress;

// Preciso me mudar para uma casa nova!
myAddress.street = "456 TS Ave";

console.log(yourAddress.street);
// 456 TS Ave
```

Como o valor atribuído a `myAddress` é um objeto, ele é guardado/atribuído por referência e, portanto, a atribuição à variável `yourAddress` é uma cópia da referência, e não do valor do objeto em si. É por isso que o valor atualizado atribuído a `myAddress.street` se reflete quando acessamos `yourAddress.street`. `myAddress` e `yourAddress` têm cópias da referência ao único objeto compartilhado, então uma atualização em uma é uma atualização em ambas.

De novo: o JS escolhe o comportamento de cópia de valor vs. cópia de referência com base no tipo do valor. Primitivos são guardados por valor, objetos são guardados por referência. Não há como sobrescrever isso em JS, em nenhuma das direções.

## Tantas Formas de Função

Lembre-se deste trecho da seção "Funções", no Capítulo 2:

```js
var awesomeFunction = function(coolThings) {
    // ..
    return amazingStuff;
};
```

A expressão de função aqui é chamada de *expressão de função anônima*, já que não tem identificador de nome entre a palavra-chave `function` e a lista de parâmetros `(..)`. Esse ponto confunde muitos desenvolvedores JS porque, a partir do ES6, o JS realiza uma "inferência de nome" em uma função anônima:

```js
awesomeFunction.name;
// "awesomeFunction"
```

A propriedade `name` de uma função vai revelar ou seu nome dado diretamente (no caso de uma declaração) ou seu nome inferido, no caso de uma expressão de função anônima. Esse valor geralmente é usado por ferramentas de desenvolvimento ao inspecionar um valor de função ou ao reportar um stack trace de erro.

Então até uma expressão de função anônima *pode* ganhar um nome. Porém, a inferência de nome só acontece em casos limitados, como quando a expressão de função é atribuída (com `=`). Se você passar uma expressão de função como argumento para uma chamada de função, por exemplo, nenhuma inferência de nome ocorre; a propriedade `name` será uma string vazia, e o console do desenvolvedor geralmente vai reportar "(anonymous function)".

Mesmo que um nome seja inferido, **ela ainda é uma função anônima.** Por quê? Porque o nome inferido é um valor de string de metadados, não um identificador disponível para se referir à função. Uma função anônima não tem um identificador para usar e se referir a si mesma de dentro de si mesma — para recursão, desvinculação de eventos etc.

Compare a forma de expressão de função anônima com:

```js
// let awesomeFunction = ..
// const awesomeFunction = ..
var awesomeFunction = function someName(coolThings) {
    // ..
    return amazingStuff;
};

awesomeFunction.name;
// "someName"
```

Esta expressão de função é uma *expressão de função nomeada*, já que o identificador `someName` é diretamente associado à expressão de função em tempo de compilação; a associação com o identificador `awesomeFunction` ainda só acontece em tempo de execução, no momento daquela instrução. Esses dois identificadores não precisam coincidir; às vezes faz sentido que sejam diferentes, outras vezes é melhor que sejam iguais.

Note também que o nome explícito da função, o identificador `someName`, tem precedência ao atribuir um *nome* para a propriedade `name`.

Expressões de função devem ser nomeadas ou anônimas? As opiniões variam bastante sobre isso. A maioria dos desenvolvedores tende a não se preocupar em usar funções anônimas. Elas são mais curtas e, inquestionavelmente, mais comuns na ampla esfera do código JS por aí.

Na minha opinião, se uma função existe no seu programa, ela tem um propósito; caso contrário, tire-a de lá! E, se tem um propósito, tem um nome natural que descreve esse propósito.

Se uma função tem um nome, você, autor do código, deveria incluir esse nome no código, para que o leitor não precise inferi-lo lendo e executando mentalmente o código-fonte dessa função. Mesmo um corpo trivial de função como `x * 2` precisa ser lido para inferir um nome como "double" ou "multBy2"; esse breve trabalho mental extra é desnecessário quando você poderia apenas tirar um segundo para nomear a função como "double" ou "multBy2" *uma vez*, poupando o leitor desse trabalho mental repetido toda vez que ela for lida no futuro.

Existem, lamentavelmente em alguns aspectos, muitas outras formas de definição de função em JS no início de 2020 (talvez mais no futuro!).

Aqui estão mais algumas formas de declaração:

```js
// declaração de função geradora
function *two() { .. }

// declaração de função assíncrona
async function three() { .. }

// declaração de função geradora assíncrona
async function *four() { .. }

// declaração de exportação de função nomeada (módulos ES6)
export function five() { .. }
```

E aqui estão mais algumas das (muitas!) formas de expressão de função:

```js
// IIFE
(function(){ .. })();
(function namedIIFE(){ .. })();

// IIFE assíncrona
(async function(){ .. })();
(async function namedAIIFE(){ .. })();

// expressões de arrow function
var f;
f = () => 42;
f = x => x * 2;
f = (x) => x * 2;
f = (x,y) => x * y;
f = x => ({ x: x * 2 });
f = x => { return x * 2; };
f = async x => {
    var y = await doSomethingAsync(x);
    return y * 2;
};
someOperation( x => x * 2 );
// ..
```

Tenha em mente que expressões de arrow function são **sintaticamente anônimas**, ou seja, a sintaxe não oferece uma forma de fornecer um identificador de nome direto para a função. A expressão de função pode ganhar um nome inferido, mas apenas se estiver em uma das formas de atribuição, e não na forma (mais comum!) de ser passada como argumento de uma chamada de função (como na última linha do trecho).

Como não acho que funções anônimas sejam uma boa ideia para usar com frequência nos seus programas, não sou fã de usar a forma de arrow function `=>`. Esse tipo de função tem, sim, um propósito específico (ou seja, tratar a palavra-chave `this` lexicamente), mas isso não significa que devamos usá-la para toda função que escrevemos. Use a ferramenta mais apropriada para cada trabalho.

Funções também podem ser especificadas em definições de classe e em definições de literal de objeto. Nessas formas, elas são tipicamente chamadas de "métodos", embora em JS esse termo não tenha muita diferença observável em relação a "função":

```js
class SomethingKindaGreat {
    // métodos de classe
    coolMethod() { .. }   // sem vírgulas!
    boringMethod() { .. }
}

var EntirelyDifferent = {
    // métodos de objeto
    coolMethod() { .. },   // com vírgulas!
    boringMethod() { .. },

    // propriedade com expressão de função (anônima)
    oldSchool: function() { .. }
};
```

Ufa! São muitas formas diferentes de definir funções.

Não há atalho simples aqui; você só precisa construir familiaridade com todas as formas de função, para reconhecê-las em código existente e usá-las apropriadamente no código que escreve. Estude-as de perto e pratique!

## Comparação Condicional Coercitiva

Sim, o nome dessa seção é bem comprido. Mas do que estamos falando? Estamos falando de expressões condicionais que precisam realizar comparações orientadas a coerção para tomar suas decisões.

Instruções `if` e o ternário `? :`, bem como as cláusulas de teste em laços `while` e `for`, todos realizam uma comparação implícita de valor. Mas de que tipo? É "estrita" ou "coercitiva"? Ambas, na verdade.

Considere:

```js
var x = 1;

if (x) {
    // vai rodar!
}

while (x) {
    // vai rodar, uma vez!
    x = false;
}
```

Você pode pensar nessas expressões condicionais `(x)` assim:

```js
var x = 1;

if (x == true) {
    // vai rodar!
}

while (x == true) {
    // vai rodar, uma vez!
    x = false;
}
```

Neste caso específico — o valor de `x` sendo `1` — esse modelo mental funciona, mas ele não é preciso de forma mais ampla. Considere:

```js
var x = "hello";

if (x) {
    // vai rodar!
}

if (x == true) {
    // não vai rodar :(
}
```

Ops. Então, o que a instrução `if` está fazendo de fato? Este é o modelo mental mais preciso:

```js
var x = "hello";

if (Boolean(x) == true) {
    // vai rodar
}

// o que é o mesmo que:

if (Boolean(x) === true) {
    // vai rodar
}
```

Como a função `Boolean(..)` sempre retorna um valor do tipo booleano, o `==` vs. `===` neste trecho é irrelevante; ambos vão fazer a mesma coisa. Mas a parte importante é perceber que, antes da comparação, ocorre uma coerção, de qualquer tipo que `x` tenha no momento para booleano.

Simplesmente não dá para escapar das coerções nas comparações do JS. Arregace as mangas e aprenda-as.

## "Classes" Prototipais

No Capítulo 3, apresentamos protótipos e mostramos como podemos vincular objetos por meio de uma cadeia de protótipos.

Outra forma de montar esses vínculos de protótipo serviu como a (honestamente, feia) predecessora da elegância do sistema `class` do ES6 (veja o Capítulo 2, "Classes"), e é chamada de classes prototipais.

| DICA: |
| :--- |
| Embora esse estilo de código seja bastante incomum em JS hoje em dia, é ainda desconcertantemente comum ser perguntado sobre ele em entrevistas de emprego! |

Vamos primeiro relembrar o estilo de código com `Object.create(..)`:

```js
var Classroom = {
    welcome() {
        console.log("Welcome, students!");
    }
};

var mathClass = Object.create(Classroom);

mathClass.welcome();
// Welcome, students!
```

Aqui, um objeto `mathClass` é vinculado, por meio do seu protótipo, a um objeto `Classroom`. Por meio desse vínculo, a chamada de função `mathClass.welcome()` é delegada ao método definido em `Classroom`.

O padrão de classe prototipal teria rotulado esse comportamento de delegação como "herança" e, alternativamente, o teria definido (com o mesmo comportamento) assim:

```js
function Classroom() {
    // ..
}

Classroom.prototype.welcome = function hello() {
    console.log("Welcome, students!");
};

var mathClass = new Classroom();

mathClass.welcome();
// Welcome, students!
```

Todas as funções, por padrão, referenciam um objeto vazio em uma propriedade chamada `prototype`. Apesar da nomenclatura confusa, este **não** é o *protótipo* da função (aquele ao qual a função está vinculada por protótipo), e sim o objeto protótipo ao qual *vincular* outros objetos quando eles forem criados chamando a função com `new`.

Adicionamos uma propriedade `welcome` nesse objeto vazio (chamado `Classroom.prototype`), apontando para a função `hello()`.

Então `new Classroom()` cria um novo objeto (atribuído a `mathClass`) e o vincula por protótipo ao objeto `Classroom.prototype` existente.

Embora `mathClass` não tenha uma propriedade/função `welcome()`, ele delega com sucesso para a função `Classroom.prototype.welcome()`.

Esse padrão de "classe prototipal" é hoje fortemente desencorajado, em favor do uso do mecanismo `class` do ES6:

```js
class Classroom {
    constructor() {
        // ..
    }

    welcome() {
        console.log("Welcome, students!");
    }
}

var mathClass = new Classroom();

mathClass.welcome();
// Welcome, students!
```

Por baixo dos panos, o mesmo vínculo de protótipo é montado, mas essa sintaxe `class` se encaixa no padrão de design orientado a classes de forma muito mais limpa do que as "classes prototipais".
