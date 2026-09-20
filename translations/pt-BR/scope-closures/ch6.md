# You Don't Know JS Yet: Scope & Closures - 2ª Edição
# Capítulo 6: Limitando a Exposição do Escopo

Até aqui nosso foco foi explicar a mecânica de como escopos e variáveis funcionam. Com essa base agora firmemente estabelecida, nossa atenção se eleva a um nível de pensamento mais alto: decisões e padrões que aplicamos ao programa inteiro.

Para começar, vamos ver como e por que devemos usar diferentes níveis de escopo (funções e blocos) para organizar as variáveis do nosso programa, especificamente para reduzir a superexposição de escopo.

## Exposição Mínima

Faz sentido que funções definam seus próprios escopos. Mas por que precisamos que blocos também criem escopos?

A engenharia de software articula uma disciplina fundamental, tipicamente aplicada à segurança de software, chamada "Princípio do Menor Privilégio" (POLP, de *The Principle of Least Privilege*). [^POLP] E uma variação desse princípio, que se aplica à nossa discussão atual, é tipicamente rotulada de "Exposição Mínima" (POLE, de *Least Exposure*).

O POLP expressa uma postura defensiva na arquitetura de software: componentes do sistema devem ser projetados para funcionar com o mínimo de privilégio, o mínimo de acesso, a mínima exposição. Se cada peça é conectada com as capacidades mínimas necessárias, o sistema como um todo fica mais forte do ponto de vista de segurança, porque um comprometimento ou falha de uma peça tem impacto minimizado sobre o resto do sistema.

Se o POLP foca no design de componentes em nível de sistema, a variante POLE, de *Exposição*, foca em um nível mais baixo; vamos aplicá-la à forma como escopos interagem entre si.

Seguindo o POLE, o que queremos minimizar a exposição? Simplesmente: as variáveis registradas em cada escopo.

Pense assim: por que você não deveria simplesmente colocar todas as variáveis do seu programa no escopo global? Isso provavelmente já soa como uma má ideia, mas vale considerar por quê. Quando variáveis usadas por uma parte do programa ficam expostas a outra parte do programa, via escopo, há três riscos principais que frequentemente surgem:

* **Colisões de Nomes**: se você usa um nome de variável/função comum e útil em duas partes diferentes do programa, mas o identificador vem de um escopo compartilhado (como o escopo global), então ocorre colisão de nomes, e é bem provável que surjam bugs, já que uma parte usa a variável/função de um jeito que a outra parte não espera.

    Por exemplo, imagine se todos os seus laços usassem uma única variável de índice global `i`, e então acontecesse de um laço em uma função rodar durante uma iteração de um laço de outra função; agora a variável compartilhada `i` recebe um valor inesperado.

* **Comportamento Inesperado**: se você expõe variáveis/funções cujo uso é, de outra forma, *privado* a um pedaço do programa, isso permite que outros desenvolvedores as usem de formas que você não pretendia, o que pode violar o comportamento esperado e causar bugs.

    Por exemplo, se a sua parte do programa presume que um array contém apenas números, mas o código de outra pessoa acessa e modifica o array para incluir booleanos e strings, seu código pode então se comportar mal de formas inesperadas.

    Pior: expor detalhes *privados* convida quem tem má intenção a tentar contornar limitações que você impôs, para fazer com a sua parte do software coisas que não deveriam ser permitidas.

* **Dependência Não Intencional**: se você expõe variáveis/funções desnecessariamente, isso convida outros desenvolvedores a usar e depender dessas peças de outra forma *privadas*. Embora isso não quebre seu programa hoje, cria um risco de refatoração no futuro, porque agora você não pode refatorar tão facilmente aquela variável ou função sem potencialmente quebrar outras partes do software que você não controla.

    Por exemplo, se seu código depende de um array de números e você depois decide que é melhor usar alguma outra estrutura de dados em vez de um array, agora você precisa assumir a responsabilidade de ajustar outras partes afetadas do software.

O POLE, aplicado ao escopo de variáveis/funções, essencialmente diz: por padrão, exponha o mínimo necessário, mantendo todo o resto o mais privado possível. Declare variáveis nos escopos mais restritos e profundamente aninhados possíveis, em vez de colocar tudo no escopo global (ou até no escopo de função externa).

Se você projeta seu software de acordo, tem uma chance muito maior de evitar (ou pelo menos minimizar) esses três riscos.

Considere:

```js
function diff(x,y) {
    if (x > y) {
        let tmp = x;
        x = y;
        y = tmp;
    }

    return y - x;
}

diff(3,7);      // 4
diff(7,5);      // 2
```

Nesta função `diff(..)`, queremos garantir que `y` seja maior ou igual a `x`, de modo que, ao subtrair (`y - x`), o resultado seja `0` ou maior. Se `x` for inicialmente maior (o resultado seria negativo!), trocamos `x` e `y` usando uma variável `tmp`, para manter o resultado positivo.

