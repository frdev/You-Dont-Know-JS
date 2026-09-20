# You Don't Know JS Yet: Scope & Closures - 2ª Edição
# Capítulo 1: Qual é o Escopo?

Quando você já tiver escrito seus primeiros programas, é provável que esteja ficando razoavelmente confortável em criar variáveis e guardar valores nelas. Trabalhar com variáveis é uma das coisas mais fundamentais que fazemos em programação!

Mas você pode não ter considerado muito de perto os mecanismos subjacentes que a engine usa para organizar e gerenciar essas variáveis. Não me refiro a como a memória é alocada no computador, e sim: como o JS sabe quais variáveis são acessíveis por uma determinada instrução, e como ele lida com duas variáveis de mesmo nome?

As respostas para perguntas como essas assumem a forma de regras bem definidas chamadas escopo. Este livro vai cavar todos os aspectos do escopo — como ele funciona, para que serve, pegadinhas a evitar — e depois apontar padrões comuns de escopo que guiam a estrutura dos programas.

Nosso primeiro passo é descobrir como a engine JS processa nosso programa **antes** de ele rodar.

## Sobre Este Livro

Bem-vindo ao livro 2 da série *You Don't Know JS Yet*! Se você já terminou *Get Started* (o primeiro livro), está no lugar certo! Se não, antes de prosseguir, eu te encorajo a *começar por lá*, para ter a melhor base.

Nosso foco será o primeiro dos três pilares da linguagem JS: o sistema de escopo e suas closures de função, bem como o poder do padrão de projeto de módulo.

O JS é tipicamente classificado como uma linguagem de script interpretada, então a maioria presume que programas JS são processados em uma única passagem, de cima para baixo. Mas o JS, de fato, passa por parsing/compilação em uma fase separada **antes de a execução começar**. As decisões do autor do código sobre onde colocar variáveis, funções e blocos uns em relação aos outros são analisadas de acordo com as regras de escopo, durante a fase inicial de parsing/compilação. A estrutura de escopo resultante geralmente não é afetada por condições de tempo de execução.

Funções em JS são, elas próprias, valores de primeira classe; podem ser atribuídas e passadas adiante como números ou strings. Mas, como essas funções guardam e acessam variáveis, elas mantêm seu escopo original, não importa em que ponto do programa acabem sendo executadas. Isso se chama closure.

Módulos são um padrão de organização de código caracterizado por métodos públicos que têm acesso privilegiado (via closure) a variáveis e funções ocultas no escopo interno do módulo.

## Compilada vs. Interpretada

Você pode já ter ouvido falar de *compilação de código*, mas talvez isso pareça uma caixa-preta misteriosa, em que o código-fonte desliza por uma ponta e programas executáveis saltam da outra.

Porém, não é misterioso nem mágico. Compilação de código é um conjunto de passos que processa o texto do seu código e o transforma em uma lista de instruções que o computador pode entender. Tipicamente, todo o código-fonte é transformado de uma vez, e as instruções resultantes são salvas como saída (geralmente em um arquivo), que pode ser executada depois.

Você também pode ter ouvido que código pode ser *interpretado*; então, como isso difere de ser *compilado*?

A interpretação realiza uma tarefa parecida com a compilação, no sentido de transformar seu programa em instruções compreensíveis pela máquina. Mas o modelo de processamento é diferente. Diferente de um programa compilado de uma vez, na interpretação o código-fonte é transformado linha a linha; cada linha ou instrução é executada antes de se passar imediatamente ao processamento da próxima linha do código-fonte.

<figure>
    <img src="../../../scope-closures/images/fig1.png" width="650" alt="Compilação de Código e Interpretação de Código" align="center">
    <figcaption><em>Fig. 1: Código Compilado vs. Interpretado</em></figcaption>
    <br><br>
</figure>

A Figura 1 ilustra compilação vs. interpretação de programas.

Esses dois modelos de processamento são mutuamente exclusivos? Em geral, sim. Porém, a questão é mais sutil, porque a interpretação pode, na verdade, assumir outras formas além de simplesmente operar linha a linha sobre o texto do código-fonte. Engines JS modernas empregam, de fato, numerosas variações tanto de compilação quanto de interpretação no tratamento de programas JS.

Lembre-se de que percorremos esse tópico no Capítulo 1 do livro *Get Started*. Nossa conclusão lá é que o JS é retratado com mais precisão como uma **linguagem compilada**. Para o benefício dos leitores daqui, as seções a seguir vão revisitar e expandir essa afirmação.

