# You Don't Know JS Yet: Scope & Closures - 2ª Edição
# Capítulo 7: Usando Closures

Até aqui, focamos nos detalhes do escopo léxico e em como isso afeta a organização e o uso de variáveis nos nossos programas.

Nossa atenção se desloca novamente para um nível de abstração mais amplo, rumo ao tópico historicamente um tanto intimidador da closure. Não se preocupe! Você não precisa de um diploma avançado em ciência da computação para entendê-lo. Nosso objetivo amplo neste livro não é meramente entender escopo, mas usá-lo de forma mais eficaz na estrutura dos nossos programas; a closure é central nesse esforço.

Lembre-se da principal conclusão do Capítulo 6: o princípio da *exposição mínima* (POLE) nos encoraja a usar escopo de bloco (e de função) para limitar a exposição de escopo das variáveis. Isso ajuda a manter o código compreensível e manutenível, e ajuda a evitar muitas armadilhas de escopo (por exemplo, colisão de nomes etc.).

A closure se apoia nessa abordagem: para variáveis que precisamos usar ao longo do tempo, em vez de colocá-las em escopos externos maiores, podemos encapsulá-las (com escopo mais estreito) e ainda assim preservar o acesso de dentro de funções, para um uso mais amplo. Funções *lembram* dessas variáveis de escopo referenciadas via closure.

Já vimos um exemplo desse tipo de closure no capítulo anterior (`factorial(..)`, no Capítulo 6), e você quase certamente já a usou nos seus próprios programas. Se você já escreveu um callback que acessa variáveis fora do seu próprio escopo... adivinha!? Isso é closure.

Closure é uma das características de linguagem mais importantes já inventadas na programação — ela sustenta grandes paradigmas de programação, incluindo Programação Funcional (FP), módulos e até um pouco do design orientado a classes. Ficar confortável com closure é obrigatório para dominar o JS e aproveitar eficazmente muitos padrões de design importantes ao longo do seu código.

Abordar todos os aspectos da closure exige uma montanha intimidadora de discussão e código ao longo deste capítulo. Vá com calma e certifique-se de estar confortável com cada parte antes de passar para a próxima.

## Veja a Closure

Closure é, originalmente, um conceito matemático, vindo do cálculo lambda. Mas eu não vou listar fórmulas matemáticas nem usar um monte de notação e jargão para defini-la.

Em vez disso, vou focar em uma perspectiva prática. Vamos começar definindo closure em termos do que podemos observar de diferente no comportamento dos nossos programas, em comparação com um JS em que a closure não existisse. Porém, mais adiante neste capítulo, vamos virar a closure do avesso para olhá-la de uma *perspectiva alternativa*.

Closure é um comportamento de funções e apenas de funções. Se você não está lidando com uma função, closure não se aplica. Um objeto não pode ter closure, nem uma classe tem closure (embora suas funções/métodos possam ter). Apenas funções têm closure.

Para que a closure seja observada, uma função precisa ser invocada e, especificamente, precisa ser invocada em um ramo da cadeia de escopos diferente daquele em que foi originalmente definida. Uma função executando no mesmo escopo em que foi definida não exibiria nenhum comportamento observavelmente diferente com ou sem a possibilidade de closure; pela perspectiva e definição observacional, isso não é closure.

Vamos olhar um código, anotado com as cores relevantes das bolhas de escopo (veja o Capítulo 2):

```js
// escopo externo/global: VERMELHO(1)

function lookupStudent(studentID) {
    // escopo da função: AZUL(2)

    var students = [
        { id: 14, name: "Kyle" },
        { id: 73, name: "Suzy" },
        { id: 112, name: "Frank" },
        { id: 6, name: "Sarah" }
    ];

    return function greetStudent(greeting){
        // escopo da função: VERDE(3)

        var student = students.find(
            student => student.id == studentID
        );

        return `${ greeting }, ${ student.name }!`;
    };
}

var chosenStudents = [
    lookupStudent(6),
    lookupStudent(112)
];

// acessando o nome da função:
chosenStudents[0].name;
// greetStudent

chosenStudents[0]("Hello");
// Hello, Sarah!

chosenStudents[1]("Howdy");
// Howdy, Frank!
```

A primeira coisa a notar sobre este código é que a função externa `lookupStudent(..)` cria e retorna uma função interna chamada `greetStudent(..)`. `lookupStudent(..)` é chamada duas vezes, produzindo duas instâncias separadas da sua função interna `greetStudent(..)`, ambas salvas no array `chosenStudents`.

Verificamos que é esse o caso checando a propriedade `.name` da função retornada, salva em `chosenStudents[0]`, e ela é, de fato, uma instância da função interna `greetStudent(..)`.

Depois que cada chamada a `lookupStudent(..)` termina, pareceria que todas as suas variáveis internas seriam descartadas e coletadas pelo GC (garbage collector). A função interna é a única coisa que parece ser retornada e preservada. Mas é aqui que o comportamento difere de formas que podemos começar a observar.

Embora `greetStudent(..)` receba um único argumento como o parâmetro chamado `greeting`, ela também faz referência tanto a `students` quanto a `studentID`, identificadores que vêm do escopo envolvente de `lookupStudent(..)`. Cada uma dessas referências, da função interna a uma variável em um escopo externo, é chamada de *closure*. Em termos acadêmicos, cada instância de `greetStudent(..)` *faz closure sobre* as variáveis externas `students` e `studentID`.

Então, o que essas closures fazem aqui, num sentido concreto e observável?

