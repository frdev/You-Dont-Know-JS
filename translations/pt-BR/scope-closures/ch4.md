# You Don't Know JS Yet: Scope & Closures - 2ª Edição
# Capítulo 4: Em Torno do Escopo Global

O Capítulo 3 mencionou o "escopo global" várias vezes, mas você talvez ainda esteja se perguntando por que o escopo mais externo de um programa é tão importante no JS moderno. A imensa maioria do trabalho hoje é feita dentro de funções e módulos, em vez de globalmente.

Basta afirmar "Evite usar o escopo global" e pronto?

O escopo global de um programa JS é um tópico rico, com muito mais utilidade e nuance do que você provavelmente suporia. Este capítulo primeiro explora como o escopo global (ainda) é útil e relevante para escrever programas JS hoje, e depois olha as diferenças de onde e *como acessar* o escopo global em diferentes ambientes JS.

Entender plenamente o escopo global é crítico para o seu domínio no uso do escopo léxico para estruturar seus programas.

## Por que Escopo Global?

Provavelmente não é surpresa para os leitores que a maioria das aplicações é composta por múltiplos (às vezes muitos!) arquivos JS individuais. Então, como exatamente todos esses arquivos separados são costurados em um único contexto de execução pela engine JS?

No que diz respeito a aplicações executadas em navegador, há três formas principais.

Primeira: se você está usando módulos ES diretamente (sem transpilá-los para algum outro formato de bundle de módulos), esses arquivos são carregados individualmente pelo ambiente JS. Cada módulo então importa (`import`) referências a quaisquer outros módulos que precise acessar. Os arquivos de módulo separados cooperam entre si exclusivamente por meio desses imports compartilhados, sem precisar de nenhum escopo externo compartilhado.

Segunda: se você está usando um bundler no seu processo de build, todos os arquivos tipicamente são concatenados antes da entrega ao navegador e à engine JS, que então processa apenas um grande arquivo. Mesmo com todas as peças da aplicação colocadas em um único arquivo, é necessário algum mecanismo para cada peça registrar um *nome* a ser referenciado pelas outras peças, bem como alguma facilidade para que esse acesso ocorra.

Em algumas configurações de build, todo o conteúdo do arquivo é embrulhado em um único escopo envolvente, como uma função invólucro, um módulo universal (UMD — veja o Apêndice A) etc. Cada peça pode se registrar para acesso pelas outras peças por meio de variáveis locais nesse escopo compartilhado. Por exemplo:

```js
(function wrappingOuterScope(){
    var moduleOne = (function one(){
        // ..
    })();

    var moduleTwo = (function two(){
        // ..

        function callModuleOne() {
            moduleOne.someMethod();
        }

        // ..
    })();
})();
```

Como mostrado, as variáveis locais `moduleOne` e `moduleTwo`, dentro do escopo da função `wrappingOuterScope()`, são declaradas para que esses módulos possam acessar um ao outro em sua cooperação.

Embora o escopo de `wrappingOuterScope()` seja uma função, e não o escopo global completo do ambiente, ele age como uma espécie de "escopo de toda a aplicação", um balde onde todos os identificadores de nível superior podem ser armazenados, ainda que não no escopo global de verdade. É uma espécie de substituto do escopo global nesse aspecto.

E, por fim, a terceira forma: seja usando uma ferramenta de bundle para a aplicação, seja quando os arquivos (que não são módulos ES) são simplesmente carregados individualmente no navegador (via tags `<script>` ou outro carregamento dinâmico de recursos JS), se não houver um único escopo envolvente abrangendo todas essas peças, o **escopo global** é a única forma de elas cooperarem entre si:

Um arquivo empacotado desse tipo frequentemente se parece com algo assim:

```js
var moduleOne = (function one(){
    // ..
})();
var moduleTwo = (function two(){
    // ..

    function callModuleOne() {
        moduleOne.someMethod();
    }

    // ..
})();
```

