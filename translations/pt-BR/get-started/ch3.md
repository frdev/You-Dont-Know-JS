# You Don't Know JS Yet: Get Started - 2ª Edição
# Capítulo 3: Cavando Até as Raízes do JS

Se você leu os Capítulos 1 e 2 e tirou um tempo para digerir e deixar decantar, espero que esteja começando a *entender* um pouco melhor o JS. Se você pulou/leu por alto (especialmente o Capítulo 2), recomendo voltar e passar mais tempo com aquele material.

No Capítulo 2, fizemos um panorama de sintaxe, padrões e comportamentos em alto nível. Neste capítulo, nossa atenção se volta para algumas das características-raiz de nível mais baixo do JS, que sustentam virtualmente cada linha de código que escrevemos.

Fique atento: este capítulo cava muito mais fundo do que você provavelmente está acostumado a pensar sobre uma linguagem de programação. Meu objetivo é te ajudar a apreciar o núcleo de como o JS funciona, o que o faz funcionar. Este capítulo deve começar a responder algumas das perguntas do tipo "Por quê?" que podem estar surgindo enquanto você explora o JS. Ainda assim, este material não é uma exposição exaustiva da linguagem; é para isso que serve o restante da série de livros! Nosso objetivo aqui continua sendo apenas *começar* e ficar mais confortável com a *sensação* do JS, como ele flui e reflui.

Não corra por este material tão rápido a ponto de se perder no mato. Como já disse uma dúzia de vezes, **vá com calma**. Mesmo assim, você provavelmente vai terminar este capítulo com perguntas em aberto. Tudo bem, porque há uma série inteira de livros pela frente para continuar explorando!

## Iteração

Como programas são essencialmente construídos para processar dados (e tomar decisões sobre esses dados), os padrões usados para percorrer os dados têm grande impacto na legibilidade do programa.

O padrão iterador existe há décadas e sugere uma abordagem "padronizada" para consumir dados de uma fonte, um *pedaço* por vez. A ideia é que é mais comum e mais útil iterar sobre a fonte de dados — lidar progressivamente com a coleção de dados, processando a primeira parte, depois a próxima, e assim por diante — em vez de lidar com o conjunto inteiro de uma só vez.

Imagine uma estrutura de dados que representa uma consulta `SELECT` em um banco de dados relacional, que tipicamente organiza os resultados como linhas. Se essa consulta tivesse apenas uma ou duas linhas, você poderia lidar com todo o conjunto de resultados de uma vez, atribuir cada linha a uma variável local e realizar quaisquer operações apropriadas sobre esses dados.

Mas, se a consulta tiver 100 ou 1.000 (ou mais!) linhas, você vai precisar de processamento iterativo para lidar com esses dados (tipicamente, um laço).

O padrão iterador define uma estrutura de dados chamada "iterador", que tem uma referência a uma fonte de dados subjacente (como as linhas do resultado da consulta) e que expõe um método como `next()`. Chamar `next()` retorna o próximo pedaço de dados (ou seja, um "registro" ou "linha" de uma consulta ao banco de dados).

Você nem sempre sabe por quantos pedaços de dados vai precisar iterar, então o padrão tipicamente indica a conclusão por meio de algum valor especial ou exceção quando você itera por todo o conjunto e *passa do fim*.

A importância do padrão iterador está em aderir a uma forma *padrão* de processar dados iterativamente, o que cria um código mais limpo e mais fácil de entender, em oposição a cada estrutura/fonte de dados definir sua própria maneira customizada de lidar com seus dados.

Depois de muitos anos de vários esforços da comunidade JS em torno de técnicas de iteração mutuamente acordadas, o ES6 padronizou um protocolo específico para o padrão iterador diretamente na linguagem. O protocolo define um método `next()` cujo retorno é um objeto chamado *resultado do iterador*; esse objeto tem as propriedades `value` e `done`, em que `done` é um booleano que é `false` até que a iteração sobre a fonte de dados subjacente esteja completa.

### Consumindo Iteradores

Com o protocolo de iteração do ES6 no lugar, é viável consumir uma fonte de dados um valor por vez, verificando após cada chamada de `next()` se `done` é `true` para parar a iteração. Mas essa abordagem é bastante manual, então o ES6 também incluiu vários mecanismos (sintaxe e APIs) para o consumo padronizado desses iteradores.

Um desses mecanismos é o laço `for..of`:

