# You Don't Know JS Yet: Scope & Closures - 2ª Edição
# Apêndice B: Prática

Este apêndice tem como objetivo te dar alguns exercícios desafiadores e interessantes para testar e solidificar seu entendimento dos principais tópicos deste livro. É uma boa ideia tentar fazer os exercícios você mesmo — em um editor de código de verdade! — em vez de pular direto para as soluções no fim. Nada de trapaça!

Estes exercícios não têm uma resposta certa específica que você precise acertar exatamente. Sua abordagem pode diferir um pouco (ou muito!) das soluções apresentadas, e tudo bem.

Ninguém está te julgando pelo jeito como você escreve seu código. Minha esperança é que você termine este livro sentindo confiança de que consegue encarar esse tipo de tarefa de programação, apoiado em uma base sólida de conhecimento. Esse é o único objetivo aqui. Se você está feliz com seu código, eu também estou!

## Baldes de Bolinhas

Lembra da Figura 2, lá do Capítulo 2?

<figure>
    <img src="../../../scope-closures/images/fig2.png" width="300" alt="Bolhas de Escopo Coloridas" align="center">
    <figcaption><em>Fig. 2 (Cap. 2): Bolhas de Escopo Coloridas</em></figcaption>
    <br><br>
</figure>

Este exercício pede que você escreva um programa — qualquer programa! — que contenha funções aninhadas e escopos de bloco, satisfazendo estas restrições:

* Se você colorir todos os escopos (incluindo o escopo global!) com cores diferentes, vai precisar de pelo menos seis cores. Certifique-se de adicionar um comentário de código rotulando cada escopo com sua cor.

    BÔNUS: identifique quaisquer escopos implícitos que seu código possa ter.

* Cada escopo tem pelo menos um identificador.

* Contém pelo menos dois escopos de função e pelo menos dois escopos de bloco.

* Ao menos uma variável de um escopo externo precisa ser sombreada por uma variável de escopo aninhado (veja o Capítulo 3).

* Ao menos uma referência a variável precisa resolver para uma declaração de variável pelo menos dois níveis acima na cadeia de escopos.

| DICA: |
| :--- |
| Você *pode* simplesmente escrever código besta do tipo foo/bar/baz para este exercício, mas sugiro que tente criar algum tipo de código não trivial e mais "real", que pelo menos faça algo meio razoável. |

Tente o exercício você mesmo e depois confira a solução sugerida no fim deste apêndice.

## Closure (PARTE 1)

Vamos primeiro praticar closure com algumas operações comuns de matemática computacional: determinar se um valor é primo (não tem divisores além de 1 e dele mesmo) e gerar uma lista de fatores primos (divisores) de um dado número.

Por exemplo:

```js
isPrime(11);        // true
isPrime(12);        // false

factorize(11);      // [ 11 ]
factorize(12);      // [ 3, 2, 2 ] --> 3*2*2=12
```

Aqui está uma implementação de `isPrime(..)`, adaptada da biblioteca Math.js: [^MathJSisPrime]

```js
function isPrime(v) {
    if (v <= 3) {
        return v > 1;
    }
    if (v % 2 == 0 || v % 3 == 0) {
        return false;
    }
    var vSqrt = Math.sqrt(v);
    for (let i = 5; i <= vSqrt; i += 6) {
        if (v % i == 0 || v % (i + 2) == 0) {
            return false;
        }
    }
    return true;
}
```

E aqui está uma implementação um tanto básica de `factorize(..)` (não confundir com `factorial(..)`, do Capítulo 6):

```js
function factorize(v) {
    if (!isPrime(v)) {
        let i = Math.floor(Math.sqrt(v));
        while (v % i != 0) {
            i--;
        }
        return [
            ...factorize(i),
            ...factorize(v / i)
        ];
    }
    return [v];
}
```

