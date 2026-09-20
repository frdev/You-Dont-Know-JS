# You Don't Know JS Yet: Scope & Closures - 2ª Edição
# Capítulo 8: O Padrão de Módulo

Neste capítulo, encerramos o texto principal do livro explorando um dos padrões de organização de código mais importantes de toda a programação: o módulo. Como veremos, módulos são inerentemente construídos a partir do que já cobrimos: a recompensa pelos seus esforços em aprender escopo léxico e closure.

Examinamos todos os ângulos do escopo léxico, da amplitude do escopo global até os escopos de bloco aninhados, passando pelas minúcias do ciclo de vida das variáveis. Depois aproveitamos o escopo léxico para entender todo o poder da closure.

Tire um momento para refletir sobre o quanto você avançou nesta jornada até aqui; você deu grandes passos para conhecer o JS mais profundamente!

O tema central deste livro tem sido que entender e dominar escopo e closure é fundamental para estruturar e organizar nosso código adequadamente, especialmente nas decisões sobre onde armazenar informação em variáveis.

Nosso objetivo neste capítulo final é apreciar como os módulos encarnam a importância desses tópicos, elevando-os de conceitos abstratos a melhorias concretas e práticas na construção de programas.

## Encapsulamento e Exposição Mínima (POLE)

O encapsulamento é frequentemente citado como um princípio da programação orientada a objetos (OO), mas é mais fundamental e amplamente aplicável do que isso. O objetivo do encapsulamento é o agrupamento ou a colocalização de informação (dados) e comportamento (funções) que, juntos, servem a um propósito comum.

Independentemente de qualquer sintaxe ou mecanismo de código, o espírito do encapsulamento pode ser realizado em algo tão simples quanto usar arquivos separados para conter pedaços do programa com propósito comum. Se agrupamos tudo que alimenta uma lista de resultados de busca em um único arquivo chamado "search-list.js", estamos encapsulando aquela parte do programa.

A tendência recente na programação front-end moderna de organizar aplicações em torno de uma arquitetura de Componentes leva o encapsulamento ainda mais longe. Para muitos, parece natural consolidar tudo que constitui a lista de resultados de busca — indo além do código, incluindo marcação de apresentação e estilização — em uma única unidade de lógica de programa, algo tangível com o qual possamos interagir. E então rotulamos esse conjunto como o componente "SearchList".

Outro objetivo fundamental é o controle da visibilidade de certos aspectos dos dados e funcionalidades encapsulados. Lembre-se, do Capítulo 6, do princípio da *exposição mínima* (POLE), que busca defender contra vários *perigos* da superexposição de escopo; esses perigos afetam tanto variáveis quanto funções. Em JS, na maioria das vezes implementamos o controle de visibilidade por meio da mecânica do escopo léxico.

A ideia é agrupar pedaços semelhantes do programa e limitar seletivamente o acesso programático às partes que consideramos detalhes *privados*. O que não é considerado *privado* é então marcado como *público*, acessível a todo o programa.

O efeito natural desse esforço é uma melhor organização do código. É mais fácil construir e manter software quando sabemos onde as coisas estão, com limites e pontos de conexão claros e óbvios. Também é mais fácil manter a qualidade se evitarmos as armadilhas de dados e funcionalidades superexpostos.

Esses são alguns dos principais benefícios de organizar programas JS em módulos.

## O que é um Módulo?

Um módulo é uma coleção de dados e funções relacionados (frequentemente chamadas de métodos neste contexto), caracterizada por uma divisão entre detalhes *privados* ocultos e detalhes *públicos* acessíveis, geralmente chamados de "API pública".

Um módulo também tem estado: ele mantém alguma informação ao longo do tempo, junto com funcionalidades para acessar e atualizar essa informação.

| NOTA: |
| :--- |
| Uma preocupação mais ampla do padrão de módulo é abraçar plenamente a modularização em nível de sistema, por meio de acoplamento fraco e outras técnicas de arquitetura de programas. Esse é um tópico complexo, bem além dos limites da nossa discussão, mas que vale estudo adicional para além deste livro. |

