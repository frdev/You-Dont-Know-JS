# You Don't Know JS Yet: Scope & Closures - 2ª Edição
# Capítulo 5: O Ciclo de Vida (Nem Tão) Secreto das Variáveis

A esta altura você já deve ter um domínio decente do aninhamento de escopos, do escopo global para baixo — o que chamamos de cadeia de escopos de um programa.

Mas apenas saber de qual escopo uma variável vem é só parte da história. Se uma declaração de variável aparece depois da primeira instrução de um escopo, como vão se comportar quaisquer referências a esse identificador *antes* da declaração? O que acontece se você tentar declarar a mesma variável duas vezes em um escopo?

O sabor particular de escopo léxico do JS é rico em nuances sobre como e quando variáveis passam a existir e ficam disponíveis para o programa.

## Quando Posso Usar uma Variável?

A partir de que ponto uma variável fica disponível para uso dentro do seu escopo? Pode parecer haver uma resposta óbvia: *depois* de a variável ter sido declarada/criada. Certo? Não exatamente.

Considere:

```js
greeting();
// Hello!

function greeting() {
    console.log("Hello!");
}
```

Este código funciona perfeitamente. Você já deve ter visto ou até escrito código assim antes. Mas você já se perguntou como ou por que ele funciona? Especificamente, por que você pode acessar o identificador `greeting` na linha 1 (para recuperar e executar uma referência de função), mesmo que a declaração da função `greeting()` só ocorra na linha 4?

Lembre-se de que o Capítulo 1 aponta que todos os identificadores são registrados em seus respectivos escopos em tempo de compilação. Além disso, todo identificador é *criado* no início do escopo ao qual pertence, **toda vez que esse escopo é acessado**.

O termo mais comumente usado para uma variável ser visível desde o início do seu escopo envolvente, ainda que sua declaração apareça mais abaixo no escopo, é **hoisting**.

Mas o hoisting sozinho não responde totalmente à pergunta. Podemos ver um identificador chamado `greeting` desde o início do escopo, mas por que podemos **chamar** a função `greeting()` antes de ela ter sido declarada?

Em outras palavras, como a variável `greeting` tem algum valor (a referência da função) atribuído a ela desde o momento em que o escopo começa a rodar? A resposta é uma característica especial das declarações formais de `function`, chamada *function hoisting*. Quando o identificador de nome de uma declaração de `function` é registrado no topo do seu escopo, ele é adicionalmente autoinicializado com a referência daquela função. É por isso que a função pode ser chamada em todo o escopo!

Um detalhe fundamental é que tanto o *function hoisting* quanto o *hoisting de variáveis* do tipo `var` anexam seus identificadores de nome ao **escopo de função** envolvente mais próximo (ou, se não houver nenhum, ao escopo global), e não a um escopo de bloco.

| NOTA: |
| :--- |
| Declarações com `let` e `const` ainda sofrem hoisting (veja a discussão sobre TDZ mais adiante neste capítulo). Mas essas duas formas de declaração se anexam ao bloco envolvente, e não apenas a uma função envolvente, como acontece com declarações `var` e `function`. Veja "Criando Escopo com Blocos", no Capítulo 6, para mais informações. |

### Hoisting: Declaração vs. Expressão

O *function hoisting* só se aplica a declarações formais de `function` (especificamente aquelas que aparecem fora de blocos — veja "FiB", no Capítulo 6), não a atribuições de expressões de `function`. Considere:

```js
greeting();
// TypeError

var greeting = function greeting() {
    console.log("Hello!");
};
```

A linha 1 (`greeting();`) lança um erro. Mas o *tipo* de erro lançado é muito importante de notar. Um `TypeError` significa que estamos tentando fazer algo com um valor que não é permitido. Dependendo do seu ambiente JS, a mensagem de erro diria algo como "'undefined' is not a function" ou, de forma mais útil, "'greeting' is not a function".

Note que o erro **não** é um `ReferenceError`. O JS não está nos dizendo que não conseguiu encontrar `greeting` como identificador no escopo. Está nos dizendo que `greeting` foi encontrado, mas não guarda uma referência de função naquele momento. Só funções podem ser invocadas, então tentar invocar algum valor que não é função resulta em erro.