```js
// dado um iterador de alguma fonte de dados:
var it = /* .. */;

// percorre seus resultados um por vez
for (let val of it) {
    console.log(`Iterator value: ${ val }`);
}
// Iterator value: ..
// Iterator value: ..
// ..
```

| NOTA: |
| :--- |
| Vamos omitir aqui o equivalente com laço manual, mas ele é definitivamente menos legível do que o laço `for..of`! |

Outro mecanismo frequentemente usado para consumir iteradores é o operador `...`. Esse operador, na verdade, tem duas formas simétricas: *spread* (espalhamento) e *rest* (ou *gather*, "reunir", como prefiro). A forma *spread* é um consumidor de iteradores.

Para fazer *spread* de um iterador, você precisa ter *algum lugar* para espalhá-lo. Há duas possibilidades em JS: um array ou uma lista de argumentos de uma chamada de função.

Um spread em array:

```js
// espalha um iterador em um array,
// com cada valor iterado ocupando
// uma posição de elemento do array.
var vals = [ ...it ];
```

Um spread em chamada de função:

```js
// espalha um iterador em uma função,
// chamando-a com cada valor iterado
// ocupando uma posição de argumento.
doSomethingUseful( ...it );
```

Em ambos os casos, a forma de spread de iterador do `...` segue o protocolo de consumo de iteradores (o mesmo do laço `for..of`) para recuperar todos os valores disponíveis de um iterador e colocá-los (ou seja, espalhá-los) no contexto receptor (array, lista de argumentos).

### Iteráveis

O protocolo de consumo de iteradores é tecnicamente definido para consumir *iteráveis*; um iterável é um valor sobre o qual se pode iterar.

O protocolo cria automaticamente uma instância de iterador a partir de um iterável e consome *apenas aquela instância de iterador* até o fim. Isso significa que um único iterável poderia ser consumido mais de uma vez; a cada vez, uma nova instância de iterador seria criada e usada.

Então, onde encontramos iteráveis?

O ES6 definiu os tipos básicos de estrutura de dados/coleção do JS como iteráveis. Isso inclui strings, arrays, maps, sets e outros.

Considere:

```js
// um array é um iterável
var arr = [ 10, 20, 30 ];

for (let val of arr) {
    console.log(`Array value: ${ val }`);
}
// Array value: 10
// Array value: 20
// Array value: 30
```

Como arrays são iteráveis, podemos fazer uma cópia rasa de um array usando consumo de iterador com o operador spread `...`:

```js
var arrCopy = [ ...arr ];
```

Também podemos iterar sobre os caracteres de uma string, um por vez:

```js
var greeting = "Hello world!";
var chars = [ ...greeting ];

chars;
// [ "H", "e", "l", "l", "o", " ",
//   "w", "o", "r", "l", "d", "!" ]
```

Uma estrutura de dados `Map` usa objetos como chaves, associando um valor (de qualquer tipo) a esse objeto. Maps têm uma iteração padrão diferente da que vimos aqui, no sentido de que a iteração não é apenas sobre os valores do map, mas sim sobre suas *entradas*. Uma *entrada* é uma tupla (array de 2 elementos) que inclui tanto uma chave quanto um valor.

Considere:

```js
// dados dois elementos do DOM, `btn1` e `btn2`

var buttonNames = new Map();
buttonNames.set(btn1,"Button 1");
buttonNames.set(btn2,"Button 2");

for (let [btn,btnName] of buttonNames) {
    btn.addEventListener("click",function onClick(){
        console.log(`Clicked ${ btnName }`);
    });
}
```

No laço `for..of` sobre a iteração padrão do map, usamos a sintaxe `[btn,btnName]` (chamada de "desestruturação de array") para decompor cada tupla consumida nos respectivos pares chave/valor (`btn1` / `"Button 1"` e `btn2` / `"Button 2"`).

Cada um dos iteráveis embutidos no JS expõe uma iteração padrão, que provavelmente corresponde à sua intuição. Mas você também pode escolher uma iteração mais específica, se necessário. Por exemplo, se quisermos consumir apenas os valores do map `buttonNames` acima, podemos chamar `values()` para obter um iterador só de valores:

```js
for (let btnName of buttonNames.values()) {
    console.log(btnName);
}
// Button 1
// Button 2
```

Ou, se quisermos o índice *e* o valor em uma iteração de array, podemos criar um iterador de entradas com o método `entries()`:

```js
var arr = [ 10, 20, 30 ];

for (let [idx,val] of arr.entries()) {
    console.log(`[${ idx }]: ${ val }`);
}
// [0]: 10
// [1]: 20
// [2]: 30
```