A closure permite que `greetStudent(..)` continue acessando essas variáveis externas mesmo depois que o escopo externo terminou (quando cada chamada a `lookupStudent(..)` se completa). Em vez de as instâncias de `students` e `studentID` serem coletadas pelo GC, elas permanecem na memória. Mais tarde, quando qualquer uma das instâncias da função `greetStudent(..)` é invocada, essas variáveis ainda estão lá, guardando seus valores atuais.

Se as funções JS não tivessem closure, a conclusão de cada chamada a `lookupStudent(..)` derrubaria imediatamente seu escopo e coletaria as variáveis `students` e `studentID`. Quando depois chamássemos uma das funções `greetStudent(..)`, o que aconteceria?

Se `greetStudent(..)` tentasse acessar o que ela achava ser uma bolinha AZUL(2), mas essa bolinha não existisse mais, a suposição razoável é que deveríamos receber um `ReferenceError`, certo?

Mas não recebemos erro. O fato de a execução de `chosenStudents[0]("Hello")` funcionar e nos retornar a mensagem "Hello, Sarah!" significa que ela ainda conseguiu acessar as variáveis `students` e `studentID`. Esta é uma observação direta de closure!

### Closure Apontada

Na verdade, passamos por cima de um pequeno detalhe na discussão anterior, que imagino que muitos leitores não tenham percebido!

Por causa de quão concisa é a sintaxe das arrow functions `=>`, é fácil esquecer que elas ainda criam um escopo (como afirmado em "Arrow Functions", no Capítulo 3). A arrow function `student => student.id == studentID` está criando outra bolha de escopo dentro do escopo da função `greetStudent(..)`.

Construindo sobre a metáfora de baldes e bolhas coloridas do Capítulo 2, se estivéssemos criando um diagrama colorido para este código, haveria um quarto escopo neste nível de aninhamento mais interno, então precisaríamos de uma quarta cor; talvez escolhêssemos LARANJA(4) para esse escopo:

```js
var student = students.find(
    student =>
        // escopo da função: LARANJA(4)
        student.id == studentID
);
```

A referência AZUL(2) `studentID` está, na verdade, dentro do escopo LARANJA(4), e não do escopo VERDE(3) de `greetStudent(..)`; além disso, o parâmetro `student` da arrow function é LARANJA(4), sombreando o `student` VERDE(3).

A consequência aqui é que essa arrow function, passada como callback para o método `find(..)` do array, é que precisa manter a closure sobre `studentID`, em vez de `greetStudent(..)` manter essa closure. Isso não é grande coisa, já que tudo continua funcionando como esperado. Só é importante não passar batido pelo fato de que até arrow functions minúsculas podem entrar na festa da closure.

### Somando Closures

Vamos examinar um dos exemplos canônicos frequentemente citados para closure:

```js
function adder(num1) {
    return function addTo(num2){
        return num1 + num2;
    };
}

var add10To = adder(10);
var add42To = adder(42);

add10To(15);    // 25
add42To(9);     // 51
```

Cada instância da função interna `addTo(..)` está fazendo closure sobre sua própria variável `num1` (com os valores `10` e `42`, respectivamente), então esses `num1` não desaparecem só porque `adder(..)` terminou. Quando depois invocamos uma dessas instâncias internas de `addTo(..)`, como na chamada `add10To(15)`, sua variável `num1` sob closure ainda existe e ainda guarda o valor original `10`. A operação, portanto, consegue realizar `10 + 15` e retornar a resposta `25`.

Um detalhe importante pode ter sido fácil demais de passar batido no parágrafo anterior, então vamos reforçá-lo: closure está associada a uma instância de uma função, e não à sua única definição léxica. No trecho anterior, há apenas uma função interna `addTo(..)` definida dentro de `adder(..)`, então poderia parecer que isso implicaria uma única closure.

Mas, na verdade, toda vez que a função externa `adder(..)` roda, uma *nova* instância da função interna `addTo(..)` é criada e, para cada nova instância, uma nova closure. Então cada instância da função interna (rotulada como `add10To(..)` e `add42To(..)` no nosso programa) tem sua própria closure sobre sua própria instância do ambiente de escopo daquela execução de `adder(..)`.

Embora a closure se baseie no escopo léxico, que é tratado em tempo de compilação, a closure é observada como uma característica de tempo de execução de instâncias de função.

### Vínculo Vivo, Não um Instantâneo

Em ambos os exemplos das seções anteriores, **lemos o valor de uma variável** mantida em uma closure. Isso dá a sensação de que a closure poderia ser um instantâneo de um valor em determinado momento. De fato, esse é um equívoco comum.

A closure é, na verdade, um vínculo vivo, preservando o acesso à variável completa em si. Não estamos limitados a meramente ler um valor; a variável sob closure também pode ser atualizada (reatribuída)! Ao fazer closure sobre uma variável em uma função, podemos continuar usando essa variável (leitura e escrita) enquanto aquela referência de função existir no programa, e de qualquer lugar de onde queiramos invocar essa função. É por isso que closure é uma técnica tão poderosa, usada amplamente em tantas áreas da programação!

A Figura 4 mostra as instâncias de função e os vínculos de escopo:

<figure>
    <img src="../../../scope-closures/images/fig4.png" width="400" alt="Instâncias de função vinculadas a escopos via closure" align="center">
    <figcaption><em>Fig. 4: Visualizando Closures</em></figcaption>
    <br><br>
</figure>

Como mostrado na Figura 4, cada chamada a `adder(..)` cria um novo escopo AZUL(2), contendo uma variável `num1`, bem como uma nova instância da função `addTo(..)` como escopo VERDE(3). Note que as instâncias de função (`addTo10(..)` e `addTo42(..)`) estão presentes no escopo VERMELHO(1) e são invocadas a partir dele.

Agora vamos examinar um exemplo em que a variável sob closure é atualizada:

```js
function makeCounter() {
    var count = 0;

    return function getCurrent() {
        count = count + 1;
        return count;
    };
}

var hits = makeCounter();

// depois

hits();     // 1

// depois

hits();     // 2
hits();     // 3
```

A variável `count` está sob closure da função interna `getCurrent()`, que a mantém por perto em vez de ela ser submetida ao GC. As chamadas à função `hits()` acessam *e* atualizam essa variável, retornando uma contagem incrementada a cada vez.

Embora o escopo envolvente de uma closure venha tipicamente de uma função, isso não é, na verdade, obrigatório; só é preciso haver uma função interna presente dentro de um escopo externo:

```js
var hits;
{   // um escopo externo (mas não uma função)
    let count = 0;
    hits = function getCurrent(){
        count = count + 1;
        return count;
    };
}
hits();     // 1
hits();     // 2
hits();     // 3
```

| NOTA: |
| :--- |
| Eu deliberadamente defini `getCurrent()` como uma expressão de `function` em vez de uma declaração de `function`. Isso não tem a ver com closure, mas com as esquisitices perigosas do FiB (Capítulo 6). |

Como é muito comum confundir closure como orientada a valor em vez de orientada a variável, desenvolvedores às vezes tropeçam ao tentar usar closure para preservar um instantâneo de um valor de algum momento no tempo. Considere:

```js
var studentName = "Frank";

var greeting = function hello() {
    // estamos fazendo closure sobre `studentName`,
    // não sobre "Frank"
    console.log(
        `Hello, ${ studentName }!`
    );
}

// depois

studentName = "Suzy";

// depois

greeting();
// Hello, Suzy!
```

Ao definir `greeting()` (ou seja, `hello()`) quando `studentName` guarda o valor `"Frank"` (antes da reatribuição para `"Suzy"`), a suposição equivocada frequentemente é que a closure vai capturar `"Frank"`. Mas `greeting()` faz closure sobre a variável `studentName`, não sobre seu valor. Sempre que `greeting()` é invocada, o valor atual da variável (`"Suzy"`, neste caso) é refletido.

A ilustração clássica desse erro é definir funções dentro de um laço:

```js
var keeps = [];

for (var i = 0; i < 3; i++) {
    keeps[i] = function keepI(){
        // closure sobre `i`
        return i;
    };
}

keeps[0]();   // 3 -- POR QUÊ!?
keeps[1]();   // 3
keeps[2]();   // 3
```

| NOTA: |
| :--- |
| Esse tipo de ilustração de closure tipicamente usa um `setTimeout(..)` ou algum outro callback, como um manipulador de evento, dentro do laço. Simplifiquei o exemplo guardando referências de função em um array, para não precisarmos considerar timing assíncrono na nossa análise. O princípio da closure é o mesmo, de qualquer forma. |

Você pode ter esperado que a invocação `keeps[0]()` retornasse `0`, já que essa função foi criada durante a primeira iteração do laço, quando `i` era `0`. Mas, de novo, essa suposição vem de pensar na closure como orientada a valor, e não a variável.

Algo na estrutura de um laço `for` pode nos enganar e nos fazer pensar que cada iteração ganha sua própria nova variável `i`; na verdade, este programa só tem um `i`, já que ele foi declarado com `var`.

Cada função salva retorna `3`, porque, ao fim do laço, a única variável `i` do programa recebeu `3`. Cada uma das três funções do array `keeps` tem, sim, closures individuais, mas todas fazem closure sobre aquela mesma variável `i` compartilhada.

Claro, uma única variável só pode guardar um valor em qualquer dado momento. Então, se você quer preservar múltiplos valores, precisa de uma variável diferente para cada um.

Como poderíamos fazer isso no trecho com o laço? Vamos criar uma nova variável para cada iteração:

```js
var keeps = [];

for (var i = 0; i < 3; i++) {
    // novo `j` criado a cada iteração, que recebe
    // uma cópia do valor de `i` neste momento
    let j = i;

    // o `i` aqui não está sob closure, então
    // não há problema em usar imediatamente seu
    // valor atual em cada iteração do laço
    keeps[i] = function keepEachJ(){
        // closure sobre `j`, não sobre `i`!
        return j;
    };
}
keeps[0]();   // 0
keeps[1]();   // 1
keeps[2]();   // 2
```

Cada função agora faz closure sobre uma variável separada (nova) de cada iteração, ainda que todas se chamem `j`. E cada `j` recebe uma cópia do valor de `i` naquele ponto da iteração do laço; esse `j` nunca é reatribuído. Então todas as três funções agora retornam seus valores esperados: `0`, `1` e `2`!

De novo, lembre-se: mesmo que estivéssemos usando assincronismo neste programa, como passando cada função interna `keepEachJ()` para `setTimeout(..)` ou para alguma inscrição de manipulador de evento, o mesmo tipo de comportamento de closure ainda seria observado.

Lembre-se da seção "Laços", no Capítulo 5, que ilustra como uma declaração `let` em um laço `for` cria, na verdade, não apenas uma variável para o laço, mas sim uma nova variável para *cada iteração* do laço. Esse truque/esquisitice é exatamente o que precisamos para as closures dos nossos laços:

```js
var keeps = [];

for (let i = 0; i < 3; i++) {
    // o `let i` nos dá um novo `i` para
    // cada iteração, automaticamente!
    keeps[i] = function keepEachI(){
        return i;
    };
}
keeps[0]();   // 0
keeps[1]();   // 1
keeps[2]();   // 2
```

Como estamos usando `let`, três `i` são criados, um para cada iteração do laço, então cada uma das três closures *simplesmente funciona* como esperado.

### Closures Comuns: Ajax e Eventos

A closure é mais comumente encontrada com callbacks:

```js
function lookupStudentRecord(studentID) {
    ajax(
        `https://some.api/student/${ studentID }`,
        function onRecord(record) {
            console.log(
                `${ record.name } (${ studentID })`
            );
        }
    );
}

lookupStudentRecord(114);
// Frank (114)
```

O callback `onRecord(..)` vai ser invocado em algum ponto no futuro, depois que a resposta da chamada Ajax voltar. Essa invocação vai acontecer a partir das entranhas do utilitário `ajax(..)`, de onde quer que ele venha. Além disso, quando isso acontecer, a chamada a `lookupStudentRecord(..)` já terá terminado há muito tempo.

Por que, então, `studentID` ainda está por perto e acessível ao callback? Closure.

Manipuladores de eventos são outro uso comum de closure:

```js
function listenForClicks(btn,label) {
    btn.addEventListener("click",function onClick(){
        console.log(
            `The ${ label } button was clicked!`
        );
    });
}

var submitBtn = document.getElementById("submit-btn");

listenForClicks(submitBtn,"Checkout");
```

O parâmetro `label` está sob closure do callback manipulador de evento `onClick(..)`. Quando o botão é clicado, `label` ainda existe para ser usado. Isso é closure.

### E Se Eu Não Consigo Vê-la?

Você provavelmente já ouviu este ditado comum:

> Se uma árvore cai na floresta e não há ninguém por perto para ouvir, ela faz som?

É uma bobagem de ginástica filosófica. Claro que, de uma perspectiva científica, ondas sonoras são criadas. Mas o ponto de verdade é: *importa* se o som acontece?

Lembre-se de que a ênfase na nossa definição de closure é a observabilidade. Se uma closure existe (num sentido técnico, de implementação ou acadêmico), mas não pode ser observada nos nossos programas, *isso importa?* Não.

Para reforçar esse ponto, vamos olhar alguns exemplos que *não* se baseiam observavelmente em closure.

Por exemplo, invocar uma função que faz uso de busca por escopo léxico:

```js
function say(myName) {
    var greeting = "Hello";
    output();

    function output() {
        console.log(
            `${ greeting }, ${ myName }!`
        );
    }
}

say("Kyle");
// Hello, Kyle!
```

A função interna `output()` acessa as variáveis `greeting` e `myName` do seu escopo envolvente. Mas a invocação de `output()` acontece nesse mesmo escopo, onde, claro, `greeting` e `myName` ainda estão disponíveis; isso é apenas escopo léxico, não closure.

Qualquer linguagem de escopo léxico cujas funções não suportassem closure ainda se comportaria exatamente assim.

De fato, variáveis de escopo global essencialmente não podem estar (observavelmente) sob closure, porque elas são sempre acessíveis de qualquer lugar. Nenhuma função jamais pode ser invocada em alguma parte da cadeia de escopos que não seja descendente do escopo global.

Considere:

```js
var students = [
    { id: 14, name: "Kyle" },
    { id: 73, name: "Suzy" },
    { id: 112, name: "Frank" },
    { id: 6, name: "Sarah" }
];

function getFirstStudent() {
    return function firstStudent(){
        return students[0].name;
    };
}

var student = getFirstStudent();

student();
// Kyle
```

A função interna `firstStudent()` referencia, sim, `students`, que é uma variável fora do seu próprio escopo. Mas, como `students` por acaso vem do escopo global, não importa onde essa função seja invocada no programa, sua capacidade de acessar `students` não é nada mais especial do que o escopo léxico normal.

Todas as invocações de função podem acessar variáveis globais, independentemente de a linguagem suportar closure ou não. Variáveis globais não precisam estar sob closure.

Variáveis que estão meramente presentes, mas nunca são acessadas, não resultam em closure:

```js
function lookupStudent(studentID) {
    return function nobody(){
        var msg = "Nobody's here yet.";
        console.log(msg);
    };
}

var student = lookupStudent(112);

student();
// Nobody's here yet.
```

A função interna `nobody()` não faz closure sobre nenhuma variável externa — ela só usa sua própria variável `msg`. Mesmo que `studentID` esteja presente no escopo envolvente, `studentID` não é referenciada por `nobody()`. A engine JS não precisa manter `studentID` por perto depois que `lookupStudent(..)` terminou de rodar, então o GC quer limpar essa memória!

Quer as funções JS suportem closure ou não, este programa se comportaria da mesma forma. Portanto, nenhuma closure observada aqui.

Se não há invocação de função, a closure não pode ser observada:

```js
function greetStudent(studentName) {
    return function greeting(){
        console.log(
            `Hello, ${ studentName }!`
        );
    };
}

greetStudent("Kyle");