Mas o que `greeting` guarda, se não a referência da função?

Além de sofrerem hoisting, variáveis declaradas com `var` também são automaticamente inicializadas como `undefined` no início do seu escopo — de novo, a função envolvente mais próxima, ou o global. Uma vez inicializadas, ficam disponíveis para uso (atribuição, leitura etc.) em todo o escopo.

Então, naquela primeira linha, `greeting` existe, mas guarda apenas o valor padrão `undefined`. Só na linha 4 é que `greeting` recebe a referência da função.

Preste bastante atenção à distinção aqui. Uma declaração de `function` sofre hoisting **e é inicializada com o valor da função** (de novo, o chamado *function hoisting*). Uma variável `var` também sofre hoisting, e então é autoinicializada como `undefined`. Quaisquer atribuições subsequentes de expressões de `function` a essa variável só acontecem quando essa atribuição é processada durante a execução em tempo de execução.

Nos dois casos, o nome do identificador sofre hoisting. Mas a associação com a referência da função não é tratada no momento da inicialização (início do escopo), a menos que o identificador tenha sido criado em uma declaração formal de `function`.

### Hoisting de Variáveis

Vamos ver outro exemplo de *hoisting de variáveis*:

```js
greeting = "Hello!";
console.log(greeting);
// Hello!

var greeting = "Howdy!";
```

Embora `greeting` só seja declarada na linha 5, ela já está disponível para receber atribuição já na linha 1. Por quê?

Há duas partes necessárias para a explicação:

* o identificador sofre hoisting,
* **e** ele é automaticamente inicializado com o valor `undefined` a partir do topo do escopo.

| NOTA: |
| :--- |
| Usar *hoisting de variáveis* desse tipo provavelmente parece antinatural, e muitos leitores podem, com razão, querer evitar depender disso em seus programas. Mas todo hoisting (incluindo o *function hoisting*) deve ser evitado? Vamos explorar essas diferentes perspectivas sobre hoisting com mais detalhes no Apêndice A. |

## Hoisting: Mais Uma Metáfora

O Capítulo 2 foi cheio de metáforas (para ilustrar escopo), mas aqui nos deparamos com mais uma: o próprio hoisting. Em vez de o hoisting ser um passo de execução concreto que a engine JS realiza, é mais útil pensar no hoisting como uma visualização de várias ações que o JS realiza ao preparar o programa **antes da execução**.

A afirmação típica do que hoisting significa: *içar* — como içar um peso para cima — quaisquer identificadores até o topo de um escopo. A explicação frequentemente afirmada é que a engine JS, na verdade, *reescreve* esse programa antes da execução, de modo que ele fique mais parecido com isto:

```js
var greeting;           // declaração içada
greeting = "Hello!";    // a linha 1 original
console.log(greeting);  // Hello!
greeting = "Howdy!";    // o `var` sumiu!
```

O hoisting (metáfora) propõe que o JS pré-processa o programa original e o reorganiza um pouco, de modo que todas as declarações sejam movidas para o topo de seus respectivos escopos, antes da execução. Além disso, a metáfora do hoisting afirma que declarações de `function` são, em sua totalidade, içadas para o topo de cada escopo. Considere:

```js
studentName = "Suzy";
greeting();
// Hello Suzy!

function greeting() {
    console.log(`Hello ${ studentName }!`);
}
var studentName;
```

A "regra" da metáfora do hoisting é que declarações de função são içadas primeiro, e então as variáveis são içadas imediatamente depois de todas as funções. Assim, a história do hoisting sugere que o programa é *reorganizado* pela engine JS para ficar assim:

```js
function greeting() {
    console.log(`Hello ${ studentName }!`);
}
var studentName;

studentName = "Suzy";
greeting();
// Hello Suzy!
```

Essa metáfora do hoisting é conveniente. Seu benefício é nos permitir passar batido sobre o mágico pré-processamento com "olhar adiante" necessário para encontrar todas essas declarações enterradas fundo nos escopos e, de alguma forma, movê-las (içá-las) para o topo; podemos simplesmente pensar no programa como se fosse executado pela engine JS em uma **única passagem**, de cima para baixo.