Na maior parte, todos os iteráveis embutidos no JS têm três formas de iterador disponíveis: só chaves (`keys()`), só valores (`values()`) e entradas (`entries()`).

Além de apenas usar iteráveis embutidos, você também pode garantir que suas próprias estruturas de dados adiram ao protocolo de iteração; fazer isso significa optar pela capacidade de consumir seus dados com laços `for..of` e com o operador `...`. "Padronizar" nesse protocolo significa código que, no geral, é mais facilmente reconhecível e legível.

| NOTA: |
| :--- |
| Você pode ter notado uma mudança sutil que ocorreu nesta discussão. Começamos falando sobre consumir **iteradores**, mas depois mudamos para falar sobre iterar sobre **iteráveis**. O protocolo de consumo de iteração espera um *iterável*, mas a razão pela qual podemos fornecer um *iterador* diretamente é que um iterador é um iterável de si mesmo! Ao criar uma instância de iterador a partir de um iterador existente, o próprio iterador é retornado. |

## Closure

Talvez sem perceber, quase todo desenvolvedor JS já fez uso de closure. De fato, closure é uma das funcionalidades de programação mais difundidas na maioria das linguagens. Pode ser até tão importante de entender quanto variáveis ou laços; é tão fundamental assim.

Ainda assim, ela parece meio escondida, quase mágica. E frequentemente se fala dela em termos ou muito abstratos ou muito informais, o que pouco ajuda a definir exatamente o que ela é.

Precisamos ser capazes de reconhecer onde closure é usada nos programas, já que a presença ou a ausência de closure é às vezes a causa de bugs (ou até a causa de problemas de desempenho).

Então vamos definir closure de forma pragmática e concreta:

> Closure é quando uma função lembra e continua acessando variáveis de fora do seu escopo, mesmo quando essa função é executada em um escopo diferente.

Vemos duas características definidoras aqui. Primeiro, closure faz parte da natureza de uma função. Objetos não têm closures; funções têm. Segundo, para observar uma closure, você precisa executar uma função em um escopo diferente daquele em que ela foi originalmente definida.

Considere:

```js
function greeting(msg) {
    return function who(name) {
        console.log(`${ msg }, ${ name }!`);
    };
}

var hello = greeting("Hello");
var howdy = greeting("Howdy");

hello("Kyle");
// Hello, Kyle!

hello("Sarah");
// Hello, Sarah!

howdy("Grant");
// Howdy, Grant!
```

Primeiro, a função externa `greeting(..)` é executada, criando uma instância da função interna `who(..)`; essa função fecha (faz closure) sobre a variável `msg`, que é o parâmetro do escopo externo de `greeting(..)`. Quando essa função interna é retornada, sua referência é atribuída à variável `hello` no escopo externo. Depois chamamos `greeting(..)` uma segunda vez, criando uma nova instância da função interna, com uma nova closure sobre um novo `msg`, e retornamos essa referência para ser atribuída a `howdy`.

Quando a função `greeting(..)` termina de rodar, normalmente esperaríamos que todas as suas variáveis fossem coletadas pelo garbage collector (removidas da memória). Esperaríamos que cada `msg` desaparecesse, mas eles não desaparecem. A razão é a closure. Como as instâncias da função interna ainda estão vivas (atribuídas a `hello` e `howdy`, respectivamente), suas closures continuam preservando as variáveis `msg`.

Essas closures não são um instantâneo do valor da variável `msg`; são um vínculo direto e uma preservação da própria variável. Isso significa que a closure pode, de fato, observar (ou fazer!) atualizações nessas variáveis ao longo do tempo.

```js
function counter(step = 1) {
    var count = 0;
    return function increaseCount(){
        count = count + step;
        return count;
    };
}

var incBy1 = counter(1);
var incBy3 = counter(3);

incBy1();       // 1
incBy1();       // 2

incBy3();       // 3
incBy3();       // 6
incBy3();       // 9
```

Cada instância da função interna `increaseCount()` faz closure tanto sobre a variável `count` quanto sobre `step`, do escopo da sua função externa `counter(..)`. `step` permanece o mesmo ao longo do tempo, mas `count` é atualizado a cada invocação daquela função interna. Como a closure é sobre as variáveis, e não apenas sobre instantâneos dos valores, essas atualizações são preservadas.

Closure é mais comum ao trabalhar com código assíncrono, como com callbacks. Considere:

