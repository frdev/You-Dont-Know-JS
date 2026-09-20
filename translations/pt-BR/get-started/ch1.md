# You Don't Know JS Yet: Get Started - 2ª Edição
# Capítulo 1: O que JavaScript *É*?

Você não conhece JS, ainda. Nem eu, pelo menos não completamente. Nenhum de nós conhece. Mas todos nós podemos começar a conhecer melhor o JS.

Neste primeiro capítulo do primeiro livro da série *You Don't Know JS Yet* (YDKJSY), vamos dedicar um tempo a construir uma base sobre a qual seguir adiante. Precisamos começar cobrindo uma série de detalhes importantes de bastidores, esclarecendo alguns mitos e equívocos sobre o que a linguagem realmente é (e o que não é!).

Essa é uma visão valiosa sobre a identidade do JS e sobre o processo pelo qual ele é organizado e mantido; todo desenvolvedor JS deveria entendê-la. Se você quer conhecer JS, é assim que se *começa* a dar os primeiros passos nessa jornada.

## Sobre Este Livro

Enfatizo a palavra jornada porque *conhecer JS* não é um destino, é uma direção. Não importa quanto tempo você passe com a linguagem, você sempre vai encontrar algo novo para aprender e entender um pouco melhor. Então não encare este livro como algo a ser atravessado às pressas em busca de uma conquista rápida. Em vez disso, paciência e persistência são o melhor caminho enquanto você dá esses primeiros passos.

Depois deste capítulo de contextualização, o restante do livro apresenta um mapa de alto nível do que você vai encontrar ao se aprofundar e estudar JS com os livros da YDKJSY.

Em particular, o Capítulo 4 identifica três pilares principais em torno dos quais a linguagem JS se organiza: escopo/closures, protótipos/objetos e tipos/coerção. JS é uma linguagem ampla e sofisticada, com muitos recursos e capacidades. Mas todo o JS é fundado sobre esses três pilares.

Tenha em mente que, embora este livro se chame "Get Started" (Comece Agora), ele **não pretende ser um livro introdutório para iniciantes**. A principal tarefa deste livro é te preparar para estudar JS profundamente ao longo do restante da série; ele é escrito presumindo que você já tenha familiaridade com JS, com pelo menos vários meses de experiência, antes de seguir adiante na YDKJSY. Então, para extrair o máximo de *Get Started*, dedique bastante tempo a escrever código JS e acumular experiência.

Mesmo que você já tenha escrito muito JS antes, este livro não deve ser lido por alto nem pulado; tire seu tempo para processar plenamente o material daqui. **Um bom começo sempre depende de um primeiro passo sólido.**

## Qual a História Desse Nome?

O nome JavaScript é provavelmente o nome de linguagem de programação mais mal interpretado e mal compreendido de todos.

Essa linguagem tem relação com Java? É apenas a forma "script" do Java? Serve só para escrever scripts, e não programas de verdade?

A verdade é que o nome JavaScript é um artefato de artimanhas de marketing. Quando Brendan Eich concebeu a linguagem, ele lhe deu o codinome Mocha. Internamente, na Netscape, usava-se a marca LiveScript. Mas, quando chegou a hora de dar um nome público à linguagem, "JavaScript" venceu a votação.

Por quê? Porque essa linguagem foi originalmente projetada para atrair um público majoritariamente de programadores Java, e porque a palavra "script" era popular na época para se referir a programas leves. Esses "scripts" leves seriam os primeiros a serem embutidos dentro de páginas dessa coisa nova chamada web!

Em outras palavras, JavaScript foi uma jogada de marketing para tentar posicionar a linguagem como uma alternativa palatável a escrever o Java mais pesado e mais conhecido da época. Poderia facilmente ter sido chamada de "WebJava", aliás.

Existem algumas semelhanças superficiais entre o código JavaScript e o código Java. Essas semelhanças não vêm propriamente de um desenvolvimento compartilhado, mas do fato de ambas as linguagens mirarem desenvolvedores com expectativas de sintaxe herdadas de C (e, em certa medida, de C++).

Por exemplo, usamos `{` para iniciar um bloco de código e `}` para encerrá-lo, exatamente como em C/C++ e Java. Também usamos `;` para pontuar o fim de uma instrução.

De certa forma, as relações jurídicas vão ainda mais fundo do que a sintaxe. A Oracle (via Sun), empresa que ainda detém e administra o Java, também detém a marca registrada oficial do nome "JavaScript" (via Netscape). Essa marca quase nunca é reivindicada e, provavelmente, não poderia ser a esta altura.

Por essas razões, alguns sugeriram que usássemos JS em vez de JavaScript. Essa é uma abreviação muito comum, ainda que não seja uma boa candidata a marca oficial da linguagem. De fato, estes livros usam JS quase exclusivamente para se referir à linguagem.

Distanciando ainda mais a linguagem da marca pertencente à Oracle, o nome oficial da linguagem, especificado pelo TC39 e formalizado pelo órgão de padrões ECMA, é **ECMAScript**. E, de fato, desde 2016, o nome oficial da linguagem também passou a ser sufixado pelo ano da revisão; no momento em que escrevo, é ECMAScript 2019, ou, abreviando, ES2019.

Em outras palavras, o JavaScript/JS que roda no seu navegador ou no Node.js é *uma* implementação do padrão ES2019.

| NOTA: |
| :--- |
| Não use termos como "JS6" ou "ES8" para se referir à linguagem. Algumas pessoas usam, mas esses termos só servem para perpetuar a confusão. "ES20xx" ou simplesmente "JS" é o que você deve usar. |

Quer você chame de JavaScript, JS, ECMAScript ou ES2019, definitivamente não é uma variante da linguagem Java!

> "Java está para JavaScript assim como carro está para carroça." --Jeremy Keith, 2009

## Especificação da Linguagem

Mencionei o TC39, o comitê técnico diretor que administra o JS. Sua principal tarefa é administrar a especificação oficial da linguagem. Eles se reúnem regularmente para votar quaisquer mudanças acordadas, que depois submetem à ECMA, a organização de padrões.