Uma única passagem definitivamente parece mais direta do que a afirmação do Capítulo 1 sobre um processamento em duas fases.

O hoisting como mecanismo de reordenação de código pode ser uma simplificação atraente, mas não é preciso. A engine JS, na verdade, não reorganiza o código. Ela não pode magicamente olhar adiante e encontrar declarações; a única forma de encontrá-las com precisão, assim como todos os limites de escopo do programa, seria fazendo o parsing completo do código.

Adivinha o que é o parsing? A primeira fase do processamento em duas fases! Não há ginástica mental mágica que escape desse fato.

Então, se a metáfora do hoisting é (na melhor das hipóteses) imprecisa, o que devemos fazer com o termo? Acho que ele ainda é útil — de fato, até membros do TC39 o usam regularmente! —, mas não acho que devamos afirmar que se trata de uma reorganização real do código-fonte.

| AVISO: |
| :--- |
| Modelos mentais incorretos ou incompletos frequentemente ainda parecem suficientes, porque podem ocasionalmente levar a respostas certas por acidente. Mas, no longo prazo, é mais difícil analisar e prever resultados com precisão se seu raciocínio não está particularmente alinhado com o funcionamento da engine JS. |

Eu afirmo que hoisting *deveria* ser usado para se referir à **operação em tempo de compilação** de gerar instruções de tempo de execução para o registro automático de uma variável no início do seu escopo, cada vez que esse escopo é acessado.

Essa é uma mudança sutil, mas importante: do hoisting como comportamento de tempo de execução para seu devido lugar entre as tarefas de tempo de compilação.

## Redeclaração?

O que você acha que acontece quando uma variável é declarada mais de uma vez no mesmo escopo? Considere:

```js
var studentName = "Frank";
console.log(studentName);
// Frank

var studentName;
console.log(studentName);   // ???
```

O que você espera que seja impresso naquela segunda mensagem? Muita gente acredita que o segundo `var studentName` redeclarou a variável (e, portanto, a "resetou"), então esperam que `undefined` seja impresso.

Mas existe tal coisa como uma variável ser "redeclarada" no mesmo escopo? Não.

Se você considerar este programa da perspectiva da metáfora do hoisting, o código seria reorganizado assim para fins de execução:

```js
var studentName;
var studentName;    // claramente um no-op inútil!

studentName = "Frank";
console.log(studentName);
// Frank

console.log(studentName);
// Frank
```

Como o hoisting, na verdade, é sobre registrar uma variável no início de um escopo, não há nada a ser feito no meio do escopo, onde o programa original de fato tinha a segunda instrução `var studentName`. É apenas um no-op(eration), uma instrução inútil.

| DICA: |
| :--- |
| No estilo da narrativa de conversa do Capítulo 2, o *Compilador* encontraria a segunda instrução de declaração `var` e perguntaria ao *Gerente de Escopo* se ele já tinha visto um identificador `studentName`; como já tinha, não haveria mais nada a fazer. |

Também é importante apontar que `var studentName;` não significa `var studentName = undefined;`, como a maioria supõe. Vamos provar que são diferentes considerando esta variação do programa:

```js
var studentName = "Frank";
console.log(studentName);   // Frank

var studentName;
console.log(studentName);   // Frank <--- ainda!

// vamos adicionar a inicialização explicitamente
var studentName = undefined;
console.log(studentName);   // undefined <--- viu!?
```

Percebe como a inicialização explícita `= undefined` produz um resultado diferente de supor que ela acontece implicitamente quando omitida? Na próxima seção, vamos revisitar esse tópico de inicialização de variáveis a partir de suas declarações.

Uma declaração `var` repetida com o mesmo nome de identificador em um escopo é, na prática, uma operação que não faz nada. Aqui vai outra ilustração, desta vez envolvendo uma função de mesmo nome:

```js
var greeting;

function greeting() {
    console.log("Hello!");
}

// basicamente, um no-op
var greeting;

typeof greeting;        // "function"

var greeting = "Hello!";

typeof greeting;        // "string"
```

A primeira declaração `greeting` registra o identificador no escopo e, por ser um `var`, a autoinicialização será `undefined`. A declaração de `function` não precisa registrar o identificador de novo, mas, por causa do *function hoisting*, ela sobrepõe a autoinicialização para usar a referência da função. O segundo `var greeting`, sozinho, não faz nada, já que `greeting` já é um identificador e o *function hoisting* já teve precedência na autoinicialização.

Atribuir de fato `"Hello!"` a `greeting` muda seu valor da função inicial `greeting()` para a string; o `var` em si não tem efeito algum.

E quanto a repetir uma declaração dentro de um escopo usando `let` ou `const`?

```js
let studentName = "Frank";

console.log(studentName);

let studentName = "Suzy";
```

Este programa não vai executar; em vez disso, vai lançar imediatamente um `SyntaxError`. Dependendo do seu ambiente JS, a mensagem de erro vai indicar algo como: "studentName has already been declared." Em outras palavras, este é um caso em que a tentativa de "redeclaração" é explicitamente proibida!

Não é só o caso de duas declarações envolvendo `let` lançarem esse erro. Se qualquer uma das declarações usar `let`, a outra pode ser `let` ou `var`, e o erro ainda vai ocorrer, como ilustrado nestas duas variações:

```js
var studentName = "Frank";

let studentName = "Suzy";
```

e:

```js
let studentName = "Frank";

var studentName = "Suzy";
```

Em ambos os casos, um `SyntaxError` é lançado na *segunda* declaração. Em outras palavras, a única forma de "redeclarar" uma variável é usar `var` em todas (duas ou mais) as suas declarações.

Mas por que proibir isso? A razão do erro não é técnica em si, já que a "redeclaração" com `var` sempre foi permitida; claramente, a mesma permissão poderia ter sido dada ao `let`.

É mais uma questão de "engenharia social". A "redeclaração" de variáveis é vista por alguns, incluindo muitos no corpo do TC39, como um mau hábito que pode levar a bugs no programa. Então, quando o ES6 introduziu `let`, eles decidiram impedir a "redeclaração" com um erro.

| NOTA: |
| :--- |
| Isso é, claro, uma opinião estilística, e não propriamente um argumento técnico. Muitos desenvolvedores concordam com a posição, e provavelmente em parte por isso o TC39 incluiu o erro (além de fazer `let` se conformar a `const`). Mas seria possível argumentar razoavelmente que manter consistência com o precedente do `var` seria mais prudente, e que impor esse tipo de opinião seria melhor deixado a ferramentas opcionais, como linters. No Apêndice A, vamos explorar se `var` (e seu comportamento associado, como a "redeclaração") ainda pode ser útil no JS moderno. |

Quando o *Compilador* pergunta ao *Gerente de Escopo* sobre uma declaração, se aquele identificador já tiver sido declarado, e se uma/ambas as declarações foram feitas com `let`, um erro é lançado. O sinal pretendido para o desenvolvedor é: "Pare de depender de redeclaração desleixada!"

### Constantes?

A palavra-chave `const` é mais restrita do que `let`. Assim como `let`, `const` não pode ser repetida com o mesmo identificador no mesmo escopo. Mas existe, de fato, uma razão técnica predominante pela qual esse tipo de "redeclaração" é proibido, diferente de `let`, que proíbe a "redeclaração" sobretudo por razões estilísticas.

A palavra-chave `const` exige que uma variável seja inicializada, então omitir uma atribuição na declaração resulta em um `SyntaxError`:

```js
const empty;   // SyntaxError
```

Declarações `const` criam variáveis que não podem ser reatribuídas:

```js
const studentName = "Frank";
console.log(studentName);
// Frank

studentName = "Suzy";   // TypeError
```

A variável `studentName` não pode ser reatribuída porque foi declarada com `const`.