Neste exemplo simples, não parece importar se `tmp` está dentro do bloco `if` ou se pertence ao nível da função — certamente não deveria ser uma variável global! Porém, seguindo o princípio POLE, `tmp` deve ficar o mais escondida em escopo quanto possível. Então damos escopo de bloco a `tmp` (usando `let`) dentro do bloco `if`.

## Escondendo-se à Vista de Todos no Escopo (de Função)

Agora deve estar claro por que é importante esconder nossas declarações de variáveis e funções nos escopos mais baixos (mais profundamente aninhados) possíveis. Mas como fazemos isso?

Já vimos as palavras-chave `let` e `const`, que são declaradores com escopo de bloco; voltaremos a elas com mais detalhes em breve. Mas, primeiro, e quanto a esconder declarações `var` ou `function` em escopos? Isso pode ser feito facilmente envolvendo a declaração em um escopo de `function`.

Vamos considerar um exemplo em que o escopo de `function` pode ser útil.

A operação matemática "fatorial" (notada como "6!") é a multiplicação de um dado inteiro por todos os inteiros sucessivamente menores até `1` — na verdade, você pode parar em `2`, já que multiplicar por `1` não faz nada. Em outras palavras, "6!" é o mesmo que "6 * 5!", que é o mesmo que "6 * 5 * 4!", e assim por diante. Por causa da natureza da matemática envolvida, uma vez que o fatorial de um dado inteiro (como "4!") tenha sido calculado, não deveríamos precisar refazer esse trabalho, já que a resposta será sempre a mesma.

Então, se você calcular ingenuamente o fatorial de `6` e depois quiser calcular o fatorial de `7`, pode acabar recalculando desnecessariamente os fatoriais de todos os inteiros de 2 até 6. Se estiver disposto a trocar memória por velocidade, pode resolver esse desperdício de computação cacheando o fatorial de cada inteiro conforme ele é calculado:

```js
var cache = {};

function factorial(x) {
    if (x < 2) return 1;
    if (!(x in cache)) {
        cache[x] = x * factorial(x - 1);
    }
    return cache[x];
}

factorial(6);
// 720

cache;
// {
//     "2": 2,
//     "3": 6,
//     "4": 24,
//     "5": 120,
//     "6": 720
// }

factorial(7);
// 5040
```

Estamos armazenando todos os fatoriais computados em `cache`, para que, ao longo de múltiplas chamadas a `factorial(..)`, as computações anteriores permaneçam. Mas a variável `cache` é, bem obviamente, um detalhe *privado* de como `factorial(..)` funciona, não algo que deveria ser exposto em um escopo externo — especialmente não no escopo global.

| NOTA: |
| :--- |
| `factorial(..)` aqui é recursiva — uma chamada a si mesma é feita de dentro dela —, mas isso é apenas por brevidade de código; uma implementação não recursiva renderia a mesma análise de escopo com respeito a `cache`. |

Porém, corrigir esse problema de superexposição não é tão simples quanto esconder a variável `cache` dentro de `factorial(..)`, como pode parecer. Como precisamos que `cache` sobreviva a múltiplas chamadas, ela precisa estar localizada em um escopo fora dessa função. Então, o que podemos fazer?

Definir outro escopo intermediário (entre o escopo externo/global e o interior de `factorial(..)`) para `cache` ficar localizada:

```js
// escopo externo/global

function hideTheCache() {
    // "escopo intermediário", onde escondemos `cache`
    var cache = {};

    return factorial;

    // **********************

    function factorial(x) {
        // escopo interno
        if (x < 2) return 1;
        if (!(x in cache)) {
            cache[x] = x * factorial(x - 1);
        }
        return cache[x];
    }
}

var factorial = hideTheCache();

factorial(6);
// 720

factorial(7);
// 5040
```

A função `hideTheCache()` não serve a nenhum outro propósito além de criar um escopo em que `cache` persista ao longo de múltiplas chamadas a `factorial(..)`. Mas, para que `factorial(..)` tenha acesso a `cache`, precisamos definir `factorial(..)` dentro desse mesmo escopo. Então retornamos a referência da função, como valor de `hideTheCache()`, e a guardamos em uma variável de escopo externo, também chamada `factorial`. Agora, conforme chamamos `factorial(..)` (várias vezes!), seu `cache` persistente permanece escondido, mas acessível apenas a `factorial(..)`!

Ok, mas... vai ser tedioso definir (e nomear!) um escopo de função `hideTheCache(..)` cada vez que surgir tal necessidade de esconder variáveis/funções, especialmente porque provavelmente vamos querer evitar colisões de nome com essa função, dando a cada ocorrência um nome único. Eca.