A sintaxe e o comportamento do JS são definidos na especificação ES.

O ES2019 é a 10ª especificação/revisão numerada principal desde o surgimento do JS em 1995, então, na URL oficial da especificação hospedada pela ECMA, você vai encontrar "10.0":

https://www.ecma-international.org/ecma-262/10.0/

O comitê TC39 é composto por algo entre 50 e cerca de 100 pessoas diferentes, vindas de um amplo recorte de empresas com interesse na web, como fabricantes de navegadores (Mozilla, Google, Apple) e fabricantes de dispositivos (Samsung etc.). Todos os membros do comitê são voluntários, embora muitos deles sejam funcionários dessas empresas e, portanto, possam receber remuneração em parte pelas suas funções no comitê.

O TC39 se reúne, em geral, a cada dois meses, normalmente por cerca de três dias, para revisar o trabalho feito pelos membros desde a última reunião, discutir questões e votar propostas. Os locais das reuniões se alternam entre as empresas membros dispostas a sediá-las.

Todas as propostas do TC39 avançam por um processo de cinco estágios — claro, já que somos programadores, ele é baseado em zero! — do Estágio 0 ao Estágio 4. Você pode ler mais sobre o processo de estágios aqui: https://tc39.es/process-document/

Estágio 0 significa, grosso modo, que alguém do TC39 acha que é uma ideia digna e pretende apadrinhá-la e trabalhar nela. Isso significa que muitas ideias que não membros do TC39 "propõem" por meios informais, como redes sociais ou posts de blog, estão, na verdade, em "pré-estágio 0". Você precisa conseguir que um membro do TC39 apadrinhe uma proposta para que ela seja considerada oficialmente "Estágio 0".

Assim que uma proposta alcança o status de "Estágio 4", ela se torna elegível para inclusão na próxima revisão anual da linguagem. Uma proposta pode levar de vários meses a alguns anos para percorrer esses estágios.

Todas as propostas são geridas de forma aberta, no repositório do TC39 no Github: https://github.com/tc39/proposals

Qualquer pessoa, seja do TC39 ou não, é bem-vinda a participar dessas discussões públicas e dos processos de trabalho sobre as propostas. Porém, somente membros do TC39 podem participar das reuniões e votar nas propostas e mudanças. Então, na prática, a voz de um membro do TC39 tem muito peso sobre o rumo do JS.

Ao contrário de um mito estabelecido e frustrantemente perpetuado, *não* existem múltiplas versões de JavaScript soltas por aí. Existe apenas **um JS**, o padrão oficial mantido pelo TC39 e pela ECMA.

No início dos anos 2000, quando a Microsoft mantinha uma versão bifurcada e submetida a engenharia reversa (e nem totalmente compatível) do JS, chamada "JScript", legitimamente havia "múltiplas versões" de JS. Mas esses tempos ficaram para trás. É desatualizado e impreciso fazer esse tipo de afirmação sobre o JS hoje.

Todos os principais fabricantes de navegadores e dispositivos se comprometeram a manter suas implementações de JS em conformidade com essa única especificação central. É claro que as engines implementam recursos em momentos diferentes. Mas nunca deveria acontecer de a engine v8 (a engine JS do Chrome) implementar um recurso especificado de forma diferente ou incompatível em relação à engine SpiderMonkey (a engine JS da Mozilla).

Isso significa que você pode aprender **um JS** e contar com esse mesmo JS em todo lugar.

### A Web Manda em Tudo Sobre (JS)

Embora o leque de ambientes que executam JS esteja em constante expansão (de navegadores a servidores (Node.js), a robôs, a lâmpadas, a...), o ambiente que manda no JS é a web. Em outras palavras, como o JS é implementado para navegadores web é, na prática, a única realidade que importa.

Na maior parte, o JS definido na especificação e o JS que roda nas engines JS dos navegadores são a mesma coisa. Mas há algumas diferenças que precisam ser consideradas.

Às vezes a especificação do JS vai ditar algum comportamento novo ou refinado e, mesmo assim, isso não vai corresponder exatamente a como aquilo funciona nas engines JS dos navegadores. Esse descompasso é histórico: as engines JS têm mais de 20 anos de comportamentos observáveis em casos de borda de recursos, dos quais o conteúdo da web passou a depender. Assim, às vezes as engines JS se recusam a se adequar a uma mudança ditada pela especificação porque isso quebraria esse conteúdo da web.

Nesses casos, frequentemente o TC39 recua e simplesmente escolhe adequar a especificação à realidade da web. Por exemplo, o TC39 planejou adicionar um método `contains(..)` para Arrays, mas descobriu-se que esse nome conflitava com frameworks JS antigos ainda em uso em alguns sites, então mudaram o nome para o não conflitante `includes(..)`. O mesmo aconteceu com uma cômica/trágica *crise da comunidade* JS apelidada de "smooshgate", em que o método planejado `flatten(..)` acabou renomeado para `flat(..)`.

Mas, ocasionalmente, o TC39 decide que a especificação deve se manter firme em algum ponto, mesmo sendo improvável que as engines JS dos navegadores algum dia se adequem.

A solução? O Apêndice B, "Additional ECMAScript Features for Web Browsers" (Recursos Adicionais do ECMAScript para Navegadores Web).[^specApB] A especificação do JS inclui esse apêndice para detalhar quaisquer descompassos conhecidos entre a especificação oficial do JS e a realidade do JS na web. Em outras palavras, essas são exceções permitidas *apenas* para o JS da web; outros ambientes JS devem seguir a letra da lei.

As seções B.1 e B.2 cobrem *adições* ao JS (sintaxe e APIs) que o JS da web inclui, novamente por razões históricas, mas que o TC39 não planeja especificar formalmente no núcleo do JS. Exemplos incluem literais octais prefixados com `0`, os utilitários globais `escape(..)` / `unescape(..)`, "auxiliares" de String como `anchor(..)` e `blink()`, e o método `compile(..)` de RegExp.