Para ter uma noção melhor do que é um módulo, vamos comparar algumas características de módulos com padrões de código úteis que não são exatamente módulos.

### Namespaces (Agrupamento Sem Estado)

Se você agrupa um conjunto de funções relacionadas, sem dados, então você não tem realmente o encapsulamento esperado que um módulo implica. O termo melhor para esse agrupamento de funções *sem estado* é namespace:

```js
// namespace, não módulo
var Utils = {
    cancelEvt(evt) {
        evt.preventDefault();
        evt.stopPropagation();
        evt.stopImmediatePropagation();
    },
    wait(ms) {
        return new Promise(function c(res){
            setTimeout(res,ms);
        });
    },
    isValidEmail(email) {
        return /[^@]+@[^@.]+\.[^@.]+/.test(email);
    }
};
```

`Utils` aqui é uma coleção útil de utilitários, mas todos são funções independentes de estado. Reunir funcionalidades é, em geral, uma boa prática, mas isso não faz disto um módulo. Em vez disso, definimos um namespace `Utils` e organizamos as funções sob ele.

### Estruturas de Dados (Agrupamento Com Estado)

Mesmo que você agrupe dados e funções com estado, se não estiver limitando a visibilidade de nada disso, então você está parando antes do aspecto POLE do encapsulamento; não é particularmente útil rotular isso como um módulo.

Considere:

```js
// estrutura de dados, não módulo
var Student = {
    records: [
        { id: 14, name: "Kyle", grade: 86 },
        { id: 73, name: "Suzy", grade: 87 },
        { id: 112, name: "Frank", grade: 75 },
        { id: 6, name: "Sarah", grade: 91 }
    ],
    getName(studentID) {
        var student = this.records.find(
            student => student.id == studentID
        );
        return student.name;
    }
};

Student.getName(73);
// Suzy
```

Como `records` é um dado publicamente acessível, e não escondido atrás de uma API pública, `Student` aqui não é realmente um módulo.

`Student` tem, sim, o aspecto de dados-e-funcionalidade do encapsulamento, mas não o aspecto de controle de visibilidade. É melhor rotular isso como uma instância de uma estrutura de dados.

### Módulos (Controle de Acesso Com Estado)

Para encarnar todo o espírito do padrão de módulo, precisamos não só de agrupamento e estado, mas também de controle de acesso por visibilidade (privado vs. público).

Vamos transformar o `Student` da seção anterior em um módulo. Vamos começar com uma forma que chamo de "módulo clássico", que era originalmente chamada de "revealing module" quando surgiu no início dos anos 2000. Considere:

```js
var Student = (function defineStudent(){
    var records = [
        { id: 14, name: "Kyle", grade: 86 },
        { id: 73, name: "Suzy", grade: 87 },
        { id: 112, name: "Frank", grade: 75 },
        { id: 6, name: "Sarah", grade: 91 }
    ];

    var publicAPI = {
        getName
    };

    return publicAPI;

    // ************************

    function getName(studentID) {
        var student = records.find(
            student => student.id == studentID
        );
        return student.name;
    }
})();

Student.getName(73);   // Suzy
```

`Student` é agora uma instância de um módulo. Ele apresenta uma API pública com um único método: `getName(..)`. Esse método consegue acessar os dados privados e ocultos de `records`.

| AVISO: |
| :--- |
| Devo apontar que os dados explícitos de alunos, escritos diretamente nesta definição de módulo, existem apenas para fins de ilustração. Um módulo típico no seu programa vai receber esses dados de uma fonte externa, tipicamente carregados de bancos de dados, arquivos de dados JSON, chamadas Ajax etc. Os dados são então injetados na instância do módulo, tipicamente por meio de método(s) da API pública do módulo. |

Como o formato de módulo clássico funciona?

Note que a instância do módulo é criada pela execução da IIFE `defineStudent()`. Essa IIFE retorna um objeto (chamado `publicAPI`) que tem uma propriedade referenciando a função interna `getName(..)`.

Nomear o objeto como `publicAPI` é preferência estilística minha. O objeto pode ter o nome que você quiser (o JS não se importa), ou você pode simplesmente retornar um objeto diretamente, sem atribuí-lo a nenhuma variável interna nomeada. Mais sobre essa escolha no Apêndice A.

