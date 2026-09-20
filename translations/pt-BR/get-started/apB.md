# You Don't Know JS Yet: Get Started - 2ª Edição
# Apêndice B: Prática, Prática, Prática!

Neste apêndice, vamos explorar alguns exercícios e suas soluções sugeridas. Eles servem apenas para *te fazer começar* a praticar os conceitos do livro.

## Praticando Comparações

Vamos praticar o trabalho com tipos de valor e comparações (Capítulo 4, Pilar 3), em que a coerção precisará estar envolvida.

`scheduleMeeting(..)` deve receber um horário de início (no formato de 24 horas, como a string "hh:mm") e a duração da reunião (número de minutos). Deve retornar `true` se a reunião couber inteiramente dentro do expediente (de acordo com os horários especificados em `dayStart` e `dayEnd`); retornar `false` se a reunião violar os limites do expediente.

```js
const dayStart = "07:30";
const dayEnd = "17:45";

function scheduleMeeting(startTime,durationMinutes) {
    // ..TODO..
}

scheduleMeeting("7:00",15);     // false
scheduleMeeting("07:15",30);    // false
scheduleMeeting("7:30",30);     // true
scheduleMeeting("11:30",60);    // true
scheduleMeeting("17:00",45);    // true
scheduleMeeting("17:30",30);    // false
scheduleMeeting("18:00",15);    // false
```

Tente resolver isso sozinho primeiro. Considere o uso dos operadores de igualdade e de comparação relacional, e como a coerção impacta este código. Quando você tiver um código que funcione, *compare* sua(s) solução(ões) com o código em "Soluções Sugeridas", no fim deste apêndice.

## Praticando Closure

Agora vamos praticar closure (Capítulo 4, Pilar 1).

A função `range(..)` recebe um número como primeiro argumento, representando o primeiro número de um intervalo desejado de números. O segundo argumento também é um número, representando o fim do intervalo desejado (inclusive). Se o segundo argumento for omitido, então deve ser retornada outra função que espera esse argumento.

```js
function range(start,end) {
    // ..TODO..
}

range(3,3);    // [3]
range(3,8);    // [3,4,5,6,7,8]
range(3,0);    // []

var start3 = range(3);
var start4 = range(4);

start3(3);     // [3]
start3(8);     // [3,4,5,6,7,8]
start3(0);     // []

start4(6);     // [4,5,6]
```

Tente resolver isso sozinho primeiro.

Quando você tiver um código que funcione, *compare* sua(s) solução(ões) com o código em "Soluções Sugeridas", no fim deste apêndice.

## Praticando Protótipos

Por fim, vamos trabalhar com `this` e objetos vinculados por protótipo (Capítulo 4, Pilar 2).

Defina um caça-níqueis com três rolos que podem girar individualmente com `spin()` e, depois, exibir com `display()` o conteúdo atual de todos os rolos.

O comportamento básico de um único rolo está definido no objeto `reel` abaixo. Mas o caça-níqueis precisa de rolos individuais — objetos que delegam para `reel` e que têm, cada um, uma propriedade `position`.

Um rolo só *sabe como* exibir (`display()`) o símbolo da sua casa atual, mas um caça-níqueis tipicamente mostra três símbolos por rolo: a casa atual (`position`), uma casa acima (`position - 1`) e uma casa abaixo (`position + 1`). Então exibir o caça-níqueis deve acabar mostrando uma grade 3 x 3 de símbolos.

```js
function randMax(max) {
    return Math.trunc(1E9 * Math.random()) % max;
}

var reel = {
    symbols: [
        "♠", "♥", "♦", "♣", "☺", "★", "☾", "☀"
    ],
    spin() {
        if (this.position == null) {
            this.position = randMax(
                this.symbols.length - 1
            );
        }
        this.position = (
            this.position + 100 + randMax(100)
        ) % this.symbols.length;
    },
    display() {
        if (this.position == null) {
            this.position = randMax(
                this.symbols.length - 1
            );
        }
        return this.symbols[this.position];
    }
};

var slotMachine = {
    reels: [
        // este caça-níqueis precisa de 3 rolos separados
        // dica: Object.create(..)
    ],
    spin() {
        this.reels.forEach(function spinReel(reel){
            reel.spin();
        });
    },
    display() {
        // TODO
    }
};

slotMachine.spin();
slotMachine.display();
// ☾ | ☀ | ★
// ☀ | ♠ | ☾
// ♠ | ♥ | ☀

slotMachine.spin();
slotMachine.display();
// ♦ | ♠ | ♣
// ♣ | ♥ | ☺
// ☺ | ♦ | ★
```

Tente resolver isso sozinho primeiro.

Dicas:

* Use o operador de módulo `%` para dar a volta (*wrap*) em `position` ao acessar os símbolos circularmente em um rolo.

* Use `Object.create(..)` para criar um objeto e vinculá-lo por protótipo a outro objeto. Uma vez vinculados, a delegação permite que os objetos compartilhem o contexto `this` durante a invocação de métodos.

* Em vez de modificar o objeto reel diretamente para mostrar cada uma das três posições, você pode usar outro objeto temporário (`Object.create(..)` de novo), com sua própria `position`, para delegar a partir dele.