## Compilando Código

Mas, primeiro, por que sequer importa se o JS é compilado ou não?

O escopo é determinado primordialmente durante a compilação, então entender como compilação e execução se relacionam é fundamental para dominar o escopo.

Na teoria clássica de compiladores, um programa é processado por um compilador em três estágios básicos:

1. **Tokenização/Lexing:** quebrar uma string de caracteres em pedaços significativos (para a linguagem), chamados tokens. Por exemplo, considere o programa: `var a = 2;`. Esse programa provavelmente seria quebrado nos seguintes tokens: `var`, `a`, `=`, `2` e `;`. Espaços em branco podem ou não ser preservados como token, dependendo de serem significativos ou não.

    (A diferença entre tokenização e lexing é sutil e acadêmica, mas gira em torno de esses tokens serem identificados de forma *sem estado* ou *com estado*. Dito de forma simples, se o tokenizador invocasse regras de parsing com estado para descobrir se `a` deveria ser considerado um token distinto ou apenas parte de outro token, *isso* seria **lexing**.)

2. **Parsing:** pegar um fluxo (array) de tokens e transformá-lo em uma árvore de elementos aninhados, que coletivamente representam a estrutura gramatical do programa. Isso se chama Árvore Sintática Abstrata (AST, de *Abstract Syntax Tree*).

    Por exemplo, a árvore para `var a = 2;` poderia começar com um nó de nível superior chamado `VariableDeclaration`, com um nó filho chamado `Identifier` (cujo valor é `a`) e outro filho chamado `AssignmentExpression`, que, por sua vez, tem um filho chamado `NumericLiteral` (cujo valor é `2`).

3. **Geração de Código:** pegar uma AST e transformá-la em código executável. Essa parte varia enormemente dependendo da linguagem, da plataforma-alvo e de outros fatores.

    A engine JS pega a AST que acabamos de descrever para `var a = 2;` e a transforma em um conjunto de instruções de máquina para de fato *criar* uma variável chamada `a` (incluindo reservar memória etc.) e então armazenar um valor em `a`.

| NOTA: |
| :--- |
| Os detalhes de implementação de uma engine JS (uso de recursos de memória do sistema etc.) são muito mais profundos do que vamos cavar aqui. Vamos manter nosso foco no comportamento observável dos nossos programas e deixar a engine JS gerenciar essas abstrações mais profundas, de nível de sistema. |

A engine JS é muito mais complexa do que *apenas* esses três estágios. No processo de parsing e geração de código, há etapas para otimizar o desempenho da execução (por exemplo, colapsar elementos redundantes). De fato, o código pode até ser recompilado e reotimizado durante o andamento da execução.

Então estou pintando apenas com pinceladas largas aqui. Mas você vai ver em breve por que *estes* detalhes que *estamos* cobrindo, mesmo em alto nível, são relevantes.

Engines JS não têm o luxo de uma abundância de tempo para realizar seu trabalho e suas otimizações, porque a compilação JS não acontece em uma etapa de build antecipada, como em outras linguagens. Ela normalmente precisa acontecer em meros microssegundos (ou menos!), logo antes de o código ser executado. Para garantir o desempenho mais rápido sob essas restrições, engines JS usam todo tipo de truque (como JITs, que compilam de forma preguiçosa e até recompilam a quente); isso está bem além do "escopo" da nossa discussão aqui.

### Obrigatório: Duas Fases

Para colocar da forma mais simples possível, a observação mais importante que podemos fazer sobre o processamento de programas JS é que ele ocorre em (pelo menos) duas fases: primeiro parsing/compilação, depois execução.

A separação de uma fase de parsing/compilação da fase de execução subsequente é fato observável, não teoria ou opinião. Embora a especificação do JS não exija explicitamente "compilação", ela exige comportamentos que são essencialmente práticos apenas com uma abordagem de compilar-e-depois-executar.

Há três características de programa que você pode observar para provar isso a si mesmo: erros de sintaxe, erros precoces e hoisting.

#### Erros de Sintaxe Desde o Início

Considere este programa:

```js
var greeting = "Hello";

console.log(greeting);

greeting = ."Hi";
// SyntaxError: unexpected token .
```