| AVISO: |
| :--- |
| O erro lançado ao reatribuir `studentName` é um `TypeError`, não um `SyntaxError`. A distinção sutil aqui é bem importante, mas infelizmente fácil demais de passar despercebida. Erros de sintaxe representam falhas no programa que impedem até que ele comece a executar. Erros de tipo representam falhas que surgem durante a execução do programa. No trecho anterior, `"Frank"` é impresso antes de processarmos a reatribuição de `studentName`, que então lança o erro. |

Então, se declarações `const` não podem ser reatribuídas, e declarações `const` sempre exigem atribuições, então temos uma razão técnica clara para o `const` ter de proibir qualquer "redeclaração": qualquer "redeclaração" de `const` seria também, necessariamente, uma reatribuição de `const`, o que não pode ser permitido!

```js
const studentName = "Frank";

// obviamente isto tem que ser um erro
const studentName = "Suzy";
```

Como a "redeclaração" de `const` precisa ser proibida (por essas razões técnicas), o TC39 essencialmente achou que a "redeclaração" de `let` também deveria ser proibida, por consistência. É discutível se essa foi a melhor escolha, mas pelo menos temos o raciocínio por trás da decisão.

### Laços

Então, ficou claro pela nossa discussão anterior que o JS não quer muito que "redeclaremos" nossas variáveis dentro do mesmo escopo. Isso provavelmente parece uma advertência direta, até você considerar o que isso significa para a execução repetida de instruções de declaração em laços. Considere:

```js
var keepGoing = true;
while (keepGoing) {
    let value = Math.random();
    if (value > 0.5) {
        keepGoing = false;
    }
}
```

`value` está sendo "redeclarada" repetidamente neste programa? Vamos receber erros? Não.

Todas as regras de escopo (incluindo a "redeclaração" de variáveis criadas com `let`) são aplicadas *por instância de escopo*. Em outras palavras, cada vez que um escopo é acessado durante a execução, tudo é reiniciado.

Cada iteração do laço é sua própria nova instância de escopo e, dentro de cada instância de escopo, `value` está sendo declarada apenas uma vez. Então não há tentativa de "redeclaração" e, portanto, nenhum erro. Antes de considerarmos outras formas de laço, e se a declaração de `value` no trecho anterior fosse mudada para `var`?

```js
var keepGoing = true;
while (keepGoing) {
    var value = Math.random();
    if (value > 0.5) {
        keepGoing = false;
    }
}
```

`value` está sendo "redeclarada" aqui, especialmente porque sabemos que `var` permite isso? Não. Como `var` não é tratada como uma declaração com escopo de bloco (veja o Capítulo 6), ela se anexa ao escopo global. Então existe apenas uma variável `value`, no mesmo escopo que `keepGoing` (escopo global, neste caso). Nenhuma "redeclaração" aqui, também!

Uma forma de manter tudo isso claro é lembrar que as palavras-chave `var`, `let` e `const` são, na prática, *removidas* do código quando ele começa a executar. Elas são tratadas inteiramente pelo compilador.

Se você apagar mentalmente as palavras-chave declaradoras e então tentar processar o código, isso deve te ajudar a decidir se e quando (re)declarações podem ocorrer.

E quanto à "redeclaração" com outras formas de laço, como laços `for`?

```js
for (let i = 0; i < 3; i++) {
    let value = i * 10;
    console.log(`${ i }: ${ value }`);
}
// 0: 0
// 1: 10
// 2: 20
```

Deve estar claro que há apenas um `value` declarado por instância de escopo. Mas e quanto a `i`? Ele está sendo "redeclarado"?

Para responder a isso, considere em qual escopo `i` está. Pode parecer que ele estaria no escopo externo (neste caso, o global), mas não está. Ele está no escopo do corpo do laço `for`, assim como `value`. De fato, você poderia pensar nesse laço nesta forma equivalente, mais verbosa:

```js
{
    // uma variável fictícia, para ilustração
    let $$i = 0;

    for ( /* nada */; $$i < 3; $$i++) {
        // aqui está o nosso `i` de verdade!
        let i = $$i;

        let value = i * 10;
        console.log(`${ i }: ${ value }`);
    }
    // 0: 0
    // 1: 10
    // 2: 20
}
```