A seção B.3 inclui alguns conflitos em que o código pode rodar tanto em engines JS da web quanto fora dela, mas em que o comportamento *poderia* ser observavelmente diferente, resultando em desfechos distintos. A maior parte das mudanças listadas envolve situações rotuladas como erros precoces quando o código roda em modo estrito.

As *pegadinhas* do Apêndice B não são encontradas com muita frequência, mas ainda assim é uma boa ideia evitar essas construções para se manter seguro no futuro. Sempre que possível, siga a especificação do JS e não dependa de comportamento aplicável apenas em certos ambientes de engine JS.

### Nem Tudo é JS (na Web)...

Este código é um programa JS?

```js
alert("Hello, JS!");
```

Depende de como você olha para as coisas. A função `alert(..)` mostrada aqui não está incluída na especificação do JS, mas *está* em todos os ambientes JS da web. Ainda assim, você não vai encontrá-la no Apêndice B, então qual é a jogada?

Vários ambientes JS (como engines JS de navegadores, Node.js etc.) adicionam APIs ao escopo global dos seus programas JS, dando a você capacidades específicas do ambiente, como poder exibir uma caixa de alerta no navegador do usuário.

De fato, uma ampla gama de APIs com cara de JS, como `fetch(..)`, `getCurrentLocation(..)` e `getUserMedia(..)`, são todas APIs da web que se parecem com JS. No Node.js, podemos acessar centenas de métodos de API a partir de vários módulos embutidos, como `fs.write(..)`.

Outro exemplo comum é `console.log(..)` (e todos os outros métodos `console.*`!). Eles não são especificados no JS, mas, por causa de sua utilidade universal, são definidos por praticamente todo ambiente JS, de acordo com um consenso aproximado.

Então `alert(..)` e `console.log(..)` não são definidos pelo JS. Mas *parecem* JS. São funções e métodos de objeto e obedecem às regras de sintaxe do JS. Os comportamentos por trás deles são controlados pelo ambiente que executa a engine JS, mas, na superfície, eles definitivamente precisam respeitar o JS para poder brincar no parquinho do JS.

A maior parte das diferenças entre navegadores de que as pessoas reclamam com afirmações do tipo "JS é tão inconsistente!" na verdade se deve a diferenças em como esses comportamentos de ambiente funcionam, e não em como o próprio JS funciona.

Portanto, uma chamada `alert(..)` *é* JS, mas `alert` em si é apenas um convidado, não parte da especificação oficial do JS.

### Nem Sempre é JS

Usar o console/REPL (Read-Evaluate-Print-Loop) nas Ferramentas de Desenvolvedor do seu navegador (ou no Node) parece, à primeira vista, um ambiente JS bem direto. Mas, na verdade, não é.

Ferramentas de Desenvolvedor são... ferramentas para desenvolvedores. Seu propósito principal é facilitar a vida dos desenvolvedores. Elas priorizam a DX (Developer Experience, ou Experiência do Desenvolvedor). *Não* é um objetivo dessas ferramentas refletir com precisão e pureza todas as nuances do comportamento estrito do JS conforme a especificação. Por isso, há muitas peculiaridades que podem funcionar como "pegadinhas" se você tratar o console como um ambiente JS *puro*.

Essa conveniência é uma coisa boa, aliás! Fico feliz que as Ferramentas de Desenvolvedor facilitem a vida dos desenvolvedores! Fico feliz que tenhamos confortos de UX como autocompletar de variáveis/propriedades etc. Só estou apontando que não podemos e não devemos esperar que essas ferramentas *sempre* adiram estritamente à forma como programas JS são tratados, porque esse não é o propósito delas.

Como essas ferramentas variam de comportamento entre navegadores, e como elas mudam (às vezes com bastante frequência), não vou "codificar de forma fixa" nenhum detalhe específico neste texto, o que faria com que o texto deste livro ficasse desatualizado rapidamente.

Mas vou apenas dar alguns exemplos de peculiaridades que foram verdadeiras em diferentes momentos em diferentes ambientes de console JS, para reforçar meu ponto de não presumir comportamento nativo do JS ao usá-los:

* Se uma declaração `var` ou `function` no "escopo global" de nível superior do console realmente cria uma variável global de verdade (e uma propriedade espelhada em `window`, e vice-versa!).

* O que acontece com múltiplas declarações `let` e `const` no "escopo global" de nível superior.

* Se `"use strict";` em uma entrada de linha (pressionando `<enter>` em seguida) habilita o modo estrito para o restante daquela sessão do console, do jeito que faria na primeira linha de um arquivo .js, bem como se você pode usar `"use strict";` além da "primeira linha" e ainda assim ativar o modo estrito naquela sessão.

* Como funciona o *default binding* de `this` em modo não estrito nas chamadas de função, e se o "objeto global" usado vai conter as variáveis globais esperadas.

* Como o hoisting (veja o Livro 2, *Scope & Closures*) funciona ao longo de múltiplas entradas de linha.

* ...vários outros

O console do desenvolvedor não está tentando fingir ser um compilador JS que trata o código digitado exatamente do mesmo jeito que a engine JS trata um arquivo .js. Ele está tentando facilitar para você digitar rapidamente algumas linhas de código e ver os resultados imediatamente. Esses são casos de uso totalmente diferentes e, por isso, é irrazoável esperar que uma única ferramenta lide bem com ambos.

Não confie no comportamento que você vê em um console de desenvolvedor como representando a semântica do JS *exata* e ao pé da letra; para isso, leia a especificação. Em vez disso, pense no console como um ambiente "amigável ao JS". Isso já é útil por si só.

## Muitas Faces

O termo "paradigma", no contexto de linguagens de programação, se refere a uma mentalidade e abordagem ampla (quase universal) para estruturar código. Dentro de um paradigma, há incontáveis variações de estilo e forma que distinguem programas, incluindo inúmeras bibliotecas e frameworks diferentes que deixam sua assinatura única em qualquer código.