Quando você tiver um código que funcione, *compare* sua(s) solução(ões) com o código em "Soluções Sugeridas", no fim deste apêndice.

## Soluções Sugeridas

Tenha em mente que essas soluções sugeridas são apenas isso: sugestões. Há muitas formas diferentes de resolver estes exercícios de prática. Compare sua abordagem com o que você vê aqui e considere os prós e contras de cada uma.

Solução sugerida para a prática de "Comparações" (Pilar 3):

```js
const dayStart = "07:30";
const dayEnd = "17:45";

function scheduleMeeting(startTime,durationMinutes) {
    var [ , meetingStartHour, meetingStartMinutes ] =
        startTime.match(/^(\d{1,2}):(\d{2})$/) || [];

    durationMinutes = Number(durationMinutes);

    if (
        typeof meetingStartHour == "string" &&
        typeof meetingStartMinutes == "string"
    ) {
        let durationHours =
            Math.floor(durationMinutes / 60);
        durationMinutes =
            durationMinutes - (durationHours * 60);
        let meetingEndHour =
            Number(meetingStartHour) + durationHours;
        let meetingEndMinutes =
            Number(meetingStartMinutes) +
            durationMinutes;

        if (meetingEndMinutes >= 60) {
            meetingEndHour = meetingEndHour + 1;
            meetingEndMinutes =
                meetingEndMinutes - 60;
        }

        // recompõe as strings de horário completas
        // (para facilitar a comparação)
        let meetingStart = `${
            meetingStartHour.padStart(2,"0")
        }:${
            meetingStartMinutes.padStart(2,"0")
        }`;
        let meetingEnd = `${
            String(meetingEndHour).padStart(2,"0")
        }:${
            String(meetingEndMinutes).padStart(2,"0")
        }`;

        // NOTA: como as expressões são todas strings,
        // as comparações aqui são alfabéticas, mas
        // é seguro neste caso porque são strings de
        // horário completas (ex.: "07:15" < "07:30")
        return (
            meetingStart >= dayStart &&
            meetingEnd <= dayEnd
        );
    }

    return false;
}

scheduleMeeting("7:00",15);     // false
scheduleMeeting("07:15",30);    // false
scheduleMeeting("7:30",30);     // true
scheduleMeeting("11:30",60);    // true
scheduleMeeting("17:00",45);    // true
scheduleMeeting("17:30",30);    // false
scheduleMeeting("18:00",15);    // false
```

----

Solução sugerida para a prática de "Closure" (Pilar 1):

```js
function range(start,end) {
    start = Number(start) || 0;

    if (end === undefined) {
        return function getEnd(end) {
            return getRange(start,end);
        };
    }
    else {
        end = Number(end) || 0;
        return getRange(start,end);
    }


    // **********************

    function getRange(start,end) {
        var ret = [];
        for (let i = start; i <= end; i++) {
            ret.push(i);
        }
        return ret;
    }
}

range(3,3);    // [3]
range(3,8);    // [3,4,5,6,7,8]
range(3,0);    // []

var start3 = range(3);
var start4 = range(4);

start3(3);     // [3]
start3(8);     // [3,4,5,6,7,8]
start3(0);     // []

start4(6);     // [4,5,6]
```

----

Solução sugerida para a prática de "Protótipos" (Pilar 2):

```js
function randMax(max) {
    return Math.trunc(1E9 * Math.random()) % max;
}

var reel = {
    symbols: [
        "♠", "♥", "♦", "♣", "☺", "★", "☾", "☀"
    ],
    spin() {
        if (this.position == null) {
            this.position = randMax(
                this.symbols.length - 1
            );
        }
        this.position = (
            this.position + 100 + randMax(100)
        ) % this.symbols.length;
    },
    display() {
        if (this.position == null) {
            this.position = randMax(
                this.symbols.length - 1
            );
        }
        return this.symbols[this.position];
    }
};

var slotMachine = {
    reels: [
        Object.create(reel),
        Object.create(reel),
        Object.create(reel)
    ],
    spin() {
        this.reels.forEach(function spinReel(reel){
            reel.spin();
        });
    },
    display() {
        var lines = [];

        // exibe todas as 3 linhas do caça-níqueis
        for (
            let linePos = -1; linePos <= 1; linePos++
        ) {
            let line = this.reels.map(
                function getSlot(reel){
                    var slot = Object.create(reel);
                    slot.position = (
                        reel.symbols.length +
                        reel.position +
                        linePos
                    ) % reel.symbols.length;
                    return slot.display();
                }
            );
            lines.push(line.join(" | "));
        }

        return lines.join("\n");
    }
};

slotMachine.spin();
slotMachine.display();
// ☾ | ☀ | ★
// ☀ | ♠ | ☾
// ♠ | ♥ | ☀

slotMachine.spin();
slotMachine.display();
// ♦ | ♠ | ♣
// ♣ | ♥ | ☺
// ☺ | ♦ | ★
```

É isso para este livro. Mas agora é hora de procurar projetos reais para praticar estas ideias. Continue codando, porque essa é a melhor forma de aprender!