| NOTA: |
| :--- |
| A técnica ilustrada — cachear a saída computada de uma função para otimizar desempenho quando chamadas repetidas com as mesmas entradas são esperadas — é bastante comum no mundo da Programação Funcional (FP), canonicamente chamada de "memoização"; esse cache depende de closure (veja o Capítulo 7). Além disso, há preocupações de uso de memória (tratadas em "Uma Palavra Sobre Memória", no Apêndice B). Bibliotecas de FP geralmente fornecem um utilitário otimizado e testado para memoização de funções, que tomaria o lugar do `hideTheCache(..)` aqui. Memoização está além do *escopo* (trocadilho intencional!) da nossa discussão, mas veja meu livro *Functional-Light JavaScript* para mais informações. |

Em vez de definir uma função nova e com nome único cada vez que ocorre uma dessas situações de escopo-só-para-esconder-uma-variável, uma solução talvez melhor é usar uma expressão de função:

```js
var factorial = (function hideTheCache() {
    var cache = {};

    function factorial(x) {
        if (x < 2) return 1;
        if (!(x in cache)) {
            cache[x] = x * factorial(x - 1);
        }
        return cache[x];
    }

    return factorial;
})();

factorial(6);
// 720

factorial(7);
// 5040
```

Espera! Isso ainda está usando uma função para criar o escopo que esconde `cache` e, neste caso, a função ainda se chama `hideTheCache`, então como isso resolve alguma coisa?

Lembre-se de "Escopo do Nome da Função" (no Capítulo 3), do que acontece com o identificador de nome de uma expressão de `function`. Como `hideTheCache(..)` é definida como uma expressão de `function`, e não como uma declaração de `function`, seu nome fica no seu próprio escopo — essencialmente o mesmo escopo de `cache` —, em vez de ficar no escopo externo/global.

Isso significa que podemos dar a toda ocorrência dessa expressão de função exatamente o mesmo nome, e nunca haverá colisão. Mais adequadamente, podemos nomear cada ocorrência semanticamente, com base no que quer que estejamos tentando esconder, sem nos preocupar se o nome escolhido vai colidir com qualquer outro escopo de expressão de `function` no programa.

De fato, *poderíamos* simplesmente omitir o nome por completo — definindo assim uma "expressão de `function` anônima". Mas o Apêndice A vai discutir a importância dos nomes mesmo para essas funções que existem só para criar escopo.

### Invocando Expressões de Função Imediatamente

Há outro detalhe importante no programa recursivo de fatorial anterior, fácil de passar batido: a linha no fim da expressão de `function` que contém `})();`.

Note que cercamos toda a expressão de `function` com um par de `( .. )` e, no fim, adicionamos aquele segundo par de parênteses `()`; isso, na verdade, está chamando a expressão de `function` que acabamos de definir. Além disso, neste caso, o primeiro par de `( .. )` ao redor da expressão de função não é estritamente necessário (mais sobre isso em um instante), mas os usamos mesmo assim em nome da legibilidade.

Então, em outras palavras, estamos definindo uma expressão de `function` que é então imediatamente invocada. Esse padrão comum tem um nome (muito criativo!): Expressão de Função Imediatamente Invocada (IIFE, de *Immediately Invoked Function Expression*).

Uma IIFE é útil quando queremos criar um escopo para esconder variáveis/funções. Como é uma expressão, pode ser usada em **qualquer** lugar de um programa JS em que uma expressão seja permitida. Uma IIFE pode ser nomeada, como em `hideTheCache()`, ou (muito mais comumente!) sem nome/anônima. E pode ser independente ou, como antes, parte de outra instrução — `hideTheCache()` retorna a referência da função `factorial()`, que é então atribuída com `=` à variável `factorial`.

Para comparação, aqui está um exemplo de uma IIFE independente:

```js
// escopo externo

(function(){
    // escopo interno escondido
})();

// mais escopo externo
```

Diferente do caso anterior com `hideTheCache()`, em que os `(..)` externos foram citados como uma escolha estilística opcional, para uma IIFE independente eles são **obrigatórios**; eles distinguem a `function` como uma expressão, e não como uma instrução. Por consistência, porém, sempre cerque uma `function` de IIFE com `( .. )`.

| NOTA: |
| :--- |
| Tecnicamente, os `( .. )` ao redor não são a única forma sintática de garantir que a `function` em uma IIFE seja tratada pelo parser do JS como uma expressão de função. Vamos ver algumas outras opções no Apêndice A. |

#### Fronteiras de Função

Cuidado: usar uma IIFE para definir um escopo pode ter algumas consequências não intencionais, dependendo do código ao redor. Como uma IIFE é uma função completa, a fronteira de função altera o comportamento de certas instruções/construções.

Por exemplo, uma instrução `return` em algum pedaço de código mudaria de significado se uma IIFE fosse envolvida em torno dele, porque agora o `return` se referiria à função da IIFE. IIFEs com funções que não são arrow também mudam o vínculo da palavra-chave `this` — mais sobre isso no livro *Objects & Classes*. E instruções como `break` e `continue` não vão operar através da fronteira de função de uma IIFE para controlar um laço ou bloco externo.