Aqui, como não há um escopo de função envolvente, essas declarações `moduleOne` e `moduleTwo` simplesmente caem no escopo global. Isso é efetivamente o mesmo que se os arquivos não tivessem sido concatenados, mas carregados separadamente:

module1.js:

```js
var moduleOne = (function one(){
    // ..
})();
```

module2.js:

```js
var moduleTwo = (function two(){
    // ..

    function callModuleOne() {
        moduleOne.someMethod();
    }

    // ..
})();
```

Se esses arquivos forem carregados separadamente, como arquivos .js independentes normais em um ambiente de navegador, cada declaração de variável de nível superior vai acabar como uma variável global, já que o escopo global é o único recurso compartilhado entre esses dois arquivos separados — eles são programas independentes, da perspectiva da engine JS.

Além de (potencialmente) dar conta de onde o código de uma aplicação reside em tempo de execução, e de como cada peça consegue acessar as outras para cooperar, o escopo global também é onde:

* O JS expõe seus recursos embutidos:

    - primitivos: `undefined`, `null`, `Infinity`, `NaN`
    - nativos: `Date()`, `Object()`, `String()` etc.
    - funções globais: `eval()`, `parseInt()` etc.
    - namespaces: `Math`, `Atomics`, `JSON`
    - amigos do JS: `Intl`, `WebAssembly`

* O ambiente que hospeda a engine JS expõe seus próprios recursos embutidos:

    - `console` (e seus métodos)
    - o DOM (`window`, `document` etc.)
    - temporizadores (`setTimeout(..)` etc.)
    - APIs da plataforma web: `navigator`, `history`, geolocalização, WebRTC etc.

Esses são apenas alguns dos muitos *globais* com os quais seus programas vão interagir.

| NOTA: |
| :--- |
| O Node também expõe vários elementos "globalmente", mas tecnicamente eles não estão no escopo `global`: `require()`, `__dirname`, `module`, `URL` e assim por diante. |

A maioria dos desenvolvedores concorda que o escopo global não deveria ser apenas um depósito para toda variável da sua aplicação. Isso é uma bagunça de bugs esperando para acontecer. Mas também é inegável que o escopo global é uma *cola* importante para praticamente toda aplicação JS.

## Onde Exatamente Fica Esse Escopo Global?

Pode parecer óbvio que o escopo global fica na porção mais externa de um arquivo; ou seja, fora de qualquer função ou outro bloco. Mas não é tão simples assim.

Diferentes ambientes JS lidam com os escopos dos seus programas, especialmente o escopo global, de formas diferentes. É bastante comum que desenvolvedores JS nutram equívocos sem nem perceber.

### A "Window" do Navegador

No que diz respeito ao tratamento do escopo global, o ambiente mais *puro* em que o JS pode rodar é como um arquivo .js independente, carregado em um ambiente de página web em um navegador. Não quero dizer "puro" no sentido de que nada é adicionado automaticamente — muita coisa pode ser adicionada! —, mas sim em termos de intrusão mínima sobre o código ou de interferência no comportamento esperado do seu escopo global.

Considere este arquivo .js:

```js
var studentName = "Kyle";

function hello() {
    console.log(`Hello, ${ studentName }!`);
}

hello();
// Hello, Kyle!
```

Este código pode ser carregado em um ambiente de página web usando uma tag `<script>` inline, uma tag `<script src=..>` na marcação ou até um elemento DOM `<script>` criado dinamicamente. Nos três casos, os identificadores `studentName` e `hello` são declarados no escopo global.

Isso significa que, se você acessar o objeto global (comumente, `window` no navegador), vai encontrar propriedades com esses mesmos nomes lá:

```js
var studentName = "Kyle";

function hello() {
    console.log(`Hello, ${ window.studentName }!`);
}

window.hello();
// Hello, Kyle!
```

Esse é o comportamento padrão que se esperaria a partir de uma leitura da especificação do JS: o escopo externo *é* o escopo global, e `studentName` é legitimamente criada como variável global.