Este programa não produz saída alguma (`"Hello"` não é impresso), e em vez disso lança um `SyntaxError` sobre o token `.` inesperado, logo antes da string `"Hi"`. Como o erro de sintaxe acontece depois da instrução bem formada `console.log(..)`, se o JS estivesse executando de cima para baixo, linha a linha, seria de esperar que a mensagem `"Hello"` fosse impressa antes de o erro de sintaxe ser lançado. Isso não acontece.

De fato, a única forma de a engine JS saber do erro de sintaxe na terceira linha, antes de executar a primeira e a segunda, é a engine JS primeiro fazer o parsing do programa inteiro antes de qualquer parte dele ser executada.

#### Erros Precoces

Em seguida, considere:

```js
console.log("Howdy");

saySomething("Hello","Hi");
// Uncaught SyntaxError: Duplicate parameter name not
// allowed in this context

function saySomething(greeting,greeting) {
    "use strict";
    console.log(greeting);
}
```

A mensagem `"Howdy"` não é impressa, apesar de ser uma instrução bem formada.

Em vez disso, assim como no trecho da seção anterior, o `SyntaxError` aqui é lançado antes de o programa ser executado. Neste caso, é porque o modo estrito (ativado apenas para a função `saySomething(..)` aqui) proíbe, entre muitas outras coisas, que funções tenham nomes de parâmetro duplicados; isso sempre foi permitido em modo não estrito.

O erro lançado não é um erro de sintaxe no sentido de ser uma sequência malformada de tokens (como o `."Hi"` anterior), mas, em modo estrito, ainda assim é exigido pela especificação que ele seja lançado como um "erro precoce", antes de qualquer execução começar.

Mas como a engine JS sabe que o parâmetro `greeting` foi duplicado? Como ela sabe que a função `saySomething(..)` está sequer em modo estrito enquanto processa a lista de parâmetros (o pragma `"use strict"` só aparece depois, no corpo da função)?

De novo, a única explicação razoável é que o código precisa primeiro passar *completamente* pelo parsing antes de qualquer execução ocorrer.

#### Hoisting

Por fim, considere:

```js
function saySomething() {
    var greeting = "Hello";
    {
        greeting = "Howdy";  // o erro vem daqui
        let greeting = "Hi";
        console.log(greeting);
    }
}

saySomething();
// ReferenceError: Cannot access 'greeting' before
// initialization
```

O `ReferenceError` apontado ocorre na linha com a instrução `greeting = "Howdy"`. O que está acontecendo é que a variável `greeting` daquela instrução pertence à declaração da linha seguinte, `let greeting = "Hi"`, e não à instrução anterior `var greeting = "Hello"`.

A única forma de a engine JS saber, na linha em que o erro é lançado, que a *próxima instrução* declararia uma variável com escopo de bloco de mesmo nome (`greeting`) é se a engine JS já tivesse processado esse código em uma passagem anterior e já tivesse montado todos os escopos e suas associações de variáveis. Esse processamento de escopos e declarações só pode ser feito com precisão fazendo o parsing do programa antes da execução.

O `ReferenceError` aqui vem, tecnicamente, de `greeting = "Howdy"` acessar a variável `greeting` **cedo demais**, um conflito chamado de Zona Morta Temporal (TDZ, de *Temporal Dead Zone*). O Capítulo 5 vai cobrir isso com mais detalhes.

| AVISO: |
| :--- |
| Frequentemente se afirma que declarações `let` e `const` não sofrem hoisting, como explicação para o comportamento de TDZ que acabamos de ilustrar. Mas isso não é preciso. Vamos voltar e explicar tanto o hoisting quanto a TDZ de `let`/`const` no Capítulo 5. |

Espero que você esteja agora convencido de que programas JS passam por parsing antes de qualquer execução começar. Mas isso prova que eles são compilados?

Esta é uma pergunta interessante de ponderar. O JS poderia fazer o parsing de um programa e então executá-lo *interpretando* operações representadas na AST **sem** antes compilar o programa? Sim, isso é *possível*. Mas é extremamente improvável, sobretudo porque seria extremamente ineficiente em termos de desempenho.

É difícil imaginar uma engine JS de qualidade de produção tendo todo o trabalho de fazer o parsing de um programa até uma AST, mas não convertendo (ou seja, "compilando") essa AST na representação (binária) mais eficiente para a engine então executar.

Muita gente se esforçou para dividir cabelos com essa terminologia, já que há bastante nuance e interjeições do tipo "bem, na verdade...". Mas, em espírito e na prática, o que a engine faz ao processar programas JS é **muito mais parecido com compilação** do que não.