Então, se o código em torno do qual você precisa envolver um escopo tem `return`, `this`, `break` ou `continue`, uma IIFE provavelmente não é a melhor abordagem. Nesse caso, talvez você deva criar o escopo com um bloco, em vez de uma função.

## Criando Escopo com Blocos

A esta altura você deve se sentir bastante confortável com os méritos de criar escopos para limitar a exposição de identificadores.

Até aqui, vimos como fazer isso via escopo de `function` (ou seja, IIFE). Mas vamos agora considerar usar declarações `let` com blocos aninhados. Em geral, qualquer par de chaves `{ .. }` que seja uma instrução vai agir como um bloco, mas **não necessariamente** como um escopo.

Um bloco só se torna um escopo se necessário, para conter suas declarações com escopo de bloco (ou seja, `let` ou `const`). Considere:

```js
{
    // não necessariamente um escopo (ainda)

    // ..

    // agora sabemos que o bloco precisa ser um escopo
    let thisIsNowAScope = true;

    for (let i = 0; i < 5; i++) {
        // isto também é um escopo, ativado a cada
        // iteração
        if (i % 2 == 0) {
            // isto é apenas um bloco, não um escopo
            console.log(i);
        }
    }
}
// 0 2 4
```

Nem todos os pares de chaves `{ .. }` criam blocos (e, portanto, são elegíveis a se tornarem escopos):

* Literais de objeto usam pares de chaves `{ .. }` para delimitar suas listas de chave-valor, mas esses valores de objeto **não** são escopos.

* `class` usa chaves `{ .. }` em torno da definição do seu corpo, mas isso não é um bloco nem um escopo.

* Uma `function` usa `{ .. }` em torno do seu corpo, mas isso não é tecnicamente um bloco — é uma única instrução para o corpo da função. Isso *é*, porém, um escopo (de função).

* O par de chaves `{ .. }` em uma instrução `switch` (em torno do conjunto de cláusulas `case`) não define um bloco/escopo.

Fora esses exemplos de não bloco, um par de chaves `{ .. }` pode definir um bloco anexado a uma instrução (como um `if` ou um `for`), ou existir sozinho — veja o par de chaves `{ .. }` mais externo no trecho anterior. Um bloco explícito desse tipo — se não tiver declarações, não é de fato um escopo — não serve a nenhum propósito operacional, embora ainda possa ser útil como sinal semântico.

Blocos `{ .. }` explícitos e independentes sempre foram sintaxe JS válida, mas, como não podiam ser um escopo antes do `let`/`const` do ES6, são bastante raros. Porém, depois do ES6, eles estão começando a pegar um pouco.

Na maioria das linguagens que suportam escopo de bloco, um escopo de bloco explícito é um padrão extremamente comum para criar uma fatia estreita de escopo para uma ou algumas variáveis. Então, seguindo o princípio POLE, deveríamos abraçar esse padrão de forma mais difundida também em JS; use escopo de bloco (explícito) para estreitar a exposição de identificadores ao mínimo prático.

Um escopo de bloco explícito pode ser útil mesmo dentro de outro bloco (seja o bloco externo um escopo ou não).

Por exemplo:

```js
if (somethingHappened) {
    // isto é um bloco, mas não um escopo

    {
        // isto é tanto um bloco quanto um
        // escopo explícito
        let msg = somethingHappened.message();
        notifyOthers(msg);
    }

    // ..

    recoverFromSomething();
}
```

Aqui, o par de chaves `{ .. }` **dentro** da instrução `if` é um escopo de bloco explícito interno ainda menor para `msg`, já que essa variável não é necessária no bloco `if` inteiro. A maioria dos desenvolvedores simplesmente daria escopo de bloco a `msg` no bloco `if` e seguiria em frente. E, para ser justo, quando há apenas algumas linhas a considerar, é uma decisão de bom senso discutível. Mas, conforme o código cresce, essas questões de superexposição ficam mais pronunciadas.

Então, isso importa o suficiente para adicionar o par extra de `{ .. }` e um nível de indentação? Eu acho que você deveria seguir o POLE e sempre (dentro do razoável!) definir o menor bloco para cada variável. Por isso recomendo usar o escopo de bloco explícito extra, como mostrado.

Lembre-se da discussão sobre erros de TDZ em "Variáveis Não Inicializadas (TDZ)" (Capítulo 5). Minha sugestão lá foi: para minimizar o risco de erros de TDZ com declarações `let`/`const`, sempre coloque essas declarações no topo do seu escopo.

Se você se pegar colocando uma declaração `let` no meio de um escopo, pense primeiro: "Ah, não! Alerta de TDZ!" Se essa declaração `let` não for necessária na primeira metade daquele bloco, você deveria usar um escopo de bloco explícito interno para estreitar ainda mais sua exposição!

Outro exemplo com escopo de bloco explícito:

```js
function getNextMonthStart(dateStr) {
    var nextMonth, year;

    {
        let curMonth;
        [ , year, curMonth ] = dateStr.match(
                /(\d{4})-(\d{2})-\d{2}/
            ) || [];
        nextMonth = (Number(curMonth) % 12) + 1;
    }

    if (nextMonth == 1) {
        year++;
    }

    return `${ year }-${
            String(nextMonth).padStart(2,"0")
        }-01`;
}
getNextMonthStart("2019-12-25");   // 2020-01-01
```

Vamos primeiro identificar os escopos e seus identificadores:

1. O escopo externo/global tem um identificador, a função `getNextMonthStart(..)`.

2. O escopo de função de `getNextMonthStart(..)` tem três: `dateStr` (parâmetro), `nextMonth` e `year`.

3. O par de chaves `{ .. }` define um escopo de bloco interno que inclui uma variável: `curMonth`.

Então, por que colocar `curMonth` em um escopo de bloco explícito, em vez de simplesmente ao lado de `nextMonth` e `year` no escopo de função de nível superior? Porque `curMonth` só é necessária nas duas primeiras instruções; no nível do escopo de função, ela está superexposta.

Este exemplo é pequeno, então os riscos de superexpor `curMonth` são bem limitados. Mas os benefícios do princípio POLE são mais bem alcançados quando você adota, por padrão e como hábito, a mentalidade de minimizar a exposição de escopo. Se você seguir o princípio de forma consistente mesmo nos casos pequenos, ele vai te servir mais conforme seus programas crescerem.

Vamos agora olhar um exemplo ainda mais substancial:

```js
function sortNamesByLength(names) {
    var buckets = [];

    for (let firstName of names) {
        if (buckets[firstName.length] == null) {
            buckets[firstName.length] = [];
        }
        buckets[firstName.length].push(firstName);
    }

    // um bloco para estreitar o escopo
    {
        let sortedNames = [];

        for (let bucket of buckets) {
            if (bucket) {
                // ordena cada bucket alfanumericamente
                bucket.sort();

                // acrescenta os nomes ordenados à
                // nossa lista corrente
                sortedNames = [
                    ...sortedNames,
                    ...bucket
                ];
            }
        }

        return sortedNames;
    }
}

sortNamesByLength([
    "Sally",
    "Suzy",
    "Frank",
    "John",
    "Jennifer",
    "Scott"
]);
// [ "John", "Suzy", "Frank", "Sally",
//   "Scott", "Jennifer" ]
```

Há seis identificadores declarados em cinco escopos diferentes. Todas essas variáveis poderiam ter existido no único escopo externo/global? Tecnicamente, sim, já que todas têm nomes únicos e, portanto, não há colisões de nome. Mas essa seria uma organização de código realmente ruim e provavelmente levaria tanto a confusão quanto a bugs futuros.

Nós as separamos em cada escopo aninhado interno conforme apropriado. Cada variável é definida no escopo mais interno possível para que o programa opere como desejado.

`sortedNames` poderia ter sido definida no escopo de função de nível superior, mas ela só é necessária na segunda metade desta função. Para evitar superexpor essa variável em um escopo de nível mais alto, seguimos novamente o POLE e damos a ela escopo de bloco no escopo de bloco explícito interno.

### `var` *e* `let`

Em seguida, vamos falar da declaração `var buckets`. Essa variável é usada em toda a função (exceto na instrução `return` final). Qualquer variável necessária em toda (ou até na maior parte de uma) função deveria ser declarada de modo que esse uso fique óbvio.

| NOTA: |
| :--- |
| O parâmetro `names` não é usado na função inteira, mas não há como limitar o escopo de um parâmetro, então ele se comporta como uma declaração de função inteira de qualquer forma. |

Então, por que usamos `var` em vez de `let` para declarar a variável `buckets`? Há razões tanto semânticas quanto técnicas para escolher `var` aqui.

Estilisticamente, `var` sempre sinalizou, desde os primeiros dias do JS, "variável que pertence a uma função inteira". Como afirmamos em "Escopo Léxico" (Capítulo 1), `var` se anexa ao escopo de função envolvente mais próximo, não importa onde apareça. Isso é verdade mesmo se `var` aparece dentro de um bloco:

```js
function diff(x,y) {
    if (x > y) {
        var tmp = x;    // `tmp` tem escopo de função
        x = y;
        y = tmp;
    }

    return y - x;
}
```

Mesmo que `var` esteja dentro de um bloco, sua declaração tem escopo de função (de `diff(..)`), não escopo de bloco.

Embora você possa declarar `var` dentro de um bloco (e ainda assim ele ter escopo de função), eu recomendaria evitar essa abordagem, exceto em alguns casos específicos (discutidos no Apêndice A). Fora isso, `var` deveria ser reservado para uso no escopo de nível superior de uma função.

Por que não simplesmente usar `let` nesse mesmo lugar? Porque `var` é visualmente distinto de `let` e, portanto, sinaliza claramente: "esta variável tem escopo de função". Usar `let` no escopo de nível superior, especialmente se não estiver nas primeiras linhas de uma função, e quando todas as outras declarações em blocos usam `let`, não chama visualmente a atenção para a diferença em relação à declaração com escopo de função.