Mas, não importa qual seja o estilo individual de um programa, as grandes divisões em torno de paradigmas quase sempre ficam evidentes já no primeiro olhar sobre qualquer programa.

As categorias típicas de código no nível de paradigma incluem procedural, orientada a objetos (OO/classes) e funcional (FP):

* O estilo procedural organiza o código em uma progressão linear, de cima para baixo, através de um conjunto predeterminado de operações, geralmente reunidas em unidades relacionadas chamadas procedimentos.

* O estilo OO organiza o código reunindo lógica e dados em unidades chamadas classes.

* O estilo FP organiza o código em funções (computações puras, em oposição a procedimentos) e nas adaptações dessas funções como valores.

Paradigmas não são certos nem errados. São orientações que guiam e moldam como programadores abordam problemas e soluções, como estruturam e mantêm seu código.

Algumas linguagens pendem fortemente para um único paradigma — C é procedural, Java/C++ são quase inteiramente orientadas a classes e Haskell é FP de ponta a ponta.

Mas muitas linguagens também suportam padrões de código que podem vir de — e até misturar e combinar — paradigmas diferentes. As chamadas "linguagens multiparadigma" oferecem flexibilidade máxima. Em alguns casos, um único programa pode até ter duas ou mais expressões desses paradigmas lado a lado.

JavaScript é definitivamente uma linguagem multiparadigma. Você pode escrever código procedural, orientado a classes ou em estilo FP, e pode tomar essas decisões linha a linha, em vez de ser forçado a uma escolha do tipo tudo ou nada.

## Para Trás e Para Frente

Um dos princípios mais fundamentais que guiam o JavaScript é a preservação da *compatibilidade retroativa* (*backwards compatibility*). Muita gente se confunde com as implicações desse termo e frequentemente o confunde com um termo relacionado, mas diferente: *compatibilidade futura* (*forwards compatibility*).

Vamos esclarecer isso.

Compatibilidade retroativa significa que, uma vez que algo é aceito como JS válido, não haverá uma mudança futura na linguagem que faça esse código se tornar JS inválido. Código escrito em 1995 — por mais primitivo ou limitado que pudesse ser! — deve continuar funcionando hoje. Como os membros do TC39 costumam proclamar: "nós não quebramos a web!"

A ideia é que desenvolvedores JS possam escrever código com a confiança de que ele não vai parar de funcionar de forma imprevisível por causa do lançamento de uma atualização de navegador. Isso torna a decisão de escolher JS para um programa um investimento mais sábio e seguro, por anos afora.

Essa "garantia" não é pouca coisa. Manter compatibilidade retroativa, esticada ao longo de quase 25 anos da história da linguagem, cria um fardo enorme e toda uma série de desafios únicos. Você teria dificuldade em encontrar muitos outros exemplos na computação de um compromisso assim com a compatibilidade retroativa.

Os custos de se manter fiel a esse princípio não devem ser descartados levianamente. Ele necessariamente cria uma barreira muito alta para mudar ou estender a linguagem; qualquer decisão se torna, na prática, permanente, erros inclusive. Uma vez que algo entra no JS, não pode ser retirado, porque isso poderia quebrar programas, mesmo que a gente realmente, realmente queira remover!

Existem pequenas exceções a essa regra. O JS já teve algumas mudanças incompatíveis com versões anteriores, mas o TC39 é extremamente cauteloso ao fazer isso. Eles estudam o código existente na web (por meio de coleta de dados dos navegadores) para estimar o impacto dessa quebra, e os navegadores, no fim, decidem e votam se estão dispostos a encarar a irritação dos usuários por uma quebra de escala muito pequena, pesada contra os benefícios de corrigir ou melhorar algum aspecto da linguagem para muitos outros sites (e usuários).

Esse tipo de mudança é raro e quase sempre ocorre em casos de borda de uso, com baixa probabilidade de quebrar algo observável em muitos sites.

Compare a *compatibilidade retroativa* com sua contraparte, a *compatibilidade futura*. Ser compatível com o futuro significa que incluir uma nova adição da linguagem em um programa não faria esse programa quebrar se fosse executado em uma engine JS mais antiga. **JS não é compatível com o futuro**, apesar de muitos desejarem que fosse e até acreditarem incorretamente no mito de que é.

HTML e CSS, por outro lado, são compatíveis com o futuro, mas não com o passado. Se você desenterrar algum HTML ou CSS escrito lá em 1995, é perfeitamente possível que ele não funcione (ou não funcione igual) hoje. Mas, se você usar um recurso novo de 2019 em um navegador de 2010, a página não fica "quebrada" — o CSS/HTML não reconhecido é ignorado, enquanto o restante do CSS/HTML é processado normalmente.

Pode parecer desejável incluir compatibilidade futura no design de linguagens de programação, mas em geral isso é impraticável. Marcação (HTML) ou estilização (CSS) são declarativas por natureza, então é muito mais fácil "pular" declarações não reconhecidas com impacto mínimo sobre as outras declarações reconhecidas.

Mas o caos e o não determinismo se instalariam se uma engine de linguagem de programação pulasse seletivamente instruções (ou até expressões!) que não entendesse, já que é impossível garantir que uma parte posterior do programa não esperava que a parte pulada tivesse sido processada.

Embora o JS não seja, e não possa ser, compatível com o futuro, é fundamental reconhecer a compatibilidade retroativa do JS, incluindo os benefícios duradouros para a web e as restrições e dificuldades que isso impõe ao JS como consequência.

### Pulando as Lacunas

Como o JS não é compatível com o futuro, sempre existe o potencial de uma lacuna entre o código que você pode escrever e que é JS válido e a engine mais antiga que seu site ou aplicação precisa suportar. Se você rodar um programa que usa um recurso do ES2019 em uma engine de 2016, é bem provável que veja o programa quebrar e falhar.

