# You Don't Know JS Yet: Get Started - 2ª Edição
# Capítulo 4: O Quadro Geral

Este livro percorre o que você precisa saber enquanto *começa* com JS. O objetivo é preencher lacunas nas quais leitores mais novos em JS podem ter tropeçado em seus primeiros encontros com a linguagem. Também espero que tenhamos sinalizado detalhes profundos o suficiente ao longo do caminho para aguçar sua curiosidade e sua vontade de se aprofundar mais na linguagem.

Os demais livros desta série são onde vamos desempacotar todo o restante da linguagem, com muito mais detalhe do que poderíamos ter feito em alguns breves capítulos aqui.

Lembre-se de ir com calma, porém. Em vez de correr para o próximo livro numa tentativa de atravessar todos os livros rapidamente, gaste um tempo revisitando o material deste livro. Gaste mais um tempo olhando o código dos seus projetos atuais e comparando o que você vê com o que foi discutido até aqui.

Quando você estiver pronto, este capítulo final divide a organização da linguagem JS em três pilares principais, depois oferece um breve roteiro do que esperar do restante da série de livros e de como sugiro que você prossiga. E não pule os apêndices, especialmente o Apêndice B, "Prática, Prática, Prática!".

## Pilar 1: Escopo e Closure

A organização de variáveis em unidades de escopo (funções, blocos) é uma das características mais fundamentais de qualquer linguagem; talvez nenhuma outra característica tenha impacto maior sobre como os programas se comportam.

Escopos são como baldes, e variáveis são como bolinhas de gude que você coloca nesses baldes. O modelo de escopo de uma linguagem é como as regras que te ajudam a determinar quais bolinhas de qual cor vão em quais baldes da cor correspondente.

Escopos se aninham uns dentro dos outros e, para qualquer expressão ou instrução, apenas variáveis daquele nível de aninhamento de escopo, ou de escopos superiores/externos, são acessíveis; variáveis de escopos inferiores/internos ficam escondidas e inacessíveis.

É assim que escopos se comportam na maioria das linguagens, o que se chama escopo léxico. Os limites das unidades de escopo, e como as variáveis são organizadas neles, são determinados no momento em que o programa passa por parsing (compilação). Em outras palavras, é uma decisão de tempo de escrita: onde você localiza uma função/escopo no programa determina qual será a estrutura de escopo daquela parte do programa.

JS tem escopo léxico, embora muitos afirmem que não, por causa de duas características específicas do seu modelo que não estão presentes em outras linguagens de escopo léxico.

A primeira é comumente chamada de *hoisting*: quando todas as variáveis declaradas em qualquer lugar de um escopo são tratadas como se tivessem sido declaradas no início do escopo. A outra é que variáveis declaradas com `var` têm escopo de função, mesmo que apareçam dentro de um bloco.

Nem o hoisting nem o `var` com escopo de função são suficientes para sustentar a afirmação de que o JS não tem escopo léxico. Declarações `let`/`const` têm um comportamento peculiar de erro chamado "Zona Morta Temporal" (TDZ, de *Temporal Dead Zone*), que resulta em variáveis observáveis, mas inutilizáveis. Embora a TDZ possa ser estranha de encontrar, ela *também* não invalida o escopo léxico. Todas essas são apenas partes únicas da linguagem, que deveriam ser aprendidas e compreendidas por todos os desenvolvedores JS.

Closure é um resultado natural do escopo léxico quando a linguagem tem funções como valores de primeira classe, como o JS tem. Quando uma função faz referência a variáveis de um escopo externo, e essa função é passada adiante como valor e executada em outros escopos, ela mantém acesso às variáveis de seu escopo original; isso é closure.

Em toda a programação, mas especialmente em JS, closure impulsiona muitos dos padrões de programação mais importantes, incluindo módulos. Do jeito que eu vejo, módulos são o mais *a favor do grão* que se pode chegar quando o assunto é organização de código em JS.

Para se aprofundar em escopo, closures e como módulos funcionam, leia o Livro 2, *Scope & Closures*.