Classificar o JS como linguagem compilada não diz respeito ao modelo de distribuição de suas representações executáveis binárias (ou de byte-code), e sim a manter uma distinção clara em nossas mentes sobre a fase em que o código JS é processado e analisado; essa fase acontece, de forma observável e indiscutível, *antes* de o código começar a ser executado.

Precisamos de modelos mentais adequados de como a engine JS trata nosso código se quisermos entender JS e escopo de forma eficaz.

## O Falar do Compilador

Com a consciência do processamento em duas fases de um programa JS (compilar e depois executar), vamos voltar nossa atenção para como a engine JS identifica variáveis e determina os escopos de um programa enquanto ele é compilado.

Primeiro, vamos examinar um programa JS simples, que usaremos para análise ao longo dos próximos capítulos:

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

Além das declarações, todas as ocorrências de variáveis/identificadores em um programa desempenham um de dois "papéis": ou são o *alvo* (*target*) de uma atribuição, ou são a *origem* (*source*) de um valor.

(Quando aprendi teoria de compiladores, durante minha graduação em ciência da computação, nos ensinaram os termos "LHS" (ou seja, *alvo*) e "RHS" (ou seja, *origem*) para esses papéis, respectivamente. Como você pode imaginar pelo "L" e pelo "R", as siglas significam "Left-Hand Side" (lado esquerdo) e "Right-Hand Side" (lado direito), como nos lados esquerdo e direito de um operador de atribuição `=`. Porém, alvos e origens de atribuição nem sempre aparecem literalmente à esquerda ou à direita de um `=`, então provavelmente é mais claro pensar em termos de *alvo* / *origem* do que de *esquerda* / *direita*.)

Como você sabe se uma variável é um *alvo*? Verifique se há um valor sendo atribuído a ela; se houver, é um *alvo*. Se não, então a variável é uma *origem*.

Para que a engine JS lide corretamente com as variáveis de um programa, ela precisa primeiro rotular cada ocorrência de uma variável como *alvo* ou *origem*. Vamos nos aprofundar agora em como cada papel é determinado.

### Alvos

O que torna uma variável um *alvo*? Considere:

```js
students = [ // ..
```

Esta instrução é claramente uma operação de atribuição; lembre-se de que a parte `var students` é tratada inteiramente como uma declaração em tempo de compilação e, portanto, é irrelevante durante a execução; nós a deixamos de fora por clareza e foco. O mesmo vale para a instrução `nextStudent = getStudentName(73)`.

Mas há outras três operações de atribuição a *alvo* no código, que talvez sejam menos óbvias. Uma delas:

```js
for (let student of students) {
```

Essa instrução atribui um valor a `student` a cada iteração do laço. Outra referência de *alvo*:

```js
getStudentName(73)
```

Mas como isso é uma atribuição a um *alvo*? Olhe de perto: o argumento `73` é atribuído ao parâmetro `studentID`.

E há uma última referência (sutil) de *alvo* no nosso programa. Você consegue identificá-la?

..

..

..

Você identificou esta?

```js
function getStudentName(studentID) {
```

Uma declaração de `function` é um caso especial de referência de *alvo*. Você pode pensar nisso mais ou menos como `var getStudentName = function(studentID)`, mas isso não é exatamente preciso. Um identificador `getStudentName` é declarado (em tempo de compilação), mas a parte `= function(studentID)` também é tratada na compilação; a associação entre `getStudentName` e a função é montada automaticamente no início do escopo, em vez de esperar que uma instrução de atribuição `=` seja executada.

| NOTA: |
| :--- |
| Essa associação automática entre função e variável é chamada de "function hoisting" e é coberta em detalhes no Capítulo 5. |

### Origens

Então identificamos todas as cinco referências de *alvo* no programa. As demais referências de variáveis devem, portanto, ser referências de *origem* (porque essa é a única outra opção!).

Em `for (let student of students)`, dissemos que `student` é um *alvo*, mas `students` é uma referência de *origem*. Na instrução `if (student.id == studentID)`, tanto `student` quanto `studentID` são referências de *origem*. `student` também é uma referência de *origem* em `return student.name`.

Em `getStudentName(73)`, `getStudentName` é uma referência de *origem* (que esperamos que resolva para um valor de referência de função). Em `console.log(nextStudent)`, `console` é uma referência de *origem*, assim como `nextStudent`.

| NOTA: |
| :--- |
| Caso você esteja se perguntando, `id`, `name` e `log` são todos propriedades, não referências de variáveis. |