| NOTA: |
| :--- |
| Chamo isso de básico porque não é otimizado para desempenho. É recursivo binário (o que não é otimizável por tail-call) e cria muitas cópias intermediárias de arrays. Também não ordena os fatores descobertos de forma alguma. Existem muitos, muitos outros algoritmos para essa tarefa, mas quis usar algo curto e mais ou menos compreensível para o nosso exercício. |

Se você chamasse `isPrime(4327)` múltiplas vezes em um programa, veria que ele passaria por todas as suas dezenas de passos de comparação/computação toda vez. Se considerarmos `factorize(..)`, ele chama `isPrime(..)` muitas vezes enquanto computa a lista de fatores. E há uma boa chance de a maioria dessas chamadas ser repetida. É muito trabalho desperdiçado!

A primeira parte deste exercício é usar closure para implementar um cache que lembre os resultados de `isPrime(..)`, de modo que a primalidade (`true` ou `false`) de um dado número seja computada apenas uma vez. Dica: já mostramos esse tipo de cache no Capítulo 6, com `factorial(..)`.

Se você olhar `factorize(..)`, ele é implementado com recursão, ou seja, chama a si mesmo repetidamente. Isso, de novo, significa que provavelmente veremos muitas chamadas desperdiçadas para computar fatores primos do mesmo número. Então a segunda parte do exercício é usar a mesma técnica de cache com closure para `factorize(..)`.

Use closures separadas para o cache de `isPrime(..)` e de `factorize(..)`, em vez de colocá-los dentro de um único escopo.

Tente o exercício você mesmo e depois confira a solução sugerida no fim deste apêndice.

### Uma Palavra Sobre Memória

Quero compartilhar uma nota rápida sobre essa técnica de cache com closure e os impactos que ela tem no desempenho da sua aplicação.

Podemos ver que, ao poupar as chamadas repetidas, melhoramos a velocidade de computação (em alguns casos, de forma dramática). Mas esse uso de closure está fazendo um trade-off explícito do qual você deveria estar bem ciente.

O trade-off é memória. Estamos essencialmente fazendo nosso cache crescer (na memória) de forma ilimitada. Se as funções em questão fossem chamadas muitos milhões de vezes com entradas majoritariamente únicas, estaríamos devorando muita memória. Isso pode definitivamente valer o custo, mas apenas se acharmos provável ver repetição de entradas comuns, de modo a de fato aproveitar o cache.

Se quase toda chamada tiver uma entrada única, e o cache essencialmente nunca for *usado* com benefício, esta é uma técnica inapropriada de se empregar.

Também pode ser uma boa ideia ter uma abordagem de cache mais sofisticada, como um cache LRU (*least recently used*, menos recentemente usado), que limita seu tamanho; conforme se aproxima do limite, um LRU descarta os valores que são... bem, menos recentemente usados!

A desvantagem aqui é que o LRU é bastante não trivial por si só. Você vai querer usar uma implementação altamente otimizada de LRU e estar bem atento a todos os trade-offs em jogo.

## Closure (PARTE 2)

Neste exercício, vamos praticar closure novamente, definindo um utilitário `toggle(..)` que nos dá um alternador de valores.

Você vai passar um ou mais valores (como argumentos) para `toggle(..)` e receber de volta uma função. Essa função retornada vai alternar/rodar entre todos os valores passados, em ordem, um por vez, conforme for chamada repetidamente.

```js
function toggle(/* .. */) {
    // ..
}

var hello = toggle("hello");
var onOff = toggle("on","off");
var speed = toggle("slow","medium","fast");

hello();      // "hello"
hello();      // "hello"

onOff();      // "on"
onOff();      // "off"
onOff();      // "on"

speed();      // "slow"
speed();      // "medium"
speed();      // "fast"
speed();      // "slow"
```

O caso de borda de não passar nenhum valor para `toggle(..)` não é muito importante; tal instância de alternador poderia simplesmente sempre retornar `undefined`.

Tente o exercício você mesmo e depois confira a solução sugerida no fim deste apêndice.

## Closure (PARTE 3)