De fora, `Student.getName(..)` invoca essa função interna exposta, que mantém acesso à variável interna `records` via closure.

Você não *precisa* retornar um objeto com uma função como uma de suas propriedades. Você poderia simplesmente retornar uma função diretamente, no lugar do objeto. Isso ainda satisfaz todas as partes centrais de um módulo clássico.

Em virtude de como o escopo léxico funciona, definir variáveis e funções dentro da sua função externa de definição de módulo torna tudo *privado por padrão*. Apenas propriedades adicionadas ao objeto de API pública retornado pela função serão exportadas para uso público externo.

O uso de uma IIFE implica que nosso programa só precisa de uma única instância central do módulo, comumente chamada de "singleton". De fato, este exemplo específico é simples o bastante para que não haja razão óbvia para precisarmos de mais do que apenas uma instância do módulo `Student`.

#### Fábrica de Módulos (Múltiplas Instâncias)

Mas, se quiséssemos definir um módulo que suportasse múltiplas instâncias no nosso programa, podemos ajustar ligeiramente o código:

```js
// função fábrica, não IIFE singleton
function defineStudent() {
    var records = [
        { id: 14, name: "Kyle", grade: 86 },
        { id: 73, name: "Suzy", grade: 87 },
        { id: 112, name: "Frank", grade: 75 },
        { id: 6, name: "Sarah", grade: 91 }
    ];

    var publicAPI = {
        getName
    };

    return publicAPI;

    // ************************

    function getName(studentID) {
        var student = records.find(
            student => student.id == studentID
        );
        return student.name;
    }
}

var fullTime = defineStudent();
fullTime.getName(73);            // Suzy
```

Em vez de especificar `defineStudent()` como uma IIFE, simplesmente a definimos como uma função independente normal, que neste contexto é comumente chamada de função "fábrica de módulo".

Então chamamos a fábrica de módulo, produzindo uma instância do módulo que rotulamos como `fullTime`. Essa instância de módulo implica uma nova instância do escopo interno e, portanto, uma nova closure que `getName(..)` mantém sobre `records`. `fullTime.getName(..)` agora invoca o método daquela instância específica.

#### Definição de Módulo Clássico

Então, para esclarecer o que faz de algo um módulo clássico:

* Precisa haver um escopo externo, tipicamente de uma função fábrica de módulo executada ao menos uma vez.

* O escopo interno do módulo precisa ter ao menos uma parcela de informação oculta que represente estado para o módulo.

* O módulo precisa retornar, na sua API pública, uma referência a pelo menos uma função que tenha closure sobre o estado oculto do módulo (para que esse estado seja, de fato, preservado).

Você provavelmente vai se deparar com outras variações dessa abordagem de módulo clássico, que veremos com mais detalhes no Apêndice A.

## Módulos CommonJS do Node

No Capítulo 4, apresentamos o formato de módulo CommonJS usado pelo Node. Diferente do formato de módulo clássico descrito antes, em que você poderia empacotar a fábrica de módulo ou a IIFE junto de qualquer outro código, incluindo outros módulos, os módulos CommonJS são baseados em arquivo; um módulo por arquivo.

Vamos ajustar nosso exemplo de módulo para aderir a esse formato:

```js
module.exports.getName = getName;

// ************************

var records = [
    { id: 14, name: "Kyle", grade: 86 },
    { id: 73, name: "Suzy", grade: 87 },
    { id: 112, name: "Frank", grade: 75 },
    { id: 6, name: "Sarah", grade: 91 }
];

function getName(studentID) {
    var student = records.find(
        student => student.id == studentID
    );
    return student.name;
}
```

Os identificadores `records` e `getName` estão no escopo de nível superior deste módulo, mas isso não é o escopo global (como explicado no Capítulo 4). Assim, tudo aqui é, *por padrão*, privado ao módulo.

Para expor algo na API pública de um módulo CommonJS, você adiciona uma propriedade ao objeto vazio fornecido como `module.exports`. Em algum código legado mais antigo, você pode encontrar referências a apenas um `exports` sozinho, mas, por clareza de código, você deveria sempre qualificar completamente essa referência com o prefixo `module.`.