Agora deve estar claro: as variáveis `i` e `value` são ambas declaradas exatamente uma vez **por instância de escopo**. Nenhuma "redeclaração" aqui.

E quanto a outras formas de laço `for`?

```js
for (let index in students) {
    // isto está ok
}

for (let student of students) {
    // isto também
}
```

Mesma coisa com laços `for..in` e `for..of`: a variável declarada é tratada como *dentro* do corpo do laço e, portanto, é tratada por iteração (ou seja, por instância de escopo). Nenhuma "redeclaração".

Ok, eu sei que você está achando que eu pareço um disco arranhado a esta altura. Mas vamos explorar como `const` impacta essas construções de laço. Considere:

```js
var keepGoing = true;
while (keepGoing) {
    // ooh, uma constante brilhante!
    const value = Math.random();
    if (value > 0.5) {
        keepGoing = false;
    }
}
```

Assim como na variante com `let` deste programa que vimos antes, o `const` está sendo executado exatamente uma vez dentro de cada iteração do laço, então está a salvo de problemas de "redeclaração". Mas as coisas ficam mais complicadas quando falamos de laços `for`.

`for..in` e `for..of` podem ser usados tranquilamente com `const`:

```js
for (const index in students) {
    // isto está ok
}

for (const student of students) {
    // isto também está ok
}
```

Mas não o laço `for` geral:

```js
for (const i = 0; i < 3; i++) {
    // ops, isto vai falhar com um
    // Type Error depois da primeira iteração
}
```

O que há de errado aqui? Poderíamos usar `let` tranquilamente nessa construção, e afirmamos que isso cria um novo `i` para o escopo de cada iteração do laço, então nem parece ser uma "redeclaração".

Vamos "expandir" mentalmente esse laço, como fizemos antes:

```js
{
    // uma variável fictícia, para ilustração
    const $$i = 0;

    for ( ; $$i < 3; $$i++) {
        // aqui está o nosso `i` de verdade!
        const i = $$i;
        // ..
    }
}
```

Você percebe o problema? Nosso `i` é, de fato, criado apenas uma vez dentro do laço. Esse não é o problema. O problema é o `$$i` conceitual, que precisa ser incrementado a cada vez com a expressão `$$i++`. Isso é **reatribuição** (não "redeclaração"), o que não é permitido para constantes.

Lembre-se: essa forma "expandida" é apenas um modelo conceitual para te ajudar a intuir a origem do problema. Você pode se perguntar se o JS poderia efetivamente ter transformado o `const $$i = 0` em `let $ii = 0`, o que então permitiria que `const` funcionasse com o nosso laço `for` clássico. É possível, mas isso poderia ter introduzido exceções potencialmente surpreendentes à semântica do laço `for`.

Por exemplo, teria sido uma exceção sutil bastante arbitrária (e provavelmente confusa) permitir que `i++` no cabeçalho do laço `for` escapasse do rigor da atribuição `const`, mas não permitir outras reatribuições de `i` dentro da iteração do laço, o que às vezes é útil.

A resposta direta é: `const` não pode ser usado com a forma clássica de laço `for` por causa da reatribuição exigida.

Curiosamente, se você não fizer reatribuição, então é válido:

```js
var keepGoing = true;

for (const i = 0; keepGoing; /* nada aqui */ ) {
    keepGoing = (Math.random() > 0.5);
    // ..
}
```

Isso funciona, mas é inútil. Não há razão para declarar `i` naquela posição com `const`, já que todo o propósito de tal variável naquela posição é **ser usada para contar iterações**. Basta usar outra forma de laço, como um laço `while`, ou usar um `let`!

## Variáveis Não Inicializadas (ou seja, a TDZ)

Com declarações `var`, a variável sofre "hoisting" para o topo do seu escopo. Mas ela também é automaticamente inicializada com o valor `undefined`, de modo que a variável possa ser usada em todo o escopo.

Porém, declarações `let` e `const` não são exatamente iguais nesse aspecto.

Considere:

```js
console.log(studentName);
// ReferenceError

let studentName = "Suzy";
```