Qual é a importância prática de entender *alvos* vs. *origens*? No Capítulo 2, vamos revisitar esse tópico e cobrir como o papel de uma variável impacta sua busca (especificamente, se a busca falhar).

## Trapaceando: Modificações de Escopo em Tempo de Execução

A esta altura deve estar claro que o escopo é determinado enquanto o programa é compilado e, em geral, não deveria ser afetado por condições de tempo de execução. Porém, em modo não estrito, tecnicamente ainda existem duas formas de trapacear essa regra, modificando os escopos de um programa durante a execução.

Nenhuma dessas técnicas *deveria* ser usada — ambas são perigosas e confusas, e você deveria estar usando modo estrito (onde são proibidas) de qualquer forma. Mas é importante conhecê-las, caso você se depare com elas em algum programa.

A função `eval(..)` recebe uma string de código para compilar e executar na hora, durante a execução do programa. Se essa string de código tiver uma declaração `var` ou `function`, essas declarações vão modificar o escopo atual em que o `eval(..)` está executando:

```js
function badIdea() {
    eval("var oops = 'Ugh!';");
    console.log(oops);
}
badIdea();   // Ugh!
```

Se o `eval(..)` não estivesse presente, a variável `oops` em `console.log(oops)` não existiria e lançaria um `ReferenceError`. Mas `eval(..)` modifica o escopo da função `badIdea()` em tempo de execução. Isso é ruim por muitas razões, incluindo o custo de desempenho de modificar o escopo já compilado e otimizado toda vez que `badIdea()` roda.

A segunda trapaça é a palavra-chave `with`, que essencialmente transforma dinamicamente um objeto em um escopo local — suas propriedades são tratadas como identificadores no bloco desse novo escopo:

```js
var badIdea = { oops: "Ugh!" };

with (badIdea) {
    console.log(oops);   // Ugh!
}
```

O escopo global não foi modificado aqui, mas `badIdea` foi transformado em um escopo em tempo de execução, e não em tempo de compilação, e sua propriedade `oops` se torna uma variável nesse escopo. De novo, essa é uma ideia terrível, por razões de desempenho e legibilidade.

Evite a todo custo `eval(..)` (pelo menos `eval(..)` criando declarações) e `with`. De novo, nenhuma dessas trapaças está disponível em modo estrito, então, se você simplesmente usar modo estrito (e deveria!), a tentação desaparece!

## Escopo Léxico

Demonstramos que o escopo do JS é determinado em tempo de compilação; o termo para esse tipo de escopo é "escopo léxico". "Léxico" está associado ao estágio de "lexing" da compilação, discutido antes neste capítulo.

Para reduzir este capítulo a uma conclusão útil, a ideia central do "escopo léxico" é que ele é controlado inteiramente pelo posicionamento de funções, blocos e declarações de variáveis, uns em relação aos outros.

Se você coloca uma declaração de variável dentro de uma função, o compilador trata essa declaração enquanto faz o parsing da função e associa essa declaração ao escopo da função. Se uma variável é declarada com escopo de bloco (`let` / `const`), então ela é associada ao bloco `{ .. }` envolvente mais próximo, em vez de à sua função envolvente (como acontece com `var`).

Além disso, uma referência (papel de *alvo* ou *origem*) a uma variável precisa ser resolvida como vinda de um dos escopos que estão *lexicalmente disponíveis* para ela; caso contrário, diz-se que a variável é "não declarada" (o que geralmente resulta em erro!). Se a variável não é declarada no escopo atual, o próximo escopo externo/envolvente será consultado. Esse processo de subir um nível de aninhamento de escopo continua até que uma declaração de variável correspondente seja encontrada ou até que o escopo global seja alcançado e não haja mais para onde ir.

É importante notar que a compilação, na verdade, não *faz nada* em termos de reservar memória para escopos e variáveis. Nada do programa foi executado ainda.

Em vez disso, a compilação cria um mapa de todos os escopos léxicos, que estabelece o que o programa vai precisar enquanto executa. Você pode pensar nesse plano como código inserido para uso em tempo de execução, que define todos os escopos (ou seja, "ambientes léxicos") e registra todos os identificadores (variáveis) de cada escopo.

Em outras palavras, embora os escopos sejam identificados durante a compilação, eles só são de fato criados em tempo de execução, cada vez que um escopo precisa rodar. No próximo capítulo, vamos esboçar as bases conceituais do escopo léxico.