// nada mais acontece
```

Este é capcioso, porque a função externa definitivamente é invocada. Mas a função interna é a que *poderia* ter tido closure e, ainda assim, nunca é invocada; a função retornada aqui é simplesmente descartada. Então, mesmo que tecnicamente a engine JS tenha criado closure por um breve momento, ela não foi observada de nenhuma forma significativa neste programa.

Uma árvore pode ter caído... mas não a ouvimos, então não nos importamos.

### Definição Observável

Agora estamos prontos para definir closure:

> A closure é observada quando uma função usa variável(is) de escopo(s) externo(s) mesmo enquanto roda em um escopo em que essa(s) variável(is) não seria(m) acessível(is).

As partes-chave dessa definição são:

* Precisa haver uma função envolvida

* Precisa referenciar ao menos uma variável de um escopo externo

* Precisa ser invocada em um ramo da cadeia de escopos diferente daquele da(s) variável(is)

Essa definição orientada à observação significa que não devemos descartar a closure como alguma trivialidade acadêmica e indireta. Em vez disso, devemos procurar e planejar os efeitos diretos e concretos que a closure tem sobre o comportamento do nosso programa.

## O Ciclo de Vida da Closure e a Coleta de Lixo (GC)

Como a closure está inerentemente atrelada a uma instância de função, sua closure sobre uma variável dura enquanto ainda houver uma referência àquela função.

Se dez funções fazem closure sobre a mesma variável e, com o tempo, nove dessas referências de função são descartadas, a única referência de função restante ainda preserva aquela variável. Uma vez que essa última referência de função é descartada, a última closure sobre aquela variável some, e a própria variável é coletada pelo GC.

Isso tem um impacto importante na construção de programas eficientes e performáticos. A closure pode, inesperadamente, impedir o GC de uma variável com a qual você já acabou, o que leva a um uso de memória desgovernado ao longo do tempo. É por isso que é importante descartar referências de função (e, portanto, suas closures) quando elas não são mais necessárias.

Considere:

```js
function manageBtnClickEvents(btn) {
    var clickHandlers = [];

    return function listener(cb){
        if (cb) {
            let clickHandler =
                function onClick(evt){
                    console.log("clicked!");
                    cb(evt);
                };
            clickHandlers.push(clickHandler);
            btn.addEventListener(
                "click",
                clickHandler
            );
        }
        else {
            // não passar callback cancela a inscrição
            // de todos os manipuladores de clique
            for (let handler of clickHandlers) {
                btn.removeEventListener(
                    "click",
                    handler
                );
            }

            clickHandlers = [];
        }
    };
}

// var mySubmitBtn = ..
var onSubmit = manageBtnClickEvents(mySubmitBtn);

onSubmit(function checkout(evt){
    // trata o checkout
});

onSubmit(function trackAction(evt){
    // registra a ação na analytics
});

// depois, cancela a inscrição de todos os manipuladores:
onSubmit();
```

Neste programa, a função interna `onClick(..)` mantém uma closure sobre o `cb` recebido (o callback de evento fornecido). Isso significa que as referências às expressões de função `checkout()` e `trackAction()` são mantidas via closure (e não podem ser coletadas pelo GC) enquanto esses manipuladores de evento estiverem inscritos.

Quando chamamos `onSubmit()` sem entrada, na última linha, todos os manipuladores de evento têm a inscrição cancelada, e o array `clickHandlers` é esvaziado. Uma vez que todas as referências às funções manipuladoras de clique são descartadas, as closures das referências `cb` a `checkout()` e `trackAction()` são descartadas.

Ao considerar a saúde e a eficiência gerais do programa, cancelar a inscrição de um manipulador de evento quando ele não é mais necessário pode ser até mais importante do que a inscrição inicial!

### Por Variável ou Por Escopo?

Outra pergunta que precisamos encarar: devemos pensar na closure como aplicada apenas à(s) variável(is) externa(s) referenciada(s), ou a closure preserva toda a cadeia de escopos com todas as suas variáveis?

Em outras palavras, no trecho anterior de inscrição de eventos, a função interna `onClick(..)` faz closure apenas sobre `cb`, ou também sobre `clickHandler`, `clickHandlers` e `btn`?

Conceitualmente, a closure é **por variável**, e não *por escopo*. Callbacks de Ajax, manipuladores de evento e todas as outras formas de closures de função são tipicamente presumidos como fazendo closure apenas sobre aquilo que referenciam explicitamente.

Mas a realidade é mais complicada do que isso.

Outro programa a considerar:

```js
function manageStudentGrades(studentRecords) {
    var grades = studentRecords.map(getGrade);

    return addGrade;

    // ************************

    function getGrade(record){
        return record.grade;
    }

    function sortAndTrimGradesList() {
        // ordena por notas, decrescente
        grades.sort(function desc(g1,g2){
            return g2 - g1;
        });

        // mantém apenas as 10 melhores notas
        grades = grades.slice(0,10);
    }

    function addGrade(newGrade) {
        grades.push(newGrade);
        sortAndTrimGradesList();
        return grades;
    }
}

var addNextGrade = manageStudentGrades([
    { id: 14, name: "Kyle", grade: 86 },
    { id: 73, name: "Suzy", grade: 87 },
    { id: 112, name: "Frank", grade: 75 },
    // ..muitos mais registros..
    { id: 6, name: "Sarah", grade: 91 }
]);

// depois