O resultado deste programa é que um `ReferenceError` é lançado na primeira linha. Dependendo do seu ambiente JS, a mensagem de erro pode dizer algo como: "Cannot access studentName before initialization."

| NOTA: |
| :--- |
| A mensagem de erro como vista aqui costumava ser muito mais vaga ou enganosa. Felizmente, vários de nós na comunidade conseguimos com sucesso pressionar as engines JS a melhorar essa mensagem de erro, para que ela diga com mais precisão o que está errado! |

Essa mensagem de erro é bem indicativa do que está errado: `studentName` existe na linha 1, mas não foi inicializada, então ainda não pode ser usada. Vamos tentar isto:

```js
studentName = "Suzy";   // vamos tentar inicializá-la!
// ReferenceError

console.log(studentName);

let studentName;
```

Ops. Ainda recebemos o `ReferenceError`, mas agora na primeira linha, onde estamos tentando atribuir (ou seja, inicializar!) essa tal variável "não inicializada" `studentName`. Qual é a jogada!?

A verdadeira pergunta é: como inicializamos uma variável não inicializada? Para `let`/`const`, a **única forma** de fazer isso é com uma atribuição acoplada a uma instrução de declaração. Uma atribuição sozinha é insuficiente! Considere:

```js
let studentName = "Suzy";
console.log(studentName);   // Suzy
```

Aqui, estamos inicializando `studentName` (neste caso, com `"Suzy"` em vez de `undefined`) por meio da forma de instrução de declaração `let` acoplada a uma atribuição.

Alternativamente:

```js
// ..

let studentName;
// ou:
// let studentName = undefined;

// ..

studentName = "Suzy";

console.log(studentName);
// Suzy
```

| NOTA: |
| :--- |
| Isso é interessante! Lembre-se de que, antes, dissemos que `var studentName;` *não* é o mesmo que `var studentName = undefined;`, mas aqui, com `let`, eles se comportam da mesma forma. A diferença se resume ao fato de que `var studentName` se autoinicializa no topo do escopo, enquanto `let studentName` não. |

Lembre-se de que já afirmamos algumas vezes até aqui que o *Compilador* acaba removendo quaisquer declaradores `var`/`let`/`const`, substituindo-os por instruções no topo de cada escopo para registrar os identificadores apropriados.

Então, se analisarmos o que está acontecendo aqui, vemos que uma nuance adicional é que o *Compilador* também está adicionando uma instrução no meio do programa, no ponto em que a variável `studentName` foi declarada, para tratar a autoinicialização dessa declaração. Não podemos usar a variável em nenhum ponto anterior a essa inicialização ocorrer. O mesmo vale para `const` e para `let`.

O termo cunhado pelo TC39 para se referir a esse *período de tempo*, desde a entrada em um escopo até o ponto em que ocorre a autoinicialização da variável, é: Zona Morta Temporal (TDZ, de *Temporal Dead Zone*).

A TDZ é a janela de tempo em que uma variável existe, mas ainda está não inicializada e, portanto, não pode ser acessada de forma alguma. Apenas a execução das instruções deixadas pelo *Compilador* no ponto da declaração original pode fazer essa inicialização. Depois desse momento, a TDZ acaba, e a variável fica livre para ser usada no restante do escopo.

Um `var` também tem, tecnicamente, uma TDZ, mas ela tem comprimento zero e, portanto, é inobservável nos nossos programas! Apenas `let` e `const` têm uma TDZ observável.

A propósito, "temporal" em TDZ se refere mesmo a *tempo*, e não a *posição no código*. Considere:

```js
askQuestion();
// ReferenceError

let studentName = "Suzy";

function askQuestion() {
    console.log(`${ studentName }, do you know?`);
}
```

Embora, posicionalmente, o `console.log(..)` que referencia `studentName` venha *depois* da declaração `let studentName`, em termos de tempo a função `askQuestion()` é invocada *antes* de a instrução `let` ser encontrada, enquanto `studentName` ainda está na sua TDZ! Daí o erro.

Há um equívoco comum de que a TDZ significa que `let` e `const` não sofrem hoisting. Essa é uma afirmação imprecisa, ou pelo menos ligeiramente enganosa. Eles definitivamente sofrem hoisting.