Em outras palavras, eu sinto que `var` comunica melhor "escopo de função" do que `let`, e `let` tanto comunica quanto alcança o escopo de bloco, onde `var` é insuficiente. Enquanto seus programas precisarem tanto de variáveis com escopo de função quanto com escopo de bloco, a abordagem mais sensata e legível é usar tanto `var` *quanto* `let` juntos, cada um para seu melhor propósito.

Há outras razões semânticas e operacionais para escolher `var` ou `let` em cenários diferentes. Vamos explorar o caso do `var` *e* do `let` com mais detalhes no Apêndice A.

| AVISO: |
| :--- |
| Minha recomendação de usar tanto `var` *quanto* `let` é claramente controversa e contradiz a maioria. É muito mais comum ouvir afirmações como "var é quebrado, let conserta isso" e "nunca use var, let é o substituto". Essas opiniões são válidas, mas são meramente opiniões, assim como a minha. `var` não é, de fato, quebrado nem obsoleto; funciona desde os primórdios do JS e vai continuar funcionando enquanto o JS existir. |

### Onde Usar `let`?

Meu conselho de reservar `var` (majoritariamente) apenas para um escopo de função de nível superior significa que a maior parte das outras declarações deveria usar `let`. Mas você ainda pode estar se perguntando como decidir onde cada declaração do seu programa pertence.

O POLE já te guia nessas decisões, mas vamos deixá-lo explícito. A forma de decidir não se baseia em qual palavra-chave você quer usar. A forma de decidir é perguntar: "Qual é a exposição de escopo mais mínima que é suficiente para esta variável?"

Uma vez respondido isso, você vai saber se uma variável pertence a um escopo de bloco ou ao escopo de função. Se você decidir inicialmente que uma variável deve ter escopo de bloco e depois perceber que ela precisa ser elevada para escopo de função, isso dita uma mudança não apenas no local da declaração dessa variável, mas também na palavra-chave declaradora usada. O processo de tomada de decisão realmente deveria se dar assim.

Se uma declaração pertence a um escopo de bloco, use `let`. Se pertence ao escopo de função, use `var` (de novo, apenas minha opinião).

Mas outra forma de meio que visualizar essa tomada de decisão é considerar a versão pré-ES6 de um programa. Por exemplo, vamos relembrar `diff(..)` de antes:

```js
function diff(x,y) {
    var tmp;

    if (x > y) {
        tmp = x;
        x = y;
        y = tmp;
    }

    return y - x;
}
```

Nesta versão de `diff(..)`, `tmp` é claramente declarada no escopo de função. Isso é apropriado para `tmp`? Eu argumentaria que não. `tmp` só é necessária para aquelas poucas instruções. Não é necessária para a instrução `return`. Portanto, deveria ter escopo de bloco.

Antes do ES6, não tínhamos `let`, então não podíamos *de fato* dar escopo de bloco a ela. Mas podíamos fazer a segunda melhor coisa ao sinalizar nossa intenção:

```js
function diff(x,y) {
    if (x > y) {
        // `tmp` ainda tem escopo de função, mas
        // o posicionamento aqui sinaliza
        // semanticamente escopo de bloco
        var tmp = x;
        x = y;
        y = tmp;
    }

    return y - x;
}
```

Colocar a declaração `var` de `tmp` dentro da instrução `if` sinaliza a quem lê o código que `tmp` pertence àquele bloco. Mesmo que o JS não force esse escopo, o sinal semântico ainda traz benefício a quem lê seu código.

Seguindo essa perspectiva, você pode encontrar qualquer `var` que esteja dentro de um bloco desse tipo e trocá-lo por `let`, para fazer valer o sinal semântico que já está sendo enviado. Esse é, na minha opinião, o uso apropriado do `let`.

Outro exemplo que historicamente se baseava em `var`, mas que hoje deveria praticamente sempre usar `let`, é o laço `for`:

```js
for (var i = 0; i < 5; i++) {
    // faz alguma coisa
}
```

Não importa onde esse laço seja definido, o `i` basicamente sempre deveria ser usado apenas dentro do laço; nesse caso, o POLE dita que ele deveria ser declarado com `let` em vez de `var`:

```js
for (let i = 0; i < 5; i++) {
    // faz alguma coisa
}
```

Quase o único caso em que trocar um `var` por um `let` dessa forma "quebraria" seu código é se você estivesse dependendo de acessar o iterador do laço (`i`) fora/depois do laço, como:

```js
for (var i = 0; i < 5; i++) {
    if (checkValue(i)) {
        break;
    }
}

if (i < 5) {
    console.log("The loop stopped early!");
}
```

Esse padrão de uso não é terrivelmente incomum, mas a maioria sente que ele cheira a má estrutura de código. Uma abordagem preferível é usar outra variável de escopo externo para esse propósito:

```js
var lastI;

for (let i = 0; i < 5; i++) {
    lastI = i;
    if (checkValue(i)) {
        break;
    }
}

if (lastI < 5) {
    console.log("The loop stopped early!");
}
```

`lastI` é necessária em todo esse escopo, então é declarada com `var`. `i` só é necessária em (cada) iteração do laço, então é declarada com `let`.

### Qual é a do Catch?

Até aqui afirmamos que `var` e parâmetros têm escopo de função, e que `let`/`const` sinalizam declarações com escopo de bloco. Há uma pequena exceção a apontar: a cláusula `catch`.

Desde a introdução do `try..catch` lá no ES3 (em 1999), a cláusula `catch` usa uma capacidade adicional (pouco conhecida) de declaração com escopo de bloco:

```js
try {
    doesntExist();
}
catch (err) {
    console.log(err);
    // ReferenceError: 'doesntExist' is not defined
    // ^^^^ mensagem impressa a partir da exceção capturada

    let onlyHere = true;
    var outerVariable = true;
}

console.log(outerVariable);     // true

console.log(err);
// ReferenceError: 'err' is not defined
// ^^^^ esta é outra exceção lançada (não capturada)
```

A variável `err` declarada pela cláusula `catch` tem escopo de bloco limitado àquele bloco. Esse bloco da cláusula `catch` pode conter outras declarações com escopo de bloco via `let`. Mas uma declaração `var` dentro desse bloco ainda se anexa ao escopo de função/global externo.

O ES2019 (recentemente, no momento em que escrevo) mudou as cláusulas `catch` para que sua declaração seja opcional; se a declaração é omitida, o bloco `catch` deixa de ser (por padrão) um escopo; ainda é um bloco, porém!

Então, se você precisa reagir à condição *de que uma exceção ocorreu* (para poder se recuperar elegantemente), mas não se importa com o valor do erro em si, pode omitir a declaração do `catch`:

```js
try {
    doOptionOne();
}
catch {   // declaração do catch omitida
    doOptionTwoInstead();
}
```

Esta é uma pequena, mas deliciosa simplificação de sintaxe para um caso de uso bastante comum, e pode até ser ligeiramente mais performática ao remover um escopo desnecessário!

## Declarações de Função em Blocos (FiB)

Vimos agora que declarações usando `let` ou `const` têm escopo de bloco, e que declarações `var` têm escopo de função. Então, e quanto a declarações de `function` que aparecem diretamente dentro de blocos? Como recurso, isso se chama "FiB" (*Function in Block*).

Tipicamente pensamos em declarações de `function` como se fossem o equivalente de uma declaração `var`. Então elas têm escopo de função, como `var`?

Não e sim. Eu sei... isso é confuso. Vamos nos aprofundar:

```js
if (false) {
    function ask() {
        console.log("Does this run?");
    }
}
ask();
```

O que você espera que este programa faça? Três resultados razoáveis:

1. A chamada `ask()` pode falhar com uma exceção `ReferenceError`, porque o identificador `ask` tem escopo de bloco limitado ao escopo do bloco `if` e, portanto, não está disponível no escopo externo/global.

2. A chamada `ask()` pode falhar com uma exceção `TypeError`, porque o identificador `ask` existe, mas está `undefined` (já que a instrução `if` não roda) e, portanto, não é uma função chamável.

3. A chamada `ask()` pode rodar corretamente, imprimindo a mensagem "Does it run?".

Aqui está a parte confusa: dependendo do ambiente JS em que você testar esse trecho de código, pode obter resultados diferentes! Esta é uma daquelas poucas áreas malucas em que o comportamento legado existente trai um resultado previsível.

A especificação do JS diz que declarações de `function` dentro de blocos têm escopo de bloco, então a resposta deveria ser (1). Porém, a maioria das engines JS baseadas em navegador (incluindo a v8, que vem do Chrome mas também é usada no Node) vai se comportar como (2), ou seja, o identificador tem escopo fora do bloco `if`, mas o valor da função não é automaticamente inicializado, então permanece `undefined`.

Por que engines JS de navegador podem se comportar de forma contrária à especificação? Porque essas engines já tinham certos comportamentos em torno de FiB antes de o ES6 introduzir escopo de bloco, e havia preocupação de que mudar para aderir à especificação pudesse quebrar código JS de sites existentes. Assim, foi feita uma exceção no Apêndice B da especificação do JS, que permite certos desvios para engines JS de navegador (apenas!).

| NOTA: |
| :--- |
| Você tipicamente não categorizaria o Node como um ambiente JS de navegador, já que ele geralmente roda em um servidor. Mas a engine v8 do Node é compartilhada com os navegadores Chrome (e Edge). Como a v8 é, antes de tudo, uma engine JS de navegador, ela adota essa exceção do Apêndice B, o que então significa que as exceções de navegador se estendem ao Node. |