Por questões de estilo, gosto de colocar meus "exports" no topo e a implementação do módulo embaixo. Mas esses exports podem ser colocados em qualquer lugar. Recomendo fortemente reuni-los todos juntos, seja no topo, seja no fim do seu arquivo.

Alguns desenvolvedores têm o hábito de substituir o objeto exports padrão, assim:

```js
// definindo um novo objeto para a API
module.exports = {
    // ..exports..
};
```

Há algumas esquisitices com essa abordagem, incluindo comportamento inesperado se múltiplos módulos desse tipo dependerem circularmente uns dos outros. Por isso, recomendo não substituir o objeto. Se você quer atribuir múltiplos exports de uma vez, usando definição em estilo de literal de objeto, pode fazer isto:

```js
Object.assign(module.exports,{
   // .. exports ..
});
```

O que está acontecendo aqui é a definição do literal de objeto `{ .. }` com a API pública do seu módulo especificada, e então `Object.assign(..)` realiza uma cópia rasa de todas essas propriedades para o objeto `module.exports` existente, em vez de substituí-lo. Esse é um bom equilíbrio entre conveniência e comportamento de módulo mais seguro.

Para incluir outra instância de módulo no seu módulo/programa, use o método `require(..)` do Node. Supondo que este módulo esteja em "/path/to/student.js", é assim que podemos acessá-lo:

```js
var Student = require("/path/to/student.js");

Student.getName(73);
// Suzy
```

`Student` agora referencia a API pública do nosso módulo de exemplo.

Módulos CommonJS se comportam como instâncias singleton, de forma parecida com o estilo de definição de módulo por IIFE apresentado antes. Não importa quantas vezes você faça `require(..)` do mesmo módulo, você só obtém referências adicionais à única instância compartilhada do módulo.

`require(..)` é um mecanismo de tudo ou nada; ele inclui uma referência a toda a API pública exposta do módulo. Para acessar efetivamente apenas parte da API, a abordagem típica se parece com isto:

```js
var getName = require("/path/to/student.js").getName;

// ou, alternativamente:

var { getName } = require("/path/to/student.js");
```

De forma parecida com o formato de módulo clássico, os métodos publicamente exportados da API de um módulo CommonJS mantêm closures sobre os detalhes internos do módulo. É assim que o estado singleton do módulo é mantido ao longo da vida do seu programa.

| NOTA: |
| :--- |
| Em instruções `require("student")` do Node, caminhos não absolutos (`"student"`) presumem uma extensão de arquivo ".js" e buscam em "node_modules". |

## Módulos ES Modernos (ESM)

O formato ESM compartilha várias semelhanças com o formato CommonJS. O ESM é baseado em arquivo, e instâncias de módulo são singletons, com tudo privado *por padrão*. Uma diferença notável é que arquivos ESM são presumidos como estando em modo estrito, sem precisar de um pragma `"use strict"` no topo. Não há como definir um ESM em modo não estrito.

Em vez de `module.exports` do CommonJS, o ESM usa a palavra-chave `export` para expor algo na API pública do módulo. A palavra-chave `import` substitui a instrução `require(..)`. Vamos ajustar "students.js" para usar o formato ESM:

```js
export { getName };

// ************************

var records = [
    { id: 14, name: "Kyle", grade: 86 },
    { id: 73, name: "Suzy", grade: 87 },
    { id: 112, name: "Frank", grade: 75 },
    { id: 6, name: "Sarah", grade: 91 }
];

function getName(studentID) {
    var student = records.find(
        student => student.id == studentID
    );
    return student.name;
}
```

A única mudança aqui é a instrução `export { getName }`. Como antes, instruções `export` podem aparecer em qualquer lugar do arquivo, embora `export` precise estar no escopo de nível superior; não pode estar dentro de nenhum outro bloco ou função.

O ESM oferece bastante variação de como as instruções `export` podem ser especificadas. Por exemplo:

```js
export function getName(studentID) {
    // ..
}
```