A diferença real é que declarações `let`/`const` não se autoinicializam no início do escopo, do jeito que `var` faz. O *debate*, então, é se a autoinicialização é *parte do* hoisting ou não. Eu acho que o autorregistro de uma variável no topo do escopo (ou seja, o que chamo de "hoisting") e a autoinicialização no topo do escopo (com `undefined`) são operações distintas e não deveriam ser agrupadas sob o termo único "hoisting".

Já vimos que `let` e `const` não se autoinicializam no topo do escopo. Mas vamos provar que `let` e `const` *sofrem*, sim, hoisting (autorregistro no topo do escopo), com a ajuda do nosso amigo sombreamento (veja "Sombreamento", no Capítulo 3):

```js
var studentName = "Kyle";

{
    console.log(studentName);
    // ???

    // ..

    let studentName = "Suzy";

    console.log(studentName);
    // Suzy
}
```

O que vai acontecer com a primeira instrução `console.log(..)`? Se `let studentName` não sofresse hoisting para o topo do escopo, então o primeiro `console.log(..)` *deveria* imprimir `"Kyle"`, certo? Naquele momento, ao que parece, apenas o `studentName` externo existiria, então essa seria a variável que o `console.log(..)` acessaria e imprimiria.

Mas, em vez disso, o primeiro `console.log(..)` lança um erro de TDZ, porque, de fato, o `studentName` do escopo interno **sofreu** hoisting (autorregistro no topo do escopo). O que **não** aconteceu (ainda!) foi a autoinicialização daquele `studentName` interno; ele continua não inicializado naquele momento, daí a violação de TDZ!

Então, resumindo, erros de TDZ ocorrem porque declarações `let`/`const` *sim*, içam suas declarações para o topo de seus escopos, mas, diferente de `var`, adiam a autoinicialização de suas variáveis até o momento, na sequência do código, em que a declaração original apareceu. Essa janela de tempo (dica: temporal), qualquer que seja seu comprimento, é a TDZ.

Como você pode evitar erros de TDZ?

Meu conselho: sempre coloque suas declarações `let` e `const` no topo de qualquer escopo. Encolha a janela da TDZ para comprimento zero (ou quase zero), e então ela se torna irrelevante.

Mas por que a TDZ sequer existe? Por que o TC39 não determinou que `let`/`const` se autoinicializassem do jeito que `var` faz? Só tenha paciência, vamos voltar a explorar o *porquê* da TDZ no Apêndice A.

## Finalmente Inicializadas

Trabalhar com variáveis tem muito mais nuance do que parece à primeira vista. *Hoisting*, *(re)declaração* e a *TDZ* são fontes comuns de confusão para desenvolvedores, especialmente para quem trabalhou em outras linguagens antes de chegar ao JS. Antes de seguir adiante, certifique-se de que seu modelo mental está totalmente fundamentado nesses aspectos de escopo e variáveis do JS.

O hoisting é geralmente citado como um mecanismo explícito da engine JS, mas é, na verdade, mais uma metáfora para descrever as várias formas como o JS lida com declarações de variáveis durante a compilação. Mas, mesmo como metáfora, o hoisting oferece uma estrutura útil para pensar sobre o ciclo de vida de uma variável — quando ela é criada, quando fica disponível para uso, quando desaparece.

Declaração e redeclaração de variáveis tendem a causar confusão quando pensadas como operações de tempo de execução. Mas, se você mudar para um pensamento de tempo de compilação para essas operações, as esquisitices e *sombras* diminuem.

O erro de TDZ (zona morta temporal) é estranho e frustrante quando encontrado. Felizmente, a TDZ é relativamente simples de evitar se você sempre tiver o cuidado de colocar declarações `let`/`const` no topo de qualquer escopo.

Enquanto você navega com sucesso por essas voltas e reviravoltas do escopo de variáveis, o próximo capítulo vai apresentar os fatores que guiam nossas decisões de colocar nossas declarações em diversos escopos, especialmente em blocos aninhados.