Neste terceiro e último exercício sobre closure, vamos implementar uma calculadora básica. A função `calculator()` vai produzir uma instância de calculadora que mantém seu próprio estado, na forma de uma função (`calc(..)`, abaixo):

```js
function calculator() {
    // ..
}

var calc = calculator();
```

Cada vez que `calc(..)` é chamada, você vai passar um único caractere que representa o pressionar de um botão da calculadora. Para manter as coisas mais diretas, vamos restringir nossa calculadora a suportar apenas a entrada de dígitos (0-9), operações aritméticas (+, -, \*, /) e "=" para computar a operação. As operações são processadas estritamente na ordem em que são digitadas; não há agrupamento com "( )" nem precedência de operadores.

Não suportamos a entrada de decimais, mas a operação de divisão pode resultar neles. Não suportamos a entrada de números negativos, mas a operação "-" pode resultar neles. Então, você deve conseguir produzir qualquer número negativo ou decimal digitando primeiro uma operação que o compute. Você pode então continuar computando com esse valor.

O retorno das chamadas a `calc(..)` deve imitar o que seria mostrado em uma calculadora de verdade, como refletir o que acabou de ser pressionado, ou computar o total ao pressionar "=".

Por exemplo:

```js
calc("4");     // 4
calc("+");     // +
calc("7");     // 7
calc("3");     // 3
calc("-");     // -
calc("2");     // 2
calc("=");     // 75
calc("*");     // *
calc("4");     // 4
calc("=");     // 300
calc("5");     // 5
calc("-");     // -
calc("5");     // 5
calc("=");     // 0
```

Como esse uso é um pouco desajeitado, aqui está um auxiliar `useCalc(..)`, que roda a calculadora com caracteres um por vez a partir de uma string e computa o display a cada vez:

```js
function useCalc(calc,keys) {
    return [...keys].reduce(
        function showDisplay(display,key){
            var ret = String( calc(key) );
            return (
                display +
                (
                  (ret != "" && key == "=") ?
                      "=" :
                      ""
                ) +
                ret
            );
        },
        ""
    );
}

useCalc(calc,"4+3=");           // 4+3=7
useCalc(calc,"+9=");            // +9=16
useCalc(calc,"*8=");            // *5=128
useCalc(calc,"7*2*3=");         // 7*2*3=42
useCalc(calc,"1/0=");           // 1/0=ERR
useCalc(calc,"+3=");            // +3=ERR
useCalc(calc,"51=");            // 51
```

O uso mais sensato deste auxiliar `useCalc(..)` é sempre ter "=" como último caractere digitado.

Parte da formatação dos totais exibidos pela calculadora exige tratamento especial. Estou fornecendo esta função `formatTotal(..)`, que sua calculadora deve usar sempre que for retornar um total computado atual (depois que um `"="` é digitado):

```js
function formatTotal(display) {
    if (Number.isFinite(display)) {
        // limita o display a no máximo 11 caracteres
        let maxDigits = 11;
        // reserva espaço para a notação "e+"?
        if (Math.abs(display) > 99999999999) {
            maxDigits -= 6;
        }
        // reserva espaço para "-"?
        if (display < 0) {
            maxDigits--;
        }

        // número inteiro?
        if (Number.isInteger(display)) {
            display = display
                .toPrecision(maxDigits)
                .replace(/\.0+$/,"");
        }
        // decimal
        else {
            // reserva espaço para "."
            maxDigits--;
            // reserva espaço para o "0" inicial?
            if (
                Math.abs(display) >= 0 &&
                Math.abs(display) < 1
            ) {
                maxDigits--;
            }
            display = display
                .toPrecision(maxDigits)
                .replace(/0+$/,"");
        }
    }
    else {
        display = "ERR";
    }
    return display;
}
```

Não se preocupe demais com como `formatTotal(..)` funciona. A maior parte da sua lógica é um monte de tratamento para limitar o display da calculadora a no máximo 11 caracteres, mesmo que sejam necessários negativos, dízimas periódicas ou até notação exponencial "e+".