addNextGrade(81);
addNextGrade(68);
// [ .., .., ... ]
```

A função externa `manageStudentGrades(..)` recebe uma lista de registros de alunos e retorna uma referência à função `addGrade(..)`, que rotulamos externamente como `addNextGrade(..)`. Cada vez que chamamos `addNextGrade(..)` com uma nova nota, recebemos de volta a lista atual das 10 melhores notas, ordenadas numericamente de forma decrescente (veja `sortAndTrimGradesList()`).

Desde o fim da chamada original a `manageStudentGrades(..)` e entre as múltiplas chamadas a `addNextGrade(..)`, a variável `grades` é preservada dentro de `addGrade(..)` via closure; é assim que a lista corrente das melhores notas é mantida. Lembre-se: é uma closure sobre a própria variável `grades`, não sobre o array que ela guarda.

Essa não é a única closure envolvida, porém. Você consegue identificar outras variáveis sob closure?

Você percebeu que `addGrade(..)` referencia `sortAndTrimGradesList`? Isso significa que ela também faz closure sobre esse identificador, que por acaso guarda uma referência à função `sortAndTrimGradesList()`. Essa segunda função interna precisa continuar por perto para que `addGrade(..)` possa continuar chamando-a, o que também significa que quaisquer variáveis sobre as quais *ela* faz closure continuam por perto — embora, neste caso, nada extra esteja sob closure ali.

O que mais está sob closure?

Considere a variável `getGrade` (e sua função); ela está sob closure? Ela é referenciada no escopo externo de `manageStudentGrades(..)`, na chamada `.map(getGrade)`. Mas não é referenciada em `addGrade(..)` nem em `sortAndTrimGradesList()`.

E quanto à lista (potencialmente) grande de registros de alunos que passamos como `studentRecords`? Essa variável está sob closure? Se estiver, o array de registros de alunos nunca é coletado pelo GC, o que faz com que este programa segure uma quantidade de memória maior do que poderíamos supor. Mas, se olharmos de perto de novo, nenhuma das funções internas referencia `studentRecords`.

De acordo com a definição de closure *por variável*, como `getGrade` e `studentRecords` *não* são referenciadas pelas funções internas, elas não estão sob closure. Deveriam estar livremente disponíveis para GC logo depois que a chamada a `manageStudentGrades(..)` se completa.

De fato, tente depurar este código em uma engine JS recente, como a v8 do Chrome, colocando um breakpoint dentro da função `addGrade(..)`. Você pode notar que o inspetor **não** lista a variável `studentRecords`. Isso é prova, ao menos do ponto de vista da depuração, de que a engine não mantém `studentRecords` via closure. Ufa!

Mas quão confiável é essa observação como prova? Considere este programa (bastante forçado!):

```js
function storeStudentInfo(id,name,grade) {
    return function getInfo(whichValue){
        // aviso:
        //   usar `eval(..)` é uma má ideia!
        var val = eval(whichValue);
        return val;
    };
}

var info = storeStudentInfo(73,"Suzy",87);

info("name");
// Suzy

info("grade");
// 87
```

Note que a função interna `getInfo(..)` não faz closure explicitamente sobre nenhuma das variáveis `id`, `name` ou `grade`. E, ainda assim, chamadas a `info(..)` parecem conseguir acessar as variáveis, ainda que por meio da trapaça de escopo léxico do `eval(..)` (veja o Capítulo 1).

Então todas as variáveis foram definitivamente preservadas via closure, apesar de não serem explicitamente referenciadas pela função interna. Isso desmente a afirmação *por variável* em favor de *por escopo*? Depende.

Muitas engines JS modernas aplicam, sim, uma *otimização* que remove de um escopo de closure quaisquer variáveis que não sejam explicitamente referenciadas. Porém, como vemos com `eval(..)`, há situações em que tal otimização não pode ser aplicada, e o escopo de closure continua contendo todas as suas variáveis originais. Em outras palavras, a closure precisa ser *por escopo*, em termos de implementação, e então uma otimização opcional apara o escopo para conter apenas o que está sob closure (um resultado parecido com o da closure *por variável*).

Mesmo há poucos anos, muitas engines JS não aplicavam essa otimização; é possível que seus sites ainda rodem nesses navegadores, especialmente em dispositivos mais antigos ou de baixo custo. Isso significa que é possível que closures de vida longa, como manipuladores de evento, estejam segurando memória por muito mais tempo do que teríamos suposto.

E o fato de ser, antes de tudo, uma otimização opcional, e não um requisito da especificação, significa que não devemos simplesmente presumir casualmente sua aplicabilidade.

Nos casos em que uma variável guarda um valor grande (como um objeto ou array) e essa variável está presente em um escopo de closure, se você não precisa mais desse valor e não quer que essa memória fique retida, é mais seguro (em uso de memória) descartar o valor manualmente do que depender da otimização de closure/GC.

Vamos aplicar uma *correção* ao exemplo anterior de `manageStudentGrades(..)` para garantir que o array potencialmente grande guardado em `studentRecords` não fique preso desnecessariamente em um escopo de closure:

```js
function manageStudentGrades(studentRecords) {
    var grades = studentRecords.map(getGrade);

    // anula `studentRecords` para evitar retenção
    // indesejada de memória na closure
    studentRecords = null;

    return addGrade;
    // ..
}
```

Não estamos removendo `studentRecords` do escopo de closure; isso não podemos controlar. Estamos garantindo que, mesmo que `studentRecords` permaneça no escopo de closure, essa variável não esteja mais referenciando o array de dados potencialmente grande; o array pode ser coletado pelo GC.

De novo, em muitos casos o JS pode automaticamente otimizar o programa com o mesmo efeito. Mas ainda é um bom hábito ter cuidado e garantir explicitamente que não mantemos nenhuma quantidade significativa de memória do dispositivo presa por mais tempo do que o necessário.

Aliás, tecnicamente também não precisamos mais da função `getGrade()` depois que a chamada `.map(getGrade)` se completa. Se o profiling da nossa aplicação mostrasse que esta é uma área crítica de uso excessivo de memória, poderíamos talvez espremer um pouquinho mais de memória liberando essa referência, para que seu valor também não fique preso. Isso provavelmente é desnecessário neste exemplo de brinquedo, mas é uma técnica geral a ter em mente se você estiver otimizando a pegada de memória da sua aplicação.

A lição: é importante saber onde closures aparecem nos nossos programas e quais variáveis estão incluídas. Devemos gerenciar essas closures com cuidado, para segurar apenas o mínimo necessário e não desperdiçar memória.

## Uma Perspectiva Alternativa

Revisando nossa definição de trabalho para closure, a afirmação é que funções são "valores de primeira classe" que podem ser passados pelo programa, como qualquer outro valor. A closure é a associação-vínculo que conecta essa função ao escopo/variáveis fora dela, não importa para onde essa função vá.

Vamos relembrar um exemplo de código de antes neste capítulo, novamente com as cores relevantes das bolhas de escopo anotadas:

```js
// escopo externo/global: VERMELHO(1)