```js
function getSomeData(url) {
    ajax(url,function onResponse(resp){
        console.log(
            `Response (from ${ url }): ${ resp }`
        );
    });
}

getSomeData("https://some.url/wherever");
// Response (from https://some.url/wherever): ...
```

A função interna `onResponse(..)` faz closure sobre `url` e, portanto, o preserva e o lembra até que a chamada Ajax retorne e execute `onResponse(..)`. Mesmo que `getSomeData(..)` termine imediatamente, a variável de parâmetro `url` é mantida viva na closure pelo tempo que for necessário.

Não é necessário que o escopo externo seja uma função — normalmente é, mas nem sempre — apenas que haja pelo menos uma variável em um escopo externo acessada a partir de uma função interna:

```js
for (let [idx,btn] of buttons.entries()) {
    btn.addEventListener("click",function onClick(){
       console.log(`Clicked on button (${ idx })!`);
    });
}
```

Como este laço usa declarações `let`, cada iteração recebe novas variáveis `idx` e `btn` com escopo de bloco (ou seja, locais); o laço também cria uma nova função interna `onClick(..)` a cada vez. Essa função interna faz closure sobre `idx`, preservando-o enquanto o manipulador de clique estiver definido no `btn`. Assim, quando cada botão é clicado, seu manipulador pode imprimir o valor de índice associado, porque o manipulador lembra sua respectiva variável `idx`.

Lembre-se: essa closure não é sobre o valor (como `1` ou `3`), mas sobre a própria variável `idx`.

Closure é um dos padrões de programação mais prevalentes e importantes em qualquer linguagem. Mas isso é especialmente verdadeiro no JS; é difícil imaginar fazer algo útil sem aproveitar closure de uma forma ou de outra.

Se você ainda se sente confuso ou inseguro sobre closure, a maior parte do Livro 2, *Scope & Closures*, é focada nesse tópico.

## A Palavra-chave `this`

Um dos mecanismos mais poderosos do JS também é um dos mais mal compreendidos: a palavra-chave `this`. Um equívoco comum é que o `this` de uma função se refere à própria função. Por causa de como `this` funciona em outras linguagens, outro equívoco é que `this` aponta para a instância à qual um método pertence. Ambos estão incorretos.

Como discutido anteriormente, quando uma função é definida, ela é *anexada* ao seu escopo envolvente por meio de closure. Escopo é o conjunto de regras que controla como referências a variáveis são resolvidas.

Mas funções também têm outra característica, além do seu escopo, que influencia o que elas podem acessar. Essa característica é mais bem descrita como um *contexto de execução*, e é exposta à função por meio da palavra-chave `this`.

Escopo é estático e contém um conjunto fixo de variáveis disponíveis no momento e no local em que você define uma função, mas o *contexto* de execução de uma função é dinâmico, inteiramente dependente de **como ela é chamada** (independentemente de onde ela é definida ou até de onde é chamada).

`this` não é uma característica fixa de uma função baseada na definição dela, e sim uma característica dinâmica, determinada a cada vez que a função é chamada.

Uma forma de pensar no *contexto de execução* é como um objeto tangível cujas propriedades são disponibilizadas a uma função enquanto ela executa. Compare isso com o escopo, que também pode ser pensado como um *objeto*; exceto que o *objeto de escopo* fica oculto dentro da engine JS, é sempre o mesmo para aquela função, e suas *propriedades* assumem a forma de variáveis identificadoras disponíveis dentro da função.

```js
function classroom(teacher) {
    return function study() {
        console.log(
            `${ teacher } says to study ${ this.topic }`
        );
    };
}
var assignment = classroom("Kyle");
```

A função externa `classroom(..)` não faz nenhuma referência à palavra-chave `this`, então ela é como qualquer outra função que vimos até agora. Mas a função interna `study()` referencia `this`, o que a torna uma função ciente de `this`. Em outras palavras, é uma função dependente do seu *contexto de execução*.

| NOTA: |
| :--- |
| `study()` também faz closure sobre a variável `teacher` do seu escopo externo. |

A função interna `study()` retornada por `classroom("Kyle")` é atribuída a uma variável chamada `assignment`. Então, como `assignment()` (ou seja, `study()`) pode ser chamada?

```js
assignment();
// Kyle says to study undefined  -- Ops :(
```

Neste trecho, chamamos `assignment()` como uma função simples e normal, sem fornecer a ela nenhum *contexto de execução*.