## Pilar 2: Protótipos

O segundo pilar da linguagem é o sistema de protótipos. Cobrimos esse tópico em profundidade no Capítulo 3 ("Protótipos"), mas quero apenas fazer mais alguns comentários sobre sua importância.

JS é uma das pouquíssimas linguagens em que você tem a opção de criar objetos direta e explicitamente, sem primeiro definir sua estrutura em uma classe.

Por muitos anos, as pessoas implementaram o padrão de design de classes sobre protótipos — a chamada "herança prototipal" (veja o Apêndice A, "'Classes' Prototipais") — e então, com a chegada da palavra-chave `class` do ES6, a linguagem reforçou sua inclinação à programação em estilo OO/classes.

Mas acho que esse foco obscureceu a beleza e o poder do sistema de protótipos: a capacidade de dois objetos simplesmente se conectarem entre si e cooperarem dinamicamente (durante a execução de função/método) por meio do compartilhamento de um contexto `this`.

Classes são apenas um padrão que você pode construir sobre esse poder. Mas outra abordagem, numa direção bem diferente, é simplesmente abraçar objetos como objetos, esquecer classes por completo e deixar os objetos cooperarem pela cadeia de protótipos. Isso se chama *delegação de comportamento*. Eu acho que a delegação é mais poderosa do que a herança de classes como meio de organizar comportamento e dados nos nossos programas.

Mas a herança de classes fica com quase toda a atenção. E o resto vai para a programação funcional (FP), como o jeito meio "anticlasse" de projetar programas. Isso me entristece, porque sufoca qualquer chance de exploração da delegação como alternativa viável.

Eu te encorajo a passar bastante tempo mergulhado no Livro 3, *Objects & Classes*, para ver como a delegação de objetos guarda muito mais potencial do que talvez tenhamos percebido. Esta não é uma mensagem anti-`class`, mas é intencionalmente uma mensagem de "classes não são a única forma de usar objetos", que eu gostaria que mais desenvolvedores JS considerassem.

A delegação de objetos é, eu diria, muito mais *a favor do grão* do JS do que as classes (mais sobre *grão* daqui a pouco).

## Pilar 3: Tipos e Coerção

O terceiro pilar do JS é, de longe, a parte mais negligenciada da natureza do JS.

A imensa maioria dos desenvolvedores tem fortes equívocos sobre como *tipos* funcionam em linguagens de programação e, especialmente, sobre como funcionam em JS. Uma onda de interesse na comunidade JS mais ampla começou a se deslocar para abordagens de "tipagem estática", usando ferramental ciente de tipos como TypeScript ou Flow.

Concordo que desenvolvedores JS deveriam aprender mais sobre tipos e deveriam aprender mais sobre como o JS gerencia conversões de tipo. Também concordo que ferramental ciente de tipos pode ajudar desenvolvedores, presumindo que eles tenham adquirido e usado esse conhecimento antes de tudo!

Mas não concordo nem um pouco que a conclusão inevitável disso seja decidir que o mecanismo de tipos do JS é ruim e que precisamos encobrir os tipos do JS com soluções de fora da linguagem. Não precisamos seguir o caminho da "tipagem estática" para sermos espertos e sólidos com tipos nos nossos programas. Há outras opções, se você estiver disposto a ir *contra o grão* da multidão e *a favor do grão* do JS (de novo, mais sobre isso adiante).

Argumentavelmente, este pilar é mais importante do que os outros dois, no sentido de que nenhum programa JS fará nada útil se não aproveitar corretamente os tipos de valor do JS, bem como a conversão (coerção) de valores entre tipos.

Mesmo que você ame TypeScript/Flow, você não vai extrair o máximo dessas ferramentas ou abordagens de codificação se não for profundamente familiarizado com como a própria linguagem gerencia tipos de valor.

Para aprender mais sobre tipos e coerção em JS, confira o Livro 4, *Types & Grammar*. Mas, por favor, não pule esse tópico só porque você sempre ouviu que devemos usar `===` e esquecer o resto.