Se o recurso for uma sintaxe nova, o programa em geral vai falhar completamente ao compilar e executar, normalmente lançando um erro de sintaxe. Se o recurso for uma API (como o `Object.is(..)` do ES6), o programa pode rodar até certo ponto, mas então lançar uma exceção em tempo de execução e parar assim que encontrar a referência à API desconhecida.

Isso significa que desenvolvedores JS devem sempre ficar para trás em relação ao ritmo do progresso, usando apenas código que esteja na borda mais atrasada dos ambientes de engine JS mais antigos que precisam suportar? Não!

Mas significa, sim, que desenvolvedores JS precisam tomar um cuidado especial para lidar com essa lacuna.

Para sintaxe nova e incompatível, a solução é a transpilação. Transpilar é um termo artificial, inventado pela comunidade, para descrever o uso de uma ferramenta que converte o código-fonte de um programa de uma forma para outra (mas ainda como código-fonte textual). Tipicamente, problemas de compatibilidade futura relacionados à sintaxe são resolvidos usando um transpilador (o mais comum sendo o Babel (https://babeljs.io)) para converter daquela versão mais nova de sintaxe JS para uma sintaxe mais antiga equivalente.

Por exemplo, um desenvolvedor pode escrever um trecho de código assim:

```js
if (something) {
    let x = 3;
    console.log(x);
}
else {
    let x = 4;
    console.log(x);
}
```

É assim que o código ficaria na árvore de código-fonte daquela aplicação. Mas, ao produzir o(s) arquivo(s) para publicar no site público, o transpilador Babel pode converter esse código para algo assim:

```js
var x$0, x$1;
if (something) {
    x$0 = 3;
    console.log(x$0);
}
else {
    x$1 = 4;
    console.log(x$1);
}
```

O trecho original dependia de `let` para criar variáveis `x` com escopo de bloco tanto na cláusula `if` quanto na `else`, sem que uma interferisse na outra. Um programa equivalente (com retrabalho mínimo), que o Babel consegue produzir, simplesmente escolhe nomear duas variáveis diferentes com nomes únicos, produzindo o mesmo resultado de não interferência.

| NOTA: |
| :--- |
| A palavra-chave `let` foi adicionada no ES6 (em 2015). O exemplo anterior de transpilação só precisaria ser aplicado se uma aplicação precisasse rodar em um ambiente JS anterior ao suporte a ES6. O exemplo aqui é só para simplificar a ilustração. Quando o ES6 era novidade, a necessidade dessa transpilação era bastante comum, mas, em 2020, é muito menos comum precisar suportar ambientes pré-ES6. O "alvo" usado para a transpilação é, portanto, uma janela deslizante que só se move para cima conforme se decide que um site/aplicação vai parar de suportar algum navegador/engine antigo. |

Você pode se perguntar: por que dar ao trabalho de usar uma ferramenta para converter de uma versão de sintaxe mais nova para uma mais antiga? Não poderíamos simplesmente escrever as duas variáveis e deixar de usar a palavra-chave `let`? A razão é que se recomenda fortemente que desenvolvedores usem a versão mais recente do JS, para que seu código fique limpo e comunique suas ideias da forma mais eficaz.

Desenvolvedores devem focar em escrever as formas de sintaxe novas e limpas, e deixar que as ferramentas cuidem de produzir uma versão compatível com o futuro desse código, adequada para publicar e rodar nos ambientes de engine JS mais antigos suportados.

### Preenchendo as Lacunas

Se o problema de compatibilidade futura não estiver relacionado a uma nova sintaxe, mas sim a um método de API ausente que foi adicionado recentemente, a solução mais comum é fornecer uma definição para esse método de API faltante, que faça as vezes dele e se comporte como se o ambiente mais antigo já o tivesse definido nativamente. Esse padrão é chamado de polyfill (também conhecido como "shim").

Considere este código:

```js
// getSomeRecords() nos devolve uma promise para
// alguns dados que ela vai buscar
var pr = getSomeRecords();

// mostra o spinner da UI enquanto buscamos os dados
startSpinner();

pr
.then(renderRecords)   // renderiza se der certo
.catch(showError)      // mostra um erro se não der
.finally(hideSpinner)  // sempre esconde o spinner
```

Este código usa um recurso do ES2019, o método `finally(..)` no prototype de promise. Se esse código fosse usado em um ambiente anterior ao ES2019, o método `finally(..)` não existiria e ocorreria um erro.

Um polyfill para `finally(..)` em ambientes pré-ES2019 poderia se parecer com isto:

```js
if (!Promise.prototype.finally) {
    Promise.prototype.finally = function f(fn){
        return this.then(
            function t(v){
                return Promise.resolve( fn() )
                    .then(function t(){
                        return v;
                    });
            },
            function c(e){
                return Promise.resolve( fn() )
                    .then(function t(){
                        throw e;
                    });
            }
        );
    };
}
```

| AVISO: |
| :--- |
| Esta é apenas uma ilustração simples de um polyfill básico (não totalmente conforme a especificação) para `finally(..)`. Não use este polyfill no seu código; sempre use um polyfill robusto e oficial sempre que possível, como a coleção de polyfills/shims do ES-Shim. |

A instrução `if` protege a definição do polyfill, impedindo que ele rode em qualquer ambiente em que a engine JS já tenha definido esse método. Em ambientes mais antigos, o polyfill é definido; em ambientes mais novos, a instrução `if` é silenciosamente ignorada.

Transpiladores como o Babel tipicamente detectam de quais polyfills seu código precisa e os fornecem automaticamente para você. Mas, ocasionalmente, você pode precisar incluí-los/defini-los explicitamente, o que funciona de forma parecida com o trecho que acabamos de ver.

Sempre escreva código usando os recursos mais apropriados para comunicar suas ideias e intenções de forma eficaz. Em geral, isso significa usar a versão estável mais recente do JS. Evite impactar negativamente a legibilidade do código tentando ajustar manualmente as lacunas de sintaxe/API. É para isso que existem as ferramentas!

Transpilação e polyfilling são duas técnicas altamente eficazes para lidar com aquela lacuna entre o código que usa os recursos estáveis mais recentes da linguagem e os ambientes antigos que um site ou aplicação ainda precisa suportar. Já que o JS não vai parar de melhorar, a lacuna nunca vai desaparecer. Ambas as técnicas devem ser abraçadas como parte padrão da cadeia de produção de todo projeto JS daqui para frente.

## O que Há em uma Interpretação?

Uma pergunta longamente debatida sobre código escrito em JS: é um script interpretado ou um programa compilado? A opinião majoritária parece ser que JS é uma linguagem interpretada (de scripting). Mas a verdade é mais complicada do que isso.

Durante boa parte da história das linguagens de programação, linguagens "interpretadas" e linguagens de "scripting" foram vistas com desdém, como inferiores às suas contrapartes compiladas. As razões dessa hostilidade são numerosas, incluindo a percepção de que falta otimização de desempenho, bem como a antipatia por certas características da linguagem, como o fato de linguagens de scripting geralmente usarem tipagem dinâmica em vez das linguagens "mais maduras" de tipagem estática.

Linguagens consideradas "compiladas" normalmente produzem uma representação portátil (binária) do programa, que é distribuída para execução posterior. Como não observamos realmente esse tipo de modelo com o JS (distribuímos o código-fonte, não a forma binária), muitos afirmam que isso desqualifica o JS dessa categoria. Na realidade, o modelo de distribuição da forma "executável" de um programa se tornou drasticamente mais variado e também menos relevante nas últimas décadas; para a questão em pauta, já não importa tanto assim qual forma de um programa é repassada por aí.

Essas afirmações e críticas mal informadas devem ser deixadas de lado. A verdadeira razão pela qual importa ter um quadro claro sobre se o JS é interpretado ou compilado diz respeito à natureza de como os erros são tratados.

Historicamente, linguagens de script ou interpretadas eram executadas, em geral, de cima para baixo e linha a linha; normalmente não há uma passagem inicial pelo programa para processá-lo antes de a execução começar (veja a Figura 1).

<figure>
    <img src="../../../get-started/images/fig1.png" width="650" alt="Interpretando um script para executá-lo" align="center">
    <figcaption><em>Fig. 1: Execução Interpretada/Script</em></figcaption>
    <br><br>
</figure>

Em linguagens de script ou interpretadas, um erro na linha 5 de um programa não será descoberto até que as linhas 1 a 4 já tenham sido executadas. Notavelmente, o erro na linha 5 pode se dever a uma condição de tempo de execução, como alguma variável ou valor com um valor inadequado para uma operação, ou pode se dever a uma instrução/comando malformado naquela linha. Dependendo do contexto, adiar o tratamento do erro até a linha em que ele ocorre pode ser um efeito desejável ou indesejável.

Compare isso com linguagens que passam por uma etapa de processamento (tipicamente chamada de parsing) antes de qualquer execução, como ilustrado na Figura 2:

<figure>
    <img src="../../../get-started/images/fig2.png" width="650" alt="Fazendo parsing, compilando e executando um programa" align="center">
    <figcaption><em>Fig. 2: Parsing + Compilação + Execução</em></figcaption>
    <br><br>
</figure>

Nesse modelo de processamento, um comando inválido (como uma sintaxe quebrada) na linha 5 seria capturado durante a fase de parsing, antes de qualquer execução ter começado, e nada do programa rodaria. Para capturar erros de sintaxe (ou de outra forma "estáticos"), em geral é preferível saber deles antes de qualquer execução parcial fadada ao fracasso.

Então, o que as linguagens "parseadas" têm em comum com as linguagens "compiladas"? Primeiro, todas as linguagens compiladas passam por parsing. Então uma linguagem que passa por parsing já está bem adiantada no caminho de ser compilada. Na teoria clássica de compilação, a última etapa restante depois do parsing é a geração de código: produzir uma forma executável.

Uma vez que qualquer programa-fonte tenha passado totalmente pelo parsing, é muito comum que sua execução subsequente inclua, de alguma forma, uma tradução da forma parseada do programa — geralmente chamada de Árvore Sintática Abstrata (AST, de *Abstract Syntax Tree*) — para aquela forma executável.

Em outras palavras, linguagens que passam por parsing geralmente também realizam geração de código antes da execução, então não é muito exagero dizer que, em espírito, elas são linguagens compiladas.

O código-fonte JS passa por parsing antes de ser executado. A especificação exige isso, porque ela determina que "erros precoces" — erros determinados estaticamente no código, como um nome de parâmetro duplicado — sejam reportados antes de o código começar a executar. Esses erros não podem ser reconhecidos sem que o código tenha passado pelo parsing.

Então **JS é uma linguagem parseada**, mas é *compilada*?

A resposta está mais para sim do que para não. O JS parseado é convertido para uma forma otimizada (binária), e esse "código" é executado em seguida (Figura 2); a engine normalmente não volta para o modo de execução linha a linha (como na Figura 1) depois de ter terminado todo o trabalho duro do parsing — a maioria das linguagens/engines não faria isso, porque seria altamente ineficiente.

Para ser específico, essa "compilação" produz uma espécie de byte code binário, que é então entregue à "máquina virtual JS" para execução. Alguns gostam de dizer que essa VM está "interpretando" o byte code. Mas então isso significaria que Java, e uma dúzia de outras linguagens que rodam na JVM, aliás, são interpretadas em vez de compiladas. Claro, isso contradiz a afirmação típica de que Java/etc. são linguagens compiladas.

Curiosamente, embora Java e JavaScript sejam linguagens muito diferentes, a questão interpretada/compilada é bastante relacionada entre elas!

Outra complicação é que as engines JS podem empregar múltiplas passagens de processamento/otimização JIT (Just-In-Time) sobre o código gerado (após o parsing), o que, novamente, poderia razoavelmente ser rotulado como "compilação" ou "interpretação", dependendo da perspectiva. Na verdade, é uma situação fantasticamente complexa sob o capô de uma engine JS.

Então, a que se resumem todos esses detalhes minuciosos? Dê um passo atrás e considere todo o fluxo de um programa-fonte JS:

1. Depois que um programa deixa o editor do desenvolvedor, ele é transpilado pelo Babel, depois empacotado pelo Webpack (e talvez por meia dúzia de outros processos de build), e então é entregue, nessa forma bem diferente, a uma engine JS.

2. A engine JS faz o parsing do código para uma AST.

3. Então a engine converte essa AST em uma espécie de byte code, uma representação intermediária (IR) binária, que é então refinada/convertida ainda mais pelo compilador JIT otimizador.

4. Por fim, a VM JS executa o programa.

Para visualizar esses passos, novamente:

<figure>
    <img src="../../../get-started/images/fig3.png" width="650" alt="Passos da compilação e execução do JS" align="center">
    <figcaption><em>Fig. 3: Parsing, Compilação e Execução do JS</em></figcaption>
    <br><br>
</figure>

O JS é tratado mais como um script interpretado, linha a linha, como na Figura 1, ou é tratado mais como uma linguagem compilada, processada em uma ou várias passagens antes da execução (como nas Figuras 2 e 3)?

Eu acho que está claro que, em espírito, se não na prática, **JS é uma linguagem compilada**.

E, de novo, a razão pela qual isso importa é que, já que o JS é compilado, somos informados sobre erros estáticos (como sintaxe malformada) antes de nosso código ser executado. Esse é um modelo de interação substancialmente diferente do que temos com programas de "scripting" tradicionais, e provavelmente mais útil!

### Web Assembly (WASM)

Uma preocupação dominante que impulsionou boa parte da evolução do JS é o desempenho: tanto a rapidez com que o JS pode passar por parsing/compilação quanto a rapidez com que esse código compilado pode ser executado.

Em 2013, engenheiros do Mozilla Firefox demonstraram uma portabilidade da engine de jogos Unreal 3 de C para JS. A capacidade desse código rodar em uma engine JS de navegador com desempenho pleno de 60fps se baseava em um conjunto de otimizações que a engine JS podia realizar especificamente porque a versão JS do código da engine Unreal usava um estilo de código que favorecia um subconjunto da linguagem JS, chamado "ASM.js".

Esse subconjunto é JS válido, escrito de formas um tanto incomuns na codificação normal, mas que sinalizam certas informações importantes de tipagem para a engine, permitindo que ela faça otimizações essenciais. O ASM.js foi introduzido como uma maneira de lidar com as pressões sobre o desempenho em tempo de execução do JS.

Mas é importante notar que o ASM.js nunca teve a intenção de ser código escrito por desenvolvedores, e sim uma representação de um programa que foi transpilado de outra linguagem (como C), em que essas "anotações" de tipagem eram inseridas automaticamente pelo ferramental.

Vários anos depois de o ASM.js demonstrar a validade de versões de programas criadas por ferramentas e que podem ser processadas de forma mais eficiente pela engine JS, outro grupo de engenheiros (também, inicialmente, da Mozilla) lançou o Web Assembly (WASM).

O WASM é parecido com o ASM.js no sentido de que sua intenção original era oferecer um caminho para que programas não-JS (C etc.) fossem convertidos para uma forma capaz de rodar na engine JS. Diferente do ASM.js, o WASM escolheu também contornar alguns dos atrasos inerentes ao parsing/compilação do JS antes que um programa possa executar, representando o programa em uma forma totalmente diferente do JS.

O WASM é um formato de representação mais parecido com Assembly (daí o nome), que pode ser processado por uma engine JS pulando o parsing/compilação que a engine JS normalmente faz. O parsing/compilação de um programa que tem o WASM como alvo acontece antecipadamente (AOT, de *ahead of time*); o que é distribuído é um programa empacotado em binário, pronto para a engine JS executar com pouquíssimo processamento.

Uma motivação inicial do WASM foram, claramente, as potenciais melhorias de desempenho. Embora isso continue sendo um foco, o WASM é motivado também pelo desejo de trazer mais paridade para linguagens não-JS na plataforma web. Por exemplo, se uma linguagem como Go suporta programação com threads, mas o JS (a linguagem) não, o WASM oferece o potencial de converter um programa Go assim para uma forma que a engine JS entenda, sem precisar de um recurso de threads na própria linguagem JS.

Em outras palavras, o WASM alivia a pressão por adicionar ao JS recursos que se destinam majoritariamente/exclusivamente a serem usados por programas transpilados de outras linguagens. Isso significa que o desenvolvimento de recursos do JS pode ser avaliado (pelo TC39) sem ser distorcido por interesses/demandas de outros ecossistemas de linguagem, ao mesmo tempo em que permite que essas linguagens tenham um caminho viável até a web.

Outra perspectiva sobre o WASM que vem emergindo, curiosamente, nem sequer está diretamente relacionada à web (o W). O WASM está evoluindo para se tornar uma espécie de máquina virtual (VM) multiplataforma, em que programas podem ser compilados uma vez e rodar em diversos ambientes de sistema diferentes.

Então o WASM não é só para a web, e o WASM também não é JS. Ironicamente, embora o WASM rode na engine JS, a linguagem JS é uma das menos adequadas como origem para programas WASM, porque o WASM depende fortemente de informações de tipagem estática. Mesmo o TypeScript (TS) — ostensivamente, JS + tipos estáticos — não é exatamente adequado (como está hoje) para transpilar para WASM, embora variantes da linguagem, como AssemblyScript, estejam tentando fazer a ponte entre JS/TS e WASM.

Este livro não é sobre WASM, então não vou gastar muito mais tempo discutindo isso, exceto para fazer uma última observação. *Algumas* pessoas sugeriram que o WASM aponta para um futuro em que o JS seria extirpado, ou minimizado, na web. Essas pessoas costumam nutrir sentimentos ruins em relação ao JS e querem alguma outra linguagem — qualquer outra linguagem! — para substituí-lo. Já que o WASM permite que outras linguagens rodem na engine JS, à primeira vista isso não é um conto de fadas totalmente fantasioso.

Mas deixe-me afirmar de forma simples: o WASM não vai substituir o JS. O WASM amplia significativamente o que a web (incluindo o JS) pode realizar. Isso é ótimo, e totalmente ortogonal ao fato de algumas pessoas usarem isso como rota de fuga para não precisar escrever JS.

## Falando *Estrita*mente

Lá em 2009, com o lançamento do ES5, o JS adicionou o *modo estrito* (*strict mode*) como um mecanismo opcional para incentivar programas JS melhores.

Os benefícios do modo estrito superam de longe os custos, mas velhos hábitos custam a morrer, e a inércia de bases de código existentes (ou seja, "legadas") é muito difícil de vencer. Então, tristemente, mais de 10 anos depois, a *opcionalidade* do modo estrito significa que ele ainda não é necessariamente o padrão para programadores JS.

Por que modo estrito? O modo estrito não deveria ser pensado como uma restrição do que você não pode fazer, mas sim como um guia da melhor maneira de fazer as coisas, para que a engine JS tenha a melhor chance de otimizar e executar o código de forma eficiente. A maior parte do código JS é trabalhada por times de desenvolvedores, então o rigor (*strict*-ness) do modo estrito (junto com ferramentas como linters!) muitas vezes ajuda na colaboração sobre o código, evitando alguns dos erros mais problemáticos que passam despercebidos no modo não estrito.

A maioria dos controles do modo estrito assume a forma de *erros precoces*, ou seja, erros que não são estritamente erros de sintaxe, mas que ainda assim são lançados em tempo de compilação (antes de o código rodar). Por exemplo, o modo estrito proíbe nomear dois parâmetros de função da mesma forma, e isso resulta em um erro precoce. Alguns outros controles do modo estrito só são observáveis em tempo de execução, como o fato de `this` assumir `undefined` por padrão em vez do objeto global.

Em vez de brigar e discutir com o modo estrito, como uma criança que só quer desafiar tudo o que os pais mandam não fazer, a melhor mentalidade é que o modo estrito é como um linter lembrando você de como o JS *deveria* ser escrito para ter a maior qualidade e a melhor chance de desempenho. Se você se pegar se sentindo algemado, tentando contornar o modo estrito, isso deveria ser um enorme alerta vermelho de que você precisa recuar e repensar toda a abordagem.

O modo estrito é ligado por arquivo, com um pragma especial (nada é permitido antes dele, exceto comentários/espaços em branco):

```js
// apenas espaços em branco e comentários são permitidos
// antes do pragma use-strict
"use strict";
// o restante do arquivo roda em modo estrito
```

| AVISO: |
| :--- |
| Algo a se ter em mente é que até mesmo um `;` solto, sozinho, aparecendo antes do pragma de modo estrito, vai tornar o pragma inútil; nenhum erro é lançado, porque é JS válido ter uma expressão literal de string em posição de instrução, mas isso também *não* vai ligar o modo estrito, silenciosamente! |

Alternativamente, o modo estrito pode ser ligado por escopo de função, com exatamente as mesmas regras quanto ao que vem antes:

```js
function someOperations() {
    // espaços em branco e comentários são ok aqui
    "use strict";

    // todo este código vai rodar em modo estrito
}
```

Curiosamente, se um arquivo tem o modo estrito ligado, os pragmas de modo estrito em nível de função são proibidos. Então você tem que escolher um ou outro.

A **única** razão válida para usar uma abordagem de modo estrito por função é quando você está convertendo um arquivo de programa existente em modo não estrito e precisa fazer as mudanças aos poucos, ao longo do tempo. Fora isso, é imensamente melhor simplesmente ligar o modo estrito para o arquivo/programa inteiro.

Muita gente se perguntou se algum dia o JS tornaria o modo estrito o padrão. A resposta é: quase certamente não. Como discutimos antes sobre compatibilidade retroativa, se uma atualização de engine JS começasse a presumir que o código está em modo estrito mesmo sem estar marcado como tal, é possível que esse código quebrasse em função dos controles do modo estrito.

No entanto, há alguns fatores que reduzem o impacto futuro dessa "obscuridade" de o modo estrito não ser o padrão.

Primeiro, virtualmente todo código transpilado acaba em modo estrito, mesmo que o código-fonte original não seja escrito assim. A maior parte do código JS em produção foi transpilada, o que significa que a maior parte do JS já está aderindo ao modo estrito. É possível desfazer essa premissa, mas você realmente teria que se esforçar para isso, então é altamente improvável.

Além disso, está acontecendo uma ampla mudança na direção de mais/quase todo código JS novo ser escrito usando o formato de módulos do ES6. Módulos ES6 assumem modo estrito, então todo código nesses arquivos é automaticamente colocado em modo estrito por padrão.

Somando tudo, o modo estrito é, em grande medida, o padrão de fato, ainda que tecnicamente não seja o padrão.

## Definido

JS é uma implementação do padrão ECMAScript (versão ES2019 no momento em que escrevo), guiado pelo comitê TC39 e hospedado pela ECMA. Ele roda em navegadores e em outros ambientes JS, como o Node.js.

JS é uma linguagem multiparadigma, ou seja, a sintaxe e as capacidades permitem que um desenvolvedor misture e combine (e dobre e remodele!) conceitos de vários paradigmas principais, como procedural, orientado a objetos (OO/classes) e funcional (FP).

JS é uma linguagem compilada, o que significa que as ferramentas (incluindo a engine JS) processam e verificam um programa (reportando quaisquer erros!) antes de ele executar.

Com nossa linguagem agora *definida*, vamos começar a conhecer seus detalhes.

[^specApB]: ECMAScript 2019 Language Specification, Appendix B: Additional ECMAScript Features for Web Browsers, https://www.ecma-international.org/ecma-262/10.0/#sec-additional-ecmascript-features-for-web-browsers (mais recente no momento em que escrevo, em janeiro de 2020)