Ainda que `export` apareça antes da palavra-chave `function` aqui, essa forma ainda é uma declaração de `function` que por acaso também é exportada. Ou seja, o identificador `getName` sofre *function hoisting* (veja o Capítulo 5), então está disponível em todo o escopo do módulo.

Outra variação permitida:

```js
export default function getName(studentID) {
    // ..
}
```

Este é o chamado "export padrão" (*default export*), que tem semântica diferente dos outros exports. Em essência, um "export padrão" é um atalho para os consumidores do módulo quando fazem `import`, dando a eles uma sintaxe mais concisa quando só precisam desse único membro padrão da API.

Exports que não são `default` são chamados de "exports nomeados".

A palavra-chave `import` — assim como `export`, precisa ser usada apenas no nível superior de um ESM, fora de quaisquer blocos ou funções — também tem várias variações de sintaxe. A primeira é chamada de "import nomeado":

```js
import { getName } from "/path/to/students.js";

getName(73);   // Suzy
```

Como você pode ver, essa forma importa apenas os membros da API pública especificamente nomeados de um módulo (pulando qualquer coisa não nomeada explicitamente), e adiciona esses identificadores ao escopo de nível superior do módulo atual. Esse tipo de import é um estilo familiar para quem está acostumado a imports de pacotes em linguagens como Java.

Múltiplos membros da API podem ser listados dentro do conjunto `{ .. }`, separados por vírgulas. Um import nomeado também pode ser *renomeado* com a palavra-chave `as`:

```js
import { getName as getStudentName }
   from "/path/to/students.js";

getStudentName(73);
// Suzy
```

Se `getName` for um "export padrão" do módulo, podemos importá-lo assim:

```js
import getName from "/path/to/students.js";

getName(73);   // Suzy
```

A única diferença aqui é a remoção das `{ }` em torno do vínculo de import. Se você quiser misturar um import padrão com outros imports nomeados:

```js
import { default as getName, /* .. outros .. */ }
   from "/path/to/students.js";

getName(73);   // Suzy
```

Em contraste, a outra grande variação de `import` é chamada de "import de namespace":

```js
import * as Student from "/path/to/students.js";

Student.getName(73);   // Suzy
```

Como provavelmente é óbvio, o `*` importa tudo que é exportado na API, padrão e nomeados, e guarda tudo sob o único identificador de namespace especificado. Essa abordagem é a que mais se aproxima da forma dos módulos clássicos, presente na maior parte da história do JS.

| NOTA: |
| :--- |
| No momento em que escrevo, navegadores modernos suportam ESM há alguns anos, mas o suporte mais ou menos estável do Node a ESM é bastante recente e vem evoluindo há um bom tempo. A evolução provavelmente vai continuar por mais um ano ou mais; a introdução do ESM ao JS lá no ES6 criou uma série de preocupações desafiadoras de compatibilidade para a interoperação do Node com módulos CommonJS. Consulte a documentação de ESM do Node para todos os detalhes mais recentes: https://nodejs.org/api/esm.html |

## Saindo do Escopo

Quer você use o formato de módulo clássico (navegador ou Node), o formato CommonJS (no Node) ou o formato ESM (navegador ou Node), módulos são uma das formas mais eficazes de estruturar e organizar a funcionalidade e os dados do seu programa.

O padrão de módulo é a conclusão da nossa jornada, neste livro, de aprender como podemos usar as regras do escopo léxico para colocar variáveis e funções em lugares apropriados. O POLE é a postura defensiva de *privado por padrão* que sempre adotamos, garantindo que evitamos a superexposição e interagimos apenas com a mínima superfície de API pública necessária.

E, por baixo dos módulos, a *mágica* de como todo o estado dos nossos módulos é mantido são as closures aproveitando o sistema de escopo léxico.

É isso para o texto principal. Parabéns por essa bela jornada até aqui! Como já disse inúmeras vezes ao longo do livro, é uma ideia muito boa pausar, refletir e praticar o que acabamos de discutir.

Quando você estiver confortável e pronto, confira os apêndices, que se aprofundam em alguns dos cantos destes tópicos e também te desafiam com exercícios de prática para solidificar o que você aprendeu.