Sem aprender este pilar, sua base em JS é, na melhor das hipóteses, instável e incompleta.

## A Favor do Grão

Tenho alguns conselhos para compartilhar sobre continuar sua jornada de aprendizado com JS e seu caminho pelo restante desta série de livros: fique atento ao *grão* (lembre-se das várias referências a *grão* antes neste capítulo).

Primeiro, considere o *grão* (como na madeira) de como a maioria das pessoas aborda e usa JS. Você provavelmente já notou que estes livros cortam contra esse *grão* em muitos aspectos. Na YDKJSY, eu te respeito, leitor, o suficiente para explicar todas as partes do JS, não apenas algumas partes populares selecionadas. Acredito que você é capaz e merecedor desse conhecimento.

Mas não é isso que você vai encontrar em muito material por aí. Isso também significa que, quanto mais você seguir e aderir à orientação destes livros — de pensar com cuidado e analisar por conta própria o que é melhor no seu código —, mais você vai se destacar. Isso pode ser uma coisa boa e ruim. Se você quiser se separar da multidão, vai ter que se separar do jeito como a multidão faz!

Mas também já tive muita gente me dizendo que citou algum tópico/explicação destes livros durante uma entrevista de emprego, e o entrevistador disse ao candidato que ele estava errado; de fato, há relatos de pessoas que perderam ofertas de emprego em decorrência disso.

Na medida do possível, eu me esforço nestes livros para fornecer informação completamente precisa sobre JS, informada geralmente pela própria especificação. Mas também distribuo bastante das minhas opiniões sobre como você pode interpretar e usar o JS com o melhor proveito nos seus programas. Eu não apresento opinião como fato, nem vice-versa. Você sempre vai saber o que é o quê nestes livros.

Fatos sobre JS não estão realmente em debate. Ou a especificação diz algo, ou não diz. Se você não gosta do que a especificação diz, ou do meu relato dela, resolva isso com o TC39! Se você estiver em uma entrevista e disserem que você está errado quanto aos fatos, pergunte ali mesmo se pode consultar a especificação. Se o entrevistador não reconsiderar, então você não deveria querer trabalhar lá mesmo.

Mas, se você escolher se alinhar às minhas opiniões, precisa estar preparado para embasar essas escolhas com o *porquê* de você sentir assim. Não repita como papagaio o que eu digo. Assuma suas opiniões. Defenda-as. E, se alguém com quem você esperava trabalhar discordar, vá embora ainda de cabeça erguida. O JS é grande, e há bastante espaço para muitos jeitos diferentes.

Em outras palavras, não tenha medo de ir contra o *grão*, como eu fiz com estes livros e com todos os meus ensinamentos. Ninguém pode te dizer como você vai fazer o melhor uso do JS; isso é você quem decide. Estou apenas tentando te empoderar para chegar às suas próprias conclusões, quaisquer que sejam.

Por outro lado, há um *grão* ao qual você realmente deveria prestar atenção e seguir: o *grão* de como o JS funciona, no nível da linguagem. Há coisas que funcionam bem e naturalmente em JS, dada a prática e a abordagem certas, e há coisas que você realmente não deveria tentar fazer na linguagem.

Você consegue fazer seu programa JS parecer um programa Java, C# ou Perl? E que tal Python ou Ruby, ou até PHP? Em graus variados, claro que consegue. Mas deveria?

Não, eu não acho que deveria. Acho que você deveria aprender e abraçar o jeito JS, e tornar seus programas JS o mais "JS'ísticos" quanto for prático. Alguns vão achar que isso significa programação desleixada e informal, mas não é nada disso que quero dizer. Só quero dizer que o JS tem muitos padrões e idiomas reconhecivelmente "JS", e ir com esse *grão* é o caminho geral para o melhor sucesso.

Por fim, talvez o *grão* mais importante de reconhecer seja como o(s) programa(s) existente(s) em que você trabalha, e os desenvolvedores com quem você trabalha, fazem as coisas. Não leia estes livros e então tente mudar *todo esse grão* nos seus projetos existentes de uma noite para a outra. Essa abordagem sempre vai falhar.