É isso que quero dizer com *puro*. Mas, infelizmente, isso nem sempre será verdade em todos os ambientes JS que você encontrar, e isso frequentemente surpreende desenvolvedores JS.

#### Globais Sombreando Globais

Lembre-se da discussão sobre sombreamento (e dessombreamento do global) no Capítulo 3, em que uma declaração de variável pode sobrepor e impedir o acesso a uma declaração de mesmo nome em um escopo externo.

Uma consequência incomum da diferença entre uma variável global e uma propriedade global de mesmo nome é que, dentro do próprio escopo global, uma propriedade do objeto global pode ser sombreada por uma variável global:

```js
window.something = 42;

let something = "Kyle";

console.log(something);
// Kyle

console.log(window.something);
// 42
```

A declaração `let` adiciona uma variável global `something`, mas não uma propriedade do objeto global (veja o Capítulo 3). O efeito, então, é que o identificador léxico `something` sombreia a propriedade `something` do objeto global.

É quase certamente uma má ideia criar uma divergência entre o objeto global e o escopo global. Quem ler seu código quase certamente vai tropeçar.

Uma forma simples de evitar essa pegadinha com declarações globais: sempre use `var` para globais. Reserve `let` e `const` para escopos de bloco (veja "Criando Escopo com Blocos", no Capítulo 6).

#### Globais do DOM

Afirmei que um ambiente JS hospedado em navegador tem o comportamento de escopo global mais *puro* que veremos. Porém, não é inteiramente *puro*.

Um comportamento surpreendente no escopo global com o qual você pode se deparar em aplicações JS baseadas em navegador: um elemento do DOM com um atributo `id` cria automaticamente uma variável global que o referencia.

Considere esta marcação:

```text
<ul id="my-todo-list">
   <li id="first">Write a book</li>
   ..
</ul>
```

E o JS dessa página poderia incluir:

```js
first;
// <li id="first">..</li>

window["my-todo-list"];
// <ul id="my-todo-list">..</ul>
```

Se o valor do `id` é um nome léxico válido (como `first`), a variável léxica é criada. Se não, a única forma de acessar esse global é pelo objeto global (`window[..]`).

O registro automático de todos os elementos do DOM que possuem `id` como variáveis globais é um comportamento legado antigo dos navegadores que, ainda assim, precisa permanecer porque muitos sites antigos ainda dependem dele. Meu conselho é nunca usar essas variáveis globais, mesmo que elas sempre sejam criadas silenciosamente.

#### O que Há em um Nome (de Window)?

Outra esquisitice do escopo global no JS de navegador:

```js
var name = 42;

console.log(name, typeof name);
// "42" string
```

`window.name` é um "global" predefinido em um contexto de navegador; é uma propriedade do objeto global, então parece uma variável global normal (mas é tudo, menos "normal").

Usamos `var` para nossa declaração, o que **não** sombreia a propriedade global `name` predefinida. Isso significa que, na prática, a declaração `var` é ignorada, já que já existe uma propriedade do objeto de escopo global com esse nome. Como discutimos antes, se tivéssemos usado `let name`, teríamos sombreado `window.name` com uma variável global `name` separada.

Mas o comportamento verdadeiramente surpreendente é que, mesmo tendo atribuído o número `42` a `name` (e, portanto, a `window.name`), quando recuperamos seu valor, ele é a string `"42"`! Nesse caso, a estranheza ocorre porque `name` é, na verdade, um getter/setter predefinido no objeto `window`, que insiste que seu valor seja uma string. Eita!

Com exceção de alguns casos de borda raros, como IDs de elementos do DOM e `window.name`, o JS rodando como um arquivo independente em uma página de navegador tem um dos comportamentos de escopo global mais *puros* que vamos encontrar.

### Web Workers