function adder(num1) {
    // escopo da função: AZUL(2)

    return function addTo(num2){
        // escopo da função: VERDE(3)

        return num1 + num2;
    };
}

var add10To = adder(10);
var add42To = adder(42);

add10To(15);    // 25
add42To(9);     // 51
```

Nossa perspectiva atual sugere que, onde quer que uma função seja passada e invocada, a closure preserva um vínculo oculto de volta ao escopo original, para facilitar o acesso às variáveis sob closure. A Figura 4, repetida aqui por conveniência, ilustra essa noção:

<figure>
    <img src="../../../scope-closures/images/fig4.png" width="400" alt="Instâncias de função vinculadas a escopos via closure" align="center">
    <figcaption><em>Fig. 4 (repetida): Visualizando Closures</em></figcaption>
    <br><br>
</figure>

Mas há outra forma de pensar sobre a closure e, mais precisamente, sobre a natureza das funções sendo *passadas adiante*, que pode ajudar a aprofundar os modelos mentais.

Esse modelo alternativo desenfatiza "funções como valores de primeira classe" e, em vez disso, abraça o fato de que funções (como todos os valores não primitivos) são mantidas por referência em JS, e atribuídas/passadas por cópia de referência — veja o Apêndice A do livro *Get Started* para mais informações.

Em vez de pensar na instância da função interna `addTo(..)` se movendo para o escopo externo VERMELHO(1) via `return` e atribuição, podemos imaginar que as instâncias de função, na verdade, simplesmente ficam no lugar, no seu próprio ambiente de escopo, é claro com sua cadeia de escopos intacta.

O que é *enviado* ao escopo VERMELHO(1) é **apenas uma referência** à instância de função que está em seu lugar, e não a instância da função em si. A Figura 5 mostra as instâncias das funções internas permanecendo no lugar, apontadas pelas referências VERMELHAS(1) `addTo10` e `addTo42`, respectivamente:

<figure>
    <img src="../../../scope-closures/images/fig5.png" width="400" alt="Instâncias de função dentro de escopos via closure, apontadas por referências" align="center">
    <figcaption><em>Fig. 5: Visualizando Closures (Alternativa)</em></figcaption>
    <br><br>
</figure>

Como mostrado na Figura 5, cada chamada a `adder(..)` ainda cria um novo escopo AZUL(2) contendo uma variável `num1`, bem como uma instância do escopo VERDE(3) de `addTo(..)`. Mas o que é diferente da Figura 4 é que agora essas instâncias VERDES(3) permanecem no lugar, naturalmente aninhadas dentro de suas instâncias de escopo AZUL(2). As referências `addTo10` e `addTo42` é que são movidas para o escopo externo VERMELHO(1), e não as instâncias de função em si.

Quando `addTo10(15)` é chamada, a instância da função `addTo(..)` (ainda no lugar, no seu ambiente de escopo AZUL(2) original) é invocada. Como a instância da função em si nunca se moveu, claro que ela ainda tem acesso natural à sua cadeia de escopos. O mesmo com a chamada `addTo42(9)` — nada de especial aqui além do escopo léxico.

Então, o que *é* a closure, se não a *mágica* que permite a uma função manter um vínculo com sua cadeia de escopos original mesmo enquanto essa função se move por outros escopos? Neste modelo alternativo, funções ficam no lugar e continuam acessando sua cadeia de escopos original como sempre puderam.

A closure, em vez disso, descreve a *mágica* de **manter viva uma instância de função**, junto com todo o seu ambiente e cadeia de escopos, enquanto houver ao menos uma referência a essa instância de função circulando em qualquer outra parte do programa.

Essa definição de closure é menos observacional e soa um pouco menos familiar em comparação com a perspectiva acadêmica tradicional. Mas ainda assim é útil, porque o benefício é que simplificamos a explicação da closure a uma combinação direta de referências e instâncias de função que ficam no lugar.

O modelo anterior (Figura 4) não está *errado* ao descrever closure em JS. Ele é apenas mais inspirado conceitualmente, uma perspectiva acadêmica sobre closure. Em contraste, o modelo alternativo (Figura 5) poderia ser descrito como um pouco mais focado em implementação, em como o JS realmente funciona.

Ambas as perspectivas/modelos são úteis para entender closure, mas o leitor pode achar um pouco mais fácil de segurar um do que o outro. Qualquer que você escolha, os resultados observáveis no nosso programa são os mesmos.

| NOTA: |
| :--- |
| Este modelo alternativo de closure afeta, sim, se classificamos callbacks síncronos como exemplos de closure ou não. Mais sobre essa nuance no Apêndice A. |

## Por que Closure?

Agora que temos uma noção bem arredondada do que é closure e de como ela funciona, vamos explorar algumas formas pelas quais ela pode melhorar a estrutura e a organização do código de um programa de exemplo.

Imagine que você tem um botão em uma página que, ao ser clicado, deve recuperar e enviar alguns dados por uma requisição Ajax. Sem usar closure:

```js
var APIendpoints = {
    studentIDs:
        "https://some.api/register-students",
    // ..
};

var data = {
    studentIDs: [ 14, 73, 112, 6 ],
    // ..
};

function makeRequest(evt) {
    var btn = evt.target;
    var recordKind = btn.dataset.kind;
    ajax(
        APIendpoints[recordKind],
        data[recordKind]
    );
}