Um dos casos de uso mais comuns para colocar uma declaração de `function` em um bloco é definir condicionalmente uma função de um jeito ou de outro (como com uma instrução `if..else`), dependendo de algum estado do ambiente. Por exemplo:

```js
if (typeof Array.isArray != "undefined") {
    function isArray(a) {
        return Array.isArray(a);
    }
}
else {
    function isArray(a) {
        return Object.prototype.toString.call(a)
            == "[object Array]";
    }
}
```

É tentador estruturar o código assim por razões de desempenho, já que a verificação `typeof Array.isArray` só é realizada uma vez, em oposição a definir apenas um `isArray(..)` e colocar a instrução `if` dentro dele — a verificação rodaria então desnecessariamente a cada chamada.

| AVISO: |
| :--- |
| Além dos riscos dos desvios de FiB, outro problema com a definição condicional de funções é que fica mais difícil depurar tal programa. Se você acabar com um bug na função `isArray(..)`, primeiro precisa descobrir *qual* implementação de `isArray(..)` está de fato rodando! Às vezes, o bug é que a errada foi aplicada porque a verificação condicional estava incorreta! Se você define múltiplas versões de uma função, esse programa é sempre mais difícil de raciocinar e manter. |

Além dos trechos anteriores, vários outros casos de borda de FiB estão à espreita; tais comportamentos em diferentes navegadores e ambientes JS não baseados em navegador provavelmente vão variar. Por exemplo:

```js
if (true) {
    function ask() {
        console.log("Am I called?");
    }
}

if (true) {
    function ask() {
        console.log("Or what about me?");
    }
}

for (let i = 0; i < 5; i++) {
    function ask() {
        console.log("Or is it one of these?");
    }
}

ask();

function ask() {
    console.log("Wait, maybe, it's this one?");
}
```

Lembre-se de que o function hoisting, como descrito em "Quando Posso Usar uma Variável?" (no Capítulo 5), poderia sugerir que o `ask()` final deste trecho, com "Wait, maybe..." como mensagem, sofreria hoisting acima da chamada a `ask()`. Como é a última declaração de função com esse nome, ela deveria "vencer", certo? Infelizmente, não.

Não é minha intenção documentar todos esses casos de borda esquisitos, nem tentar explicar por que cada um se comporta de determinada forma. Essa informação é, na minha opinião, trivialidade legada arcana.

Minha real preocupação com FiB é: que conselho eu posso dar para garantir que seu código se comporte de forma previsível em todas as circunstâncias?

No que me diz respeito, a única resposta prática para evitar os caprichos do FiB é simplesmente evitar FiB por completo. Em outras palavras, nunca coloque uma declaração de `function` diretamente dentro de qualquer bloco. Sempre coloque declarações de `function` em algum lugar do escopo de nível superior de uma função (ou no escopo global).

Então, para o exemplo anterior com `if..else`, minha sugestão é evitar definir funções condicionalmente, se possível. Sim, pode ser ligeiramente menos performático, mas esta é a melhor abordagem no geral:

```js
function isArray(a) {
    if (typeof Array.isArray != "undefined") {
        return Array.isArray(a);
    }
    else {
        return Object.prototype.toString.call(a)
            == "[object Array]";
    }
}
```

Se esse custo de desempenho se tornar uma questão de caminho crítico para sua aplicação, sugiro que você considere esta abordagem:

```js
var isArray = function isArray(a) {
    return Array.isArray(a);
};

// sobrescreve a definição, se for necessário
if (typeof Array.isArray == "undefined") {
    isArray = function isArray(a) {
        return Object.prototype.toString.call(a)
            == "[object Array]";
    };
}
```

É importante notar que aqui estou colocando uma **expressão** de `function`, e não uma declaração, dentro da instrução `if`. Isso é perfeitamente aceitável e válido: expressões de `function` podem aparecer dentro de blocos. Nossa discussão sobre FiB é sobre evitar **declarações** de `function` em blocos.

Mesmo que você teste seu programa e ele funcione corretamente, o pequeno benefício que você pode obter usando o estilo FiB no seu código é de longe superado pelos riscos potenciais, no futuro, de confusão por parte de outros desenvolvedores, ou de variações em como seu código roda em outros ambientes JS.

FiB não vale a pena e deve ser evitado.

## Bloqueado

O propósito das regras de escopo léxico em uma linguagem de programação é permitir que organizemos adequadamente as variáveis do nosso programa, tanto por razões operacionais quanto por razões semânticas de comunicação do código.

E uma das técnicas organizacionais mais importantes é garantir que nenhuma variável fique superexposta a escopos desnecessários (POLE). Espero que agora você aprecie o escopo de bloco muito mais profundamente do que antes.

Espero que, a esta altura, você sinta que está pisando em um terreno muito mais sólido no entendimento do escopo léxico. A partir dessa base, o próximo capítulo mergulha no pesado tópico da closure.

[^POLP]: *Principle of Least Privilege*, https://en.wikipedia.org/wiki/Principle_of_least_privilege, 3 de março de 2020.