Web Workers são uma extensão da plataforma web sobre o comportamento do JS de navegador, que permite que um arquivo JS rode em uma thread completamente separada (do ponto de vista do sistema operacional) daquela que roda o programa JS principal.

Como esses programas Web Worker rodam em uma thread separada, eles são restritos em suas comunicações com a thread da aplicação principal, para evitar/limitar condições de corrida e outras complicações. Código de Web Worker não tem acesso ao DOM, por exemplo. Algumas APIs web, porém, são disponibilizadas ao worker, como `navigator`.

Como um Web Worker é tratado como um programa totalmente separado, ele não compartilha o escopo global com o programa JS principal. Porém, a engine JS do navegador continua rodando o código, então podemos esperar uma *pureza* similar no comportamento do seu escopo global. Como não há acesso ao DOM, o apelido `window` para o escopo global não existe.

Em um Web Worker, a referência ao objeto global é tipicamente feita usando `self`:

```js
var studentName = "Kyle";
let studentID = 42;

function hello() {
    console.log(`Hello, ${ self.studentName }!`);
}

self.hello();
// Hello, Kyle!

self.studentID;
// undefined
```

Assim como em programas JS principais, declarações `var` e `function` criam propriedades espelhadas no objeto global (ou seja, `self`), enquanto outras declarações (`let` etc.) não.

Então, de novo, o comportamento de escopo global que vemos aqui é quase tão *puro* quanto se consegue para programas JS em execução; talvez seja até mais *puro*, já que não há DOM para bagunçar as coisas!

### Console/REPL das Ferramentas de Desenvolvedor

Lembre-se, do Capítulo 1 de *Get Started*, que as Ferramentas de Desenvolvedor não criam um ambiente JS completamente aderente. Elas processam código JS, mas também pendem a favor de uma interação de UX que seja mais amigável aos desenvolvedores (ou seja, developer experience, ou DX).

Em alguns casos, favorecer a DX ao digitar trechos curtos de JS, em vez dos passos estritos normais esperados para processar um programa JS completo, produz diferenças observáveis no comportamento do código entre programas e ferramentas. Por exemplo, certas condições de erro aplicáveis a um programa JS podem ser relaxadas e não exibidas quando o código é digitado em uma ferramenta de desenvolvedor.

No que diz respeito às nossas discussões aqui sobre escopo, essas diferenças observáveis de comportamento podem incluir:

* O comportamento do escopo global

* Hoisting (veja o Capítulo 5)

* Declaradores com escopo de bloco (`let` / `const`, veja o Capítulo 6) quando usados no escopo mais externo

Embora possa parecer, ao usar o console/REPL, que instruções digitadas no escopo mais externo estão sendo processadas no escopo global de verdade, isso não é exatamente preciso. Essas ferramentas tipicamente emulam, até certo ponto, a posição do escopo global; é emulação, não aderência estrita. Esses ambientes de ferramentas priorizam a conveniência do desenvolvedor, o que significa que, às vezes (como nas nossas discussões atuais sobre escopo), o comportamento observado pode desviar da especificação do JS.

A conclusão é que as Ferramentas de Desenvolvedor, embora otimizadas para serem convenientes e úteis em uma variedade de atividades de desenvolvimento, **não** são ambientes adequados para determinar ou verificar comportamentos explícitos e sutis do contexto de um programa JS real.

### Módulos ES (ESM)

O ES6 introduziu suporte de primeira classe ao padrão de módulo (coberto no Capítulo 8). Um dos impactos mais óbvios de usar ESM é como isso muda o comportamento do escopo aparentemente de nível superior em um arquivo.

Lembre-se deste trecho de código de antes (que vamos ajustar ao formato ESM usando a palavra-chave `export`):

```js
var studentName = "Kyle";

function hello() {
    console.log(`Hello, ${ studentName }!`);
}

hello();
// Hello, Kyle!

export hello;
```