De novo, não se atole demais no comportamento específico de calculadora. Foque na *memória* da closure.

Tente o exercício você mesmo e depois confira a solução sugerida no fim deste apêndice.

## Módulos

Este exercício é converter a calculadora de Closure (PARTE 3) em um módulo.

Não estamos adicionando nenhuma funcionalidade extra à calculadora, apenas mudando sua interface. Em vez de chamar uma única função `calc(..)`, vamos chamar métodos específicos da API pública para cada "tecla pressionada" da nossa calculadora. As saídas permanecem as mesmas.

Este módulo deve ser expresso como uma função fábrica de módulo clássico chamada `calculator()`, em vez de uma IIFE singleton, para que múltiplas calculadoras possam ser criadas, se desejado.

A API pública deve incluir os seguintes métodos:

* `number(..)` (entrada: o caractere/número "pressionado")
* `plus()`
* `minus()`
* `mult()`
* `div()`
* `eq()`

O uso ficaria assim:

```js
var calc = calculator();

calc.number("4");     // 4
calc.plus();          // +
calc.number("7");     // 7
calc.number("3");     // 3
calc.minus();         // -
calc.number("2");     // 2
calc.eq();            // 75
```

`formatTotal(..)` continua a mesma do exercício anterior. Mas o auxiliar `useCalc(..)` precisa ser ajustado para funcionar com a API do módulo:

```js
function useCalc(calc,keys) {
    var keyMappings = {
        "+": "plus",
        "-": "minus",
        "*": "mult",
        "/": "div",
        "=": "eq"
    };

    return [...keys].reduce(
        function showDisplay(display,key){
            var fn = keyMappings[key] || "number";
            var ret = String( calc[fn](key) );
            return (
                display +
                (
                  (ret != "" && key == "=") ?
                      "=" :
                      ""
                ) +
                ret
            );
        },
        ""
    );
}

useCalc(calc,"4+3=");           // 4+3=7
useCalc(calc,"+9=");            // +9=16
useCalc(calc,"*8=");            // *5=128
useCalc(calc,"7*2*3=");         // 7*2*3=42
useCalc(calc,"1/0=");           // 1/0=ERR
useCalc(calc,"+3=");            // +3=ERR
useCalc(calc,"51=");            // 51
```

Tente o exercício você mesmo e depois confira a solução sugerida no fim deste apêndice.

Enquanto trabalha neste exercício, dedique também algum tempo a considerar os prós/contras de representar a calculadora como um módulo, em oposição à abordagem de função com closure do exercício anterior.

BÔNUS: escreva algumas frases explicando o que você pensa.

BÔNUS #2: tente converter seu módulo para outros formatos de módulo, incluindo: UMD, CommonJS e ESM (Módulos ES).

## Soluções Sugeridas

Espero que você tenha tentado os exercícios antes de estar lendo tão longe. Nada de trapaça!

Lembre-se: cada solução sugerida é apenas uma dentre várias formas diferentes de abordar os problemas. Elas não são "a resposta certa", mas ilustram uma forma razoável de encarar cada exercício.

O benefício mais importante que você pode obter lendo estas soluções sugeridas é compará-las com seu código e analisar por que cada um de nós fez escolhas parecidas ou diferentes. Não entre em muito bikeshedding; tente se manter focado no tópico principal, e não nos pequenos detalhes.

### Sugerida: Baldes de Bolinhas

O *Exercício dos Baldes de Bolinhas* pode ser resolvido assim:

```js
// VERMELHO(1)
const howMany = 100;

// Crivo de Eratóstenes
function findPrimes(howMany) {
    // AZUL(2)
    var sieve = Array(howMany).fill(true);
    var max = Math.sqrt(howMany);

    for (let i = 2; i < max; i++) {
        // VERDE(3)
        if (sieve[i]) {
            // LARANJA(4)
            let j = Math.pow(i,2);
            for (let k = j; k < howMany; k += i) {
                // ROXO(5)
                sieve[k] = false;
            }
        }
    }

    return sieve
        .map(function getPrime(flag,prime){
            // ROSA(6)
            if (flag) return prime;
            return flag;
        })
        .filter(function onlyPrimes(v){
            // AMARELO(7)
            return !!v;
        })
        .slice(1);
}

findPrimes(howMany);
// [
//    2, 3, 5, 7, 11, 13, 17,
//    19, 23, 29, 31, 37, 41,
//    43, 47, 53, 59, 61, 67,
//    71, 73, 79, 83, 89, 97
// ]
```

### Sugerida: Closure (PARTE 1)

O *Exercício de Closure (PARTE 1)*, para `isPrime(..)` e `factorize(..)`, pode ser resolvido assim:

```js
var isPrime = (function isPrime(v){
    var primes = {};

    return function isPrime(v) {
        if (v in primes) {
            return primes[v];
        }
        if (v <= 3) {
            return (primes[v] = v > 1);
        }
        if (v % 2 == 0 || v % 3 == 0) {
            return (primes[v] = false);
        }
        let vSqrt = Math.sqrt(v);
        for (let i = 5; i <= vSqrt; i += 6) {
            if (v % i == 0 || v % (i + 2) == 0) {
                return (primes[v] = false);
            }
        }
        return (primes[v] = true);
    };
})();

var factorize = (function factorize(v){
    var factors = {};

    return function findFactors(v) {
        if (v in factors) {
            return factors[v];
        }
        if (!isPrime(v)) {
            let i = Math.floor(Math.sqrt(v));
            while (v % i != 0) {
                i--;
            }
            return (factors[v] = [
                ...findFactors(i),
                ...findFactors(v / i)
            ]);
        }
        return (factors[v] = [v]);
    };
})();
```

Os passos gerais que usei para cada utilitário:

1. Envolver em uma IIFE para definir o escopo em que a variável de cache vai residir.

2. Na chamada subjacente, primeiro verificar o cache e, se um resultado já é conhecido, retorná-lo.

3. Em cada lugar em que um `return` acontecia originalmente, atribuir ao cache e simplesmente retornar o resultado dessa operação de atribuição — isso é um truque de economia de espaço, principalmente por brevidade no livro.

Também renomeei a função interna de `factorize(..)` para `findFactors(..)`. Isso não é tecnicamente necessário, mas ajuda a deixar mais claro qual função as chamadas recursivas invocam.

### Sugerida: Closure (PARTE 2)

O `toggle(..)` do *Exercício de Closure (PARTE 2)* pode ser resolvido assim:

```js
function toggle(...vals) {
    var unset = {};
    var cur = unset;

    return function next(){
        // salva o valor anterior de volta
        // no fim da lista
        if (cur != unset) {
            vals.push(cur);
        }
        cur = vals.shift();
        return cur;
    };
}

var hello = toggle("hello");
var onOff = toggle("on","off");
var speed = toggle("slow","medium","fast");

hello();      // "hello"
hello();      // "hello"

onOff();      // "on"
onOff();      // "off"
onOff();      // "on"

speed();      // "slow"
speed();      // "medium"
speed();      // "fast"
speed();      // "slow"
```

### Sugerida: Closure (PARTE 3)

O `calculator()` do *Exercício de Closure (PARTE 3)* pode ser resolvido assim:

```js
// de antes:
//
// function useCalc(..) { .. }
// function formatTotal(..) { .. }

function calculator() {
    var currentTotal = 0;
    var currentVal = "";
    var currentOper = "=";

    return pressKey;

    // ********************

    function pressKey(key){
        // tecla numérica?
        if (/\d/.test(key)) {
            currentVal += key;
            return key;
        }
        // tecla de operador?
        else if (/[+*/-]/.test(key)) {
            // múltiplas operações em sequência?
            if (
                currentOper != "=" &&
                currentVal != ""
            ) {
                // pressionar de '=' implícito
                pressKey("=");
            }
            else if (currentVal != "") {
                currentTotal = Number(currentVal);
            }
            currentOper = key;
            currentVal = "";
            return key;
        }
        // tecla =?
        else if (
            key == "=" &&
            currentOper != "="
        ) {
            currentTotal = op(
                currentTotal,
                currentOper,
                Number(currentVal)
            );
            currentOper = "=";
            currentVal = "";
            return formatTotal(currentTotal);
        }
        return "";
    };

    function op(val1,oper,val2) {
        var ops = {
            // NOTA: usando arrow functions
            // apenas por brevidade no livro
            "+": (v1,v2) => v1 + v2,
            "-": (v1,v2) => v1 - v2,
            "*": (v1,v2) => v1 * v2,
            "/": (v1,v2) => v1 / v2
        };
        return ops[oper](val1,val2);
    }
}

var calc = calculator();

useCalc(calc,"4+3=");           // 4+3=7
useCalc(calc,"+9=");            // +9=16
useCalc(calc,"*8=");            // *5=128
useCalc(calc,"7*2*3=");         // 7*2*3=42
useCalc(calc,"1/0=");           // 1/0=ERR
useCalc(calc,"+3=");            // +3=ERR
useCalc(calc,"51=");            // 51
```

| NOTA: |
| :--- |
| Lembre-se: este exercício é sobre closure. Não foque demais na mecânica real de uma calculadora, e sim em se você está *lembrando* corretamente o estado da calculadora entre chamadas de função. |

### Sugerida: Módulos

O `calculator()` do *Exercício de Módulos* pode ser resolvido assim:

```js
// de antes:
//
// function useCalc(..) { .. }
// function formatTotal(..) { .. }

function calculator() {
    var currentTotal = 0;
    var currentVal = "";
    var currentOper = "=";

    var publicAPI = {
        number,
        eq,
        plus() { return operator("+"); },
        minus() { return operator("-"); },
        mult() { return operator("*"); },
        div() { return operator("/"); }
    };

    return publicAPI;

    // ********************

    function number(key) {
        // tecla numérica?
        if (/\d/.test(key)) {
            currentVal += key;
            return key;
        }
    }

    function eq() {
        // tecla =?
        if (currentOper != "=") {
            currentTotal = op(
                currentTotal,
                currentOper,
                Number(currentVal)
            );
            currentOper = "=";
            currentVal = "";
            return formatTotal(currentTotal);
        }
        return "";
    }

    function operator(key) {
        // múltiplas operações em sequência?
        if (
            currentOper != "=" &&
            currentVal != ""
        ) {
            // pressionar de '=' implícito
            eq();
        }
        else if (currentVal != "") {
            currentTotal = Number(currentVal);
        }
        currentOper = key;
        currentVal = "";
        return key;
    }

    function op(val1,oper,val2) {
        var ops = {
            // NOTA: usando arrow functions
            // apenas por brevidade no livro
            "+": (v1,v2) => v1 + v2,
            "-": (v1,v2) => v1 - v2,
            "*": (v1,v2) => v1 * v2,
            "/": (v1,v2) => v1 / v2
        };
        return ops[oper](val1,val2);
    }
}

var calc = calculator();

useCalc(calc,"4+3=");           // 4+3=7
useCalc(calc,"+9=");            // +9=16
useCalc(calc,"*8=");            // *5=128
useCalc(calc,"7*2*3=");         // 7*2*3=42
useCalc(calc,"1/0=");           // 1/0=ERR
useCalc(calc,"+3=");            // +3=ERR
useCalc(calc,"51=");            // 51
```

É isso para este livro, parabéns pela conquista! Quando estiver pronto, siga para o Livro 3, *Objects & Classes*.

[^MathJSisPrime]: *Math.js: isPrime(..)*, https://github.com/josdejong/mathjs/blob/develop/src/function/utils/isPrime.js, 3 de março de 2020.