Como este programa não está em modo estrito (veja o Capítulo 1, "Falando Estritamente"), funções cientes de contexto que são chamadas **sem nenhum contexto especificado** assumem por padrão o objeto global como contexto (`window` no navegador). Como não existe uma variável global chamada `topic` (e, portanto, nenhuma propriedade desse nome no objeto global), `this.topic` resolve para `undefined`.

Agora considere:

```js
var homework = {
    topic: "JS",
    assignment: assignment
};

homework.assignment();
// Kyle says to study JS
```

Uma cópia da referência à função `assignment` é definida como uma propriedade do objeto `homework`, e então é chamada como `homework.assignment()`. Isso significa que o `this` daquela chamada de função será o objeto `homework`. Portanto, `this.topic` resolve para `"JS"`.

Por fim:

```js
var otherHomework = {
    topic: "Math"
};

assignment.call(otherHomework);
// Kyle says to study Math
```

Uma terceira maneira de invocar uma função é com o método `call(..)`, que recebe um objeto (`otherHomework`, aqui) para definir a referência de `this` naquela chamada de função. A referência de propriedade `this.topic` resolve para `"Math"`.

A mesma função ciente de contexto, invocada de três formas diferentes, dá respostas diferentes a cada vez sobre qual objeto `this` vai referenciar.

O benefício de funções cientes de `this` — e do seu contexto dinâmico — é a capacidade de reutilizar uma única função de forma mais flexível com dados de objetos diferentes. Uma função que faz closure sobre um escopo nunca pode referenciar um escopo ou conjunto de variáveis diferente. Mas uma função com consciência dinâmica de contexto via `this` pode ser bastante útil para certas tarefas.

## Protótipos

Enquanto `this` é uma característica da execução de funções, um protótipo é uma característica de um objeto e, especificamente, da resolução de um acesso a propriedade.

Pense em um protótipo como um vínculo entre dois objetos; o vínculo fica escondido nos bastidores, embora existam formas de expô-lo e observá-lo. Esse vínculo de protótipo ocorre quando um objeto é criado; ele é vinculado a outro objeto que já existe.

Uma série de objetos vinculados entre si por meio de protótipos é chamada de "cadeia de protótipos" (*prototype chain*).

O propósito desse vínculo de protótipo (ou seja, de um objeto B para outro objeto A) é que acessos em B a propriedades/métodos que B não possui sejam *delegados* a A para serem tratados. A delegação de acesso a propriedades/métodos permite que dois (ou mais!) objetos cooperem entre si para realizar uma tarefa.

Considere definir um objeto como um literal normal:

```js
var homework = {
    topic: "JS"
};
```

O objeto `homework` tem apenas uma única propriedade: `topic`. Porém, seu vínculo de protótipo padrão se conecta ao objeto `Object.prototype`, que tem métodos embutidos comuns como `toString()` e `valueOf()`, entre outros.

Podemos observar essa *delegação* por vínculo de protótipo de `homework` para `Object.prototype`:

```js
homework.toString();    // [object Object]
```

`homework.toString()` funciona mesmo que `homework` não tenha um método `toString()` definido; a delegação invoca `Object.prototype.toString()` no lugar.

### Vínculo entre Objetos

Para definir um vínculo de protótipo entre objetos, você pode criar o objeto usando o utilitário `Object.create(..)`:

```js
var homework = {
    topic: "JS"
};

var otherHomework = Object.create(homework);

otherHomework.topic;   // "JS"
```

O primeiro argumento de `Object.create(..)` especifica um objeto ao qual vincular o objeto recém-criado, e então retorna o objeto recém-criado (e vinculado!).

A Figura 4 mostra como os três objetos (`otherHomework`, `homework` e `Object.prototype`) estão vinculados em uma cadeia de protótipos:

<figure>
    <img src="../../../get-started/images/fig4.png" width="200" alt="Cadeia de protótipos com 3 objetos" align="center">
    <figcaption><em>Fig. 4: Objetos em uma cadeia de protótipos</em></figcaption>
    <br><br>
</figure>

A delegação pela cadeia de protótipos se aplica apenas a acessos para buscar o valor de uma propriedade. Se você atribuir a uma propriedade de um objeto, isso se aplica diretamente ao objeto, independentemente de onde esse objeto esteja vinculado por protótipo.

| DICA: |
| :--- |
| `Object.create(null)` cria um objeto que não está vinculado por protótipo a lugar nenhum, então ele é puramente um objeto independente; em algumas circunstâncias, isso pode ser preferível. |

Considere:

```js
homework.topic;
// "JS"

otherHomework.topic;
// "JS"

otherHomework.topic = "Math";
otherHomework.topic;
// "Math"

homework.topic;
// "JS" -- e não "Math"
```

A atribuição a `topic` cria uma propriedade com esse nome diretamente em `otherHomework`; não há efeito sobre a propriedade `topic` em `homework`. A instrução seguinte então acessa `otherHomework.topic`, e vemos a resposta não delegada, vinda dessa nova propriedade: `"Math"`.

A Figura 5 mostra os objetos/propriedades depois da atribuição que cria a propriedade `otherHomework.topic`:

<figure>
    <img src="../../../get-started/images/fig5.png" width="200" alt="3 objetos vinculados, com propriedade sombreada" align="center">
    <figcaption><em>Fig. 5: Propriedade 'topic' sombreada</em></figcaption>
    <br><br>
</figure>

O `topic` em `otherHomework` está "sombreando" (*shadowing*) a propriedade de mesmo nome no objeto `homework` da cadeia.

| NOTA: |
| :--- |
| Outra forma, francamente mais rebuscada mas talvez ainda mais comum, de criar um objeto com vínculo de protótipo é usando o padrão de "classe prototipal", de antes de `class` (veja o Capítulo 2, "Classes") ser adicionado no ES6. Vamos cobrir esse tópico com mais detalhes no Apêndice A, "'Classes' Prototipais". |

### `this` Revisitado

Cobrimos a palavra-chave `this` antes, mas sua verdadeira importância brilha quando consideramos como ela dá poder a chamadas de função delegadas por protótipo. De fato, uma das principais razões pelas quais `this` suporta contexto dinâmico com base em como a função é chamada é para que chamadas de método em objetos que delegam pela cadeia de protótipos ainda mantenham o `this` esperado.

Considere:

```js
var homework = {
    study() {
        console.log(`Please study ${ this.topic }`);
    }
};

var jsHomework = Object.create(homework);
jsHomework.topic = "JS";
jsHomework.study();
// Please study JS

var mathHomework = Object.create(homework);
mathHomework.topic = "Math";
mathHomework.study();
// Please study Math
```

Os dois objetos `jsHomework` e `mathHomework` estão, cada um, vinculados por protótipo ao único objeto `homework`, que tem a função `study()`. `jsHomework` e `mathHomework` recebem, cada um, sua própria propriedade `topic` (veja a Figura 6).

<figure>
    <img src="../../../get-started/images/fig6.png" width="495" alt="4 objetos vinculados por protótipo" align="center">
    <figcaption><em>Fig. 6: Dois objetos vinculados a um pai comum</em></figcaption>
    <br><br>
</figure>

`jsHomework.study()` delega para `homework.study()`, mas seu `this` (`this.topic`) naquela execução resolve para `jsHomework`, por causa de como a função é chamada, então `this.topic` é `"JS"`. Da mesma forma, `mathHomework.study()` delega para `homework.study()`, mas ainda resolve `this` para `mathHomework` e, portanto, `this.topic` como `"Math"`.

O trecho de código anterior seria bem menos útil se `this` fosse resolvido para `homework`. Ainda assim, em muitas outras linguagens, pareceria que `this` seria `homework`, porque o método `study()` está, de fato, definido em `homework`.

Diferente de muitas outras linguagens, o fato de o `this` do JS ser dinâmico é um componente crítico para permitir que a delegação por protótipo — e, de fato, `class` — funcione como esperado!

## Perguntando "Por Quê?"

A conclusão pretendida deste capítulo é que existe muito mais no JS sob o capô do que é óbvio ao dar uma olhada na superfície.

Enquanto você está *começando* a aprender e conhecer o JS mais de perto, uma das habilidades mais importantes que você pode praticar e fortalecer é a curiosidade, e a arte de perguntar "Por quê?" quando encontra algo na linguagem.

Embora este capítulo tenha ido bastante fundo em alguns tópicos, muitos detalhes ainda foram completamente superficiais. Há muito mais a aprender aqui, e o caminho para isso começa com você fazendo as perguntas *certas* ao seu código. Fazer as perguntas certas é uma habilidade crítica para se tornar um desenvolvedor melhor.

No capítulo final deste livro, vamos olhar brevemente como o JS é dividido, conforme coberto ao longo do restante da série de livros *You Don't Know JS Yet*. Além disso, não pule o Apêndice B deste livro, que tem código de prática para revisar alguns dos principais tópicos cobertos aqui.