Se esse código está em um arquivo carregado como módulo ES, ele ainda vai rodar exatamente da mesma forma. Porém, os efeitos observáveis, da perspectiva geral da aplicação, serão diferentes.

Apesar de serem declarados no nível superior do arquivo (módulo), no escopo mais externo aparente, `studentName` e `hello` não são variáveis globais. Em vez disso, são de âmbito do módulo ou, se você preferir, "module-global" (globais do módulo).

Porém, em um módulo não há um "objeto de escopo do módulo" implícito ao qual essas declarações de nível superior sejam adicionadas como propriedades, como acontece quando declarações aparecem no nível superior de arquivos JS que não são módulos. Isso não quer dizer que variáveis globais não possam existir ou ser acessadas nesses programas. É só que variáveis globais não são *criadas* ao declarar variáveis no escopo de nível superior de um módulo.

O escopo de nível superior do módulo descende do escopo global, quase como se todo o conteúdo do módulo estivesse embrulhado em uma função. Assim, todas as variáveis que existem no escopo global (estejam elas no objeto global ou não!) estão disponíveis como identificadores léxicos de dentro do escopo do módulo.

O ESM incentiva uma minimização da dependência do escopo global, em que você importa quaisquer módulos de que o módulo atual precise para operar. Dessa forma, você vê com menos frequência o uso do escopo global ou do seu objeto global.

Porém, como observado antes, ainda há muitos globais do JS e da web que você vai continuar acessando a partir do escopo global, percebendo ou não!

### Node

Um aspecto do Node que frequentemente pega desenvolvedores JS desprevenidos é que o Node trata cada arquivo .js que carrega, inclusive o principal com que você inicia o processo Node, como um *módulo* (módulo ES ou módulo CommonJS, veja o Capítulo 8). O efeito prático é que o nível superior dos seus programas Node **nunca é, de fato, o escopo global**, do jeito que é ao carregar um arquivo que não é módulo no navegador.

No momento em que escrevo, o Node adicionou recentemente suporte a módulos ES. Mas, além disso, o Node suportou desde o início um formato de módulo chamado "CommonJS", que se parece com isto:

```js
var studentName = "Kyle";

function hello() {
    console.log(`Hello, ${ studentName }!`);
}

hello();
// Hello, Kyle!

module.exports.hello = hello;
```

Antes de processar, o Node efetivamente embrulha esse código em uma função, de modo que as declarações `var` e `function` fiquem contidas no escopo dessa função invólucro, **não** tratadas como variáveis globais.

Imagine o código anterior sendo visto pelo Node assim (ilustrativo, não real):

```js
function Module(module,require,__dirname,...) {
    var studentName = "Kyle";

    function hello() {
        console.log(`Hello, ${ studentName }!`);
    }

    hello();
    // Hello, Kyle!

    module.exports.hello = hello;
}
```

O Node então essencialmente invoca a função `Module(..)` adicionada para rodar seu módulo. Você pode ver claramente aqui por que os identificadores `studentName` e `hello` não são globais, e sim declarados no escopo do módulo.

Como observado antes, o Node define uma série de "globais" como `require()`, mas eles não são, de fato, identificadores no escopo global (nem propriedades do objeto global). Eles são injetados no escopo de cada módulo, essencialmente um pouco como os parâmetros listados na declaração da função `Module(..)`.

Então, como você define variáveis globais de verdade no Node? A única forma de fazer isso é adicionar propriedades a outro dos "globais" automaticamente fornecidos pelo Node, que, ironicamente, se chama `global`. `global` é uma referência ao objeto de escopo global de verdade, mais ou menos como usar `window` em um ambiente JS de navegador.

Considere:

```js
global.studentName = "Kyle";

function hello() {
    console.log(`Hello, ${ studentName }!`);
}

hello();
// Hello, Kyle!

module.exports.hello = hello;
```

Aqui adicionamos `studentName` como propriedade do objeto `global` e, então, na instrução `console.log(..)`, conseguimos acessar `studentName` como uma variável global normal.