// <button data-kind="studentIDs">
//    Register Students
// </button>
btn.addEventListener("click",makeRequest);
```

O utilitário `makeRequest(..)` só recebe um objeto `evt` de um evento de clique. A partir daí, ele precisa recuperar o atributo `data-kind` do elemento de botão alvo e usar esse valor para buscar tanto a URL do endpoint da API quanto os dados que devem ser incluídos na requisição Ajax.

Isso funciona ok, mas é lamentável (ineficiente, mais confuso) que o manipulador de evento tenha que ler um atributo do DOM toda vez que é disparado. Por que um manipulador de evento não poderia *lembrar* esse valor? Vamos tentar usar closure para melhorar o código:

```js
var APIendpoints = {
    studentIDs:
        "https://some.api/register-students",
    // ..
};

var data = {
    studentIDs: [ 14, 73, 112, 6 ],
    // ..
};

function setupButtonHandler(btn) {
    var recordKind = btn.dataset.kind;

    btn.addEventListener(
        "click",
        function makeRequest(evt){
            ajax(
                APIendpoints[recordKind],
                data[recordKind]
            );
        }
    );
}

// <button data-kind="studentIDs">
//    Register Students
// </button>

setupButtonHandler(btn);
```

Com a abordagem `setupButtonHandler(..)`, o atributo `data-kind` é recuperado uma vez e atribuído à variável `recordKind` na configuração inicial. `recordKind` então fica sob closure do manipulador de clique interno `makeRequest(..)`, e seu valor é usado a cada disparo do evento para buscar a URL e os dados que devem ser enviados.

| NOTA: |
| :--- |
| `evt` ainda é passado para `makeRequest(..)`, embora, neste caso, não o estejamos mais usando. Ele continua listado por consistência com o trecho anterior. |

Ao colocar `recordKind` dentro de `setupButtonHandler(..)`, limitamos a exposição de escopo dessa variável a um subconjunto mais apropriado do programa; armazená-la globalmente teria sido pior para a organização e a legibilidade do código. A closure permite que a instância da função interna `makeRequest()` *lembre* essa variável e a acesse sempre que for necessário.

Construindo sobre esse padrão, poderíamos ter buscado tanto a URL quanto os dados uma única vez, na configuração:

```js
function setupButtonHandler(btn) {
    var recordKind = btn.dataset.kind;
    var requestURL = APIendpoints[recordKind];
    var requestData = data[recordKind];

    btn.addEventListener(
        "click",
        function makeRequest(evt){
            ajax(requestURL,requestData);
        }
    );
}
```

Agora `makeRequest(..)` faz closure sobre `requestURL` e `requestData`, o que é um pouco mais limpo de entender e também ligeiramente mais performático.

Duas técnicas similares do paradigma de Programação Funcional (FP) que dependem de closure são a aplicação parcial (*partial application*) e o currying. Resumidamente, com essas técnicas, alteramos a *forma* de funções que exigem múltiplas entradas, de modo que algumas entradas sejam fornecidas antecipadamente e outras entradas sejam fornecidas depois; as entradas iniciais são lembradas via closure. Uma vez que todas as entradas tenham sido fornecidas, a ação subjacente é realizada.

Ao criar uma instância de função que encapsula alguma informação dentro dela (via closure), a função-com-informação-armazenada pode depois ser usada diretamente, sem precisar fornecer essa entrada novamente. Isso torna aquela parte do código mais limpa e também oferece a oportunidade de rotular funções parcialmente aplicadas com nomes semânticos melhores.

Adaptando a aplicação parcial, podemos melhorar ainda mais o código anterior:

```js
function defineHandler(requestURL,requestData) {
    return function makeRequest(evt){
        ajax(requestURL,requestData);
    };
}

function setupButtonHandler(btn) {
    var recordKind = btn.dataset.kind;
    var handler = defineHandler(
        APIendpoints[recordKind],
        data[recordKind]
    );
    btn.addEventListener("click",handler);
}
```

As entradas `requestURL` e `requestData` são fornecidas antecipadamente, resultando na função parcialmente aplicada `makeRequest(..)`, que rotulamos localmente como `handler`. Quando o evento eventualmente dispara, a entrada final (`evt`, mesmo que seja ignorada) é passada para `handler()`, completando suas entradas e disparando a requisição Ajax subjacente.

Em termos de comportamento, este programa é bem parecido com o anterior, com o mesmo tipo de closure. Mas, ao isolar a criação de `makeRequest(..)` em um utilitário separado (`defineHandler(..)`), tornamos essa definição mais reutilizável ao longo do programa. Também limitamos explicitamente o escopo da closure apenas às duas variáveis necessárias.

## Mais Perto da Closure

Enquanto encerramos um capítulo denso, respire fundo algumas vezes e deixe tudo assentar. Sério, é muita informação para qualquer um consumir!

Exploramos dois modelos para encarar mentalmente a closure:

* Observacional: closure é uma instância de função lembrando suas variáveis externas mesmo enquanto essa função é passada para e **invocada em** outros escopos.

* De implementação: closure é uma instância de função e seu ambiente de escopo preservados no lugar, enquanto quaisquer referências a ela são passadas adiante e **invocadas a partir de** outros escopos.

Resumindo os benefícios para nossos programas:

* A closure pode melhorar a eficiência ao permitir que uma instância de função lembre informação previamente determinada, em vez de ter que computá-la a cada vez.

* A closure pode melhorar a legibilidade do código, limitando a exposição de escopo ao encapsular variável(is) dentro de instâncias de função, ao mesmo tempo em que garante que a informação nessas variáveis esteja acessível para uso futuro. As instâncias de função resultantes, mais estreitas e especializadas, são mais limpas de interagir, já que a informação preservada não precisa ser passada a cada invocação.

Antes de seguir adiante, tire um tempo para reformular este resumo *com suas próprias palavras*, explicando o que é closure e por que ela é útil nos seus programas. O texto principal do livro se conclui com um capítulo final que constrói sobre a closure com o padrão de módulo.