Você vai ter que mudar essas coisas pouco a pouco, ao longo do tempo. Trabalhe para construir consenso com seus colegas desenvolvedores sobre por que é importante revisitar e reconsiderar uma abordagem. Mas faça isso com apenas um pequeno tópico por vez, e deixe que comparações de código "antes e depois" falem a maior parte. Reúna todo mundo do time para discutir e defenda decisões baseadas em análise e evidência do código, em vez da inércia do "nossos devs seniores sempre fizeram assim".

Esse é o conselho mais importante que posso transmitir para te ajudar a aprender JS. Sempre continue procurando maneiras melhores de usar o que o JS nos dá para escrever código mais legível. Todo mundo que trabalhar no seu código, incluindo seu eu do futuro, vai te agradecer!

## Em Ordem

Agora você tem uma perspectiva mais ampla do que resta explorar em JS e a atitude certa para encarar o restante da sua jornada.

Mas uma das perguntas práticas mais comuns que recebo a esta altura é: "Em que ordem devo ler os livros?" Há uma resposta direta... mas também depende.

Minha sugestão para a maioria dos leitores é seguir esta série nesta ordem:

1. Comece com uma base sólida de JS em *Get Started* (Livro 1) — boa notícia, você já quase terminou este livro!

2. Em *Scope & Closures* (Livro 2), aprofunde-se no primeiro pilar do JS: escopo léxico, como isso sustenta closure e como o padrão de módulo organiza o código.

3. Em *Objects & Classes* (Livro 3), foque no segundo pilar do JS: como o `this` do JS funciona, como protótipos de objeto sustentam a delegação e como protótipos habilitam o mecanismo `class` para organização de código em estilo OO.

4. Em *Types & Grammar* (Livro 4), encare o terceiro e último pilar do JS: tipos e coerção de tipos, além de como a sintaxe e a gramática do JS definem como escrevemos nosso código.

5. Com os **três pilares** solidamente no lugar, *Sync & Async* (Livro 5) explora como usamos o controle de fluxo para modelar mudança de estado nos nossos programas, tanto de forma síncrona (imediatamente) quanto assíncrona (ao longo do tempo).

6. A série se encerra com *ES.Next & Beyond* (Livro 6), um olhar adiante sobre o futuro próximo e de médio prazo do JS, incluindo uma variedade de recursos que provavelmente chegarão aos seus programas JS em breve.

Essa é a ordem pretendida para ler esta série de livros.

Porém, os Livros 2, 3 e 4 podem, em geral, ser lidos em qualquer ordem, dependendo de qual tópico você sente mais curiosidade e conforto para explorar primeiro. Mas não recomendo pular nenhum desses três livros — nem mesmo *Types & Grammar*, como alguns de vocês vão ficar tentados a fazer! — mesmo que você ache que já domina aquele tópico.

O Livro 5 (*Sync & Async*) é crucial para entender profundamente o JS, mas, se você começar a se aprofundar e achar intimidante demais, este livro pode ser adiado até você estar mais experiente com a linguagem. Quanto mais JS você tiver escrito (e sofrido!), mais você vai passar a apreciar este livro. Então não tenha medo de voltar a ele mais tarde.

O último livro da série, *ES.Next & Beyond*, em alguns aspectos é independente. Ele pode ser lido no fim, como sugiro, ou logo depois de *Get Started*, se você estiver procurando um atalho para ampliar seu radar sobre o que é o JS. Este livro também tem mais chance de receber atualizações no futuro, então você provavelmente vai querer revisitá-lo de tempos em tempos.

Seja como for que você escolha prosseguir com a YDKJSY, confira primeiro os apêndices deste livro, especialmente praticando os trechos do Apêndice B, "Prática, Prática, Prática!". Eu mencionei que você deveria ir praticar!? Não há forma melhor de aprender código do que escrevendo-o.