Lembre-se: o identificador `global` não é definido pelo JS; ele é definido especificamente pelo Node.

## O This Global

Revisando os ambientes JS que vimos até aqui, um programa pode ou não:

* Declarar uma variável global no escopo de nível superior com declarações `var` ou `function` — ou `let`, `const` e `class`.

* Adicionar também declarações de variáveis globais como propriedades do objeto de escopo global, se `var` ou `function` forem usados na declaração.

* Referir-se ao objeto de escopo global (para adicionar ou recuperar variáveis globais, como propriedades) com `window`, `self` ou `global`.

Acho justo dizer que o acesso e o comportamento do escopo global são mais complicados do que a maioria dos desenvolvedores supõe, como as seções anteriores ilustraram. Mas a complexidade nunca é mais óbvia do que ao tentar fixar uma referência universalmente aplicável ao objeto de escopo global.

Mais um "truque" para obter uma referência ao objeto de escopo global se parece com:

```js
const theGlobalScopeObject =
    (new Function("return this"))();
```

| NOTA: |
| :--- |
| Uma função pode ser construída dinamicamente a partir de código armazenado em um valor de string com o construtor `Function()`, de forma parecida com `eval(..)` (veja "Trapaceando: Modificações de Escopo em Tempo de Execução", no Capítulo 1). Tal função automaticamente roda em modo não estrito (por razões legadas) quando invocada com a chamada normal de função `()`, como mostrado; seu `this` vai apontar para o objeto global. Veja o terceiro livro da série, *Objects & Classes*, para mais informações sobre determinar vínculos de `this`. |

Então temos `window`, `self`, `global` e esse feio truque do `new Function(..)`. São muitas formas diferentes de tentar chegar a esse objeto global. Cada uma tem seus prós e contras.

Por que não introduzir mais uma!?!?

A partir do ES2020, o JS finalmente definiu uma referência padronizada ao objeto de escopo global, chamada `globalThis`. Então, dependendo do quão recentes são as engines JS em que seu código roda, você pode usar `globalThis` no lugar de qualquer uma dessas outras abordagens.

Poderíamos até tentar definir um polyfill multiambiente mais seguro em ambientes JS anteriores ao `globalThis`, como:

```js
const theGlobalScopeObject =
    (typeof globalThis != "undefined") ? globalThis :
    (typeof global != "undefined") ? global :
    (typeof window != "undefined") ? window :
    (typeof self != "undefined") ? self :
    (new Function("return this"))();
```

Ufa! Certamente não é o ideal, mas funciona se você se vir precisando de uma referência confiável ao escopo global.

(O nome proposto `globalThis` foi bastante controverso enquanto o recurso estava sendo adicionado ao JS. Especificamente, eu e muitos outros sentimos que a referência a "this" no nome era enganosa, já que a razão para referenciar esse objeto é acessar o escopo global, nunca acessar algum tipo de vínculo de `this` global/padrão. Muitos outros nomes foram considerados, mas descartados por várias razões. Infelizmente, o nome escolhido acabou sendo um último recurso. Se você planeja interagir com o objeto de escopo global nos seus programas, para reduzir a confusão, recomendo fortemente escolher um nome melhor, como o (risivelmente longo, mas preciso!) `theGlobalScopeObject` usado aqui.)

## Globalmente Consciente

O escopo global está presente e é relevante em todo programa JS, ainda que padrões modernos de organizar código em módulos desenfatizem boa parte da dependência de armazenar identificadores nesse namespace.

Ainda assim, conforme nosso código prolifera cada vez mais para além dos limites do navegador, é especialmente importante termos um domínio sólido sobre as diferenças de como o escopo global (e o objeto de escopo global!) se comportam nos diferentes ambientes JS.

Com o panorama geral do escopo global agora mais nítido, o próximo capítulo desce novamente aos detalhes mais profundos do escopo léxico, examinando como e quando variáveis podem ser usadas.
