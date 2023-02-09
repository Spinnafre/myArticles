# Call Stack e Memory Heap

É muito comum quando se trabalha com Objetos e Arrays no Javascript, passar por problemas de referências em memória, no qual mudar um objeto altera outro magicamente, e às vezes você encontra muitas soluções em bibliotecas que realizam cópia de objetos e mágicamente os problemas somem. 

Mas quantas vezes você teve que na mão mesmo invocar o spread operator para clonar um objeto mas não funcionava em objetos encadeados? Isso é um problema meio bobo mas que causa muitos problemas se não for prevenido com cuidado.

# Call Stack

> A melhor forma de saber se uma linguagem é single thread é ver se só tem uma call stack.
> 

Pilhas de operações que armazenam sequências de ações que o programa vai executar. Essas operações irão ser adicionadas na estrutura de dados STACK (pilha) e irão removê-las à medida em que o código for sendo executado. Ou seja, de forma mais curta podemos dizer que Call Stack é uma pilha de execuções de funções.

Essa pilha é basicamente uma estrutura que armazena a chave como endereço da memória e possui como valor uma referência para um tipo primitivo ou um apontamento para outro endereço da memória.

<aside>
🤔 Call stack guarda dados somente do tipo primitivo (int,bigint,string...), organiza a ordem de chamada das funções e armazena valores.

</aside>

<aside>
⚠️ No javascript tipos primitivos são imutáveis (*string, number, bigint, boolean, undefined, and symbol)*

</aside>

### Vamos pensar um pouco 🤔

O que será impresso abaixo será 2 ou 1?

```jsx
let n1=1
let n2=n1 
n1=n1+1
console.log(n2) //???
```

No exemplo acima, quando cria uma variável do tipo primitivo - lembrando que no Javascript todos os tipos primitivos são imutáveis - ao adicionar o valor da soma na variável `n1`, ao invés de substituir o valor na memória irá criar uma nova referência com o valor da soma (2) e como `n2` recebia a referência da memória de n1 e como vimos que ela irá ter outra nova referência, então `n2` não irá ser atualizado. Ou seja:

```jsx
let n1=1
let n2=n1 // n2 não irá ser 2 e sim 1 pois tipos primitivos são imutáveis
n1=n1+1
console.log(n2) //Não irá ser igual a 2
```

Esse comportamento já é diferente quando trabalhamos com objetos. 

No exemplo abaixo, foi criado um objeto `p2` que recebe outro objeto `p1` , ao modificar `p1` irá ocasionar na mudança também na variável `p2`. Esse é um problema que é muito fácil de replicar quando se trabalha com objetos, e a dor de cabeça que isso pode dar é grande.

```jsx
let p1 = {
    id:1,
    name:'Davi'
}

let p2 = p1

p1.name = 'John Wick'

console.log("P1 =>> ",p1)
console.log("P2 =>> ",p2)

/*
	[LOG]: "P1 =>> ",  {
	  "id": 1,
	  "name": "John Wick"
	} 
	[LOG]: "P2 =>> ",  {
	  "id": 1,
	  "name": "John Wick"
	}
*/
```

Para quem já trabalhou com desenvolvimento web usando tecnologias como **React** ou **Vue**, já está familiarizado com conceitos de **Reatividade**, bem, acima meio que fizemos uma abordagem de reatividade sem querer no qual ao mudar qualquer um dos objetos irá alterar todas as variáveis. 😎

Umas das diversas alternativas para contornar esse problema é realizar *Deep Copy* que é o processo de clonar um objeto por completo, criando uma nova referência na memória e assim evitando a *reatividade*.

```jsx
let p1 = {
    id:1,
    name:'Davi'
}
let p2 = Object.assign({},p1)

p1.name = 'John Wick'

console.log("P1 =>> ",p1)
console.log("P2 =>> ",p2)

p2.name="Bruce Dickinson"

console.log("P1 =>> ",p1)
console.log("P2 =>> ",p2)
```

Outra alternativa é usar o *spread operator* para realizar uma *****Shallow Copy***** que resumidamente é copiar os atributos principais do objeto. Muito eficiente quando se trabalha com objetos simples, ou seja, não possuem encadeamento de dados dentro de uma mesma estrutura.

```jsx
let p1 = {
    id:1,
    name:'Davi',
    log(){
        console.log(this.name)
    }
}
let p2 = {...p1}
p1.name = 'John Wick'

p1.log() // [LOG]: "John Wick"
p2.log() // [LOG]: "Davi"

p2.name="Bruce Dickinson"

p1.log() //[LOG]: "John Wick"
p2.log() // [LOG]: "Bruce Dickinson"
```

Mas *****Shallow Copy***** pode ser um problema quando se trabalha com objetos encadeados, como mostrado no exemplo abaixo:

```jsx
let p1 = {
    id:1,
    name:'Davi',
    getName(){
        return this.name
    },
    parent:{
        id:33,
        name:'Thanos',
        getName(){
            return this.name
        },
    }
}
let p2 = {...p1}
p1.name = 'John Wick'

console.log("[P1] : ",p1.getName() ) // [LOG]: "[P1] : ",  "John Wick"

console.log("[P2] : ",p2.getName()) // [LOG]: "[P2] : ",  "Davi" 
 
p2.name="Bruce Dickinson"
p2.parent.name = "Marshall Bruce Mathers III"

console.log("[P1] : ", p1.getName() ) // [LOG]: "[P1] : ",  "John Wick" 

console.log("[P2] : ",p2.getName() ) // [LOG]: "[P2] : ",  "Bruce Dickinson" 

console.log("[P1] - PARENT] : ",p1.parent.getName()) // [LOG]: "[P1] - PARENT] : ",  "Marshall Bruce Mathers III"

console.log("[P2] - PARENT] : ",p2.parent.getName()) // [LOG]: "[P2] - PARENT] : ",  "Marshall Bruce Mathers III"
```

Aí nesses caso, o ideal é usar o ***Deep Copy*** que irá copiar todos os métodos e propriedades da variável `p1` para uma nova referência, fazendo com que ao mudar `p2` não irá alterar `p1` e vice-versa.

```jsx
let p1 = {
    id:1,
    name:'Davi',
    getName(){
        return this.name
    },
    parent:{
        id:33,
        name:'Thanos',
        getName(){
            return this.name
        },
    }
}

// Função recursiva personalizada que irá realizar o deep copy
function deepClone(obj) {
    let newObj = {};

    for (let key in obj) {
        let val = obj[key];

        if (val instanceof Array) {
            newObj[key] = [...val]
        } else if (typeof val === 'object') {
            newObj[key] = deepClone(val)
        } else {
            newObj[key] = val;
        }
    }
    return newObj;
}

let p2 = deepClone(p1)

p1.name = 'John Wick'

console.log("[P1] : ",p1.getName() ) // [LOG]: "[P1] : ",  "John Wick"

console.log("[P2] : ",p2.getName()) // [LOG]: "[P2] : ",  "Davi" 
 
p2.name="Bruce Dickinson"
p2.parent.name = "Marshall Bruce Mathers III"

console.log("[P1] : ", p1.getName() ) // [LOG]: "[P1] : ",  "John Wick" 

console.log("[P2] : ",p2.getName() ) // [LOG]: "[P2] : ",  "Bruce Dickinson" 

console.log("[P1] - PARENT] : ",p1.parent.getName()) // [LOG]: "[P1] - PARENT] : ",  "Thanos" 

console.log("[P2] - PARENT] : ",p2.parent.getName()) // [LOG]: "[P2] - PARENT] : ",  "Marshall Bruce Mathers III"
```

Eu poderia fazer o deep clone de diversas maneira, como usando a biblioteca **lodash** que abstrai muitas coisas e facilita a nossa vida, mas você deve ter já visto por aí pessoas criando cópias profundas de objetos através do **JSON.stringify** junto com **JSON. parse,** porém isso só é possível se o objeto for capaz de ser serializado. Como assim?

Usar essa técnica funciona muito ao trabalhar com objetos que só possuem atributos, por exemplo:

```jsx
let p1 = {
    id:1,
    name:'Davi'
}

let p2 = JSON.parse(JSON.stringify(p1))

p1.name = 'John Wick'

console.log("[P1] - ",p1) 
/*
    [LOG]: "P1 =>> ",  {
    "id": 1,
    "name": "John Wick"
    } 
*/

console.log("[P2] - ",p2)
/*
    [LOG]: "P2 =>> ",  {
    "id": 1,
    "name": "Davi"
    } 
*/

p2.name="Bruce Dickinson"

console.log("[P1] - ",p1)
/*
    [LOG]: "P1 =>> ",  {
    "id": 1,
    "name": "John Wick"
    } 
*/

console.log("[P2] - ",p2)
/*
    [LOG]: "P2 =>> ",  {
    "id": 1,
    "name": "Bruce Dickinson"
    } 
*/
```

Tudo feito e ok até agora, certo? De fato funcionou muito bem a cópia usando serialização do objeto por meio do `JSON.stringify`.

<aside>
💡 Serialização é o processo de traduzir uma estrutura de dados para um formato adequado para ser transferido, manipulado ou armazenado.

</aside>

Porém vamos ver um caso em que essa técnica não irá funcionar, no caso irei usar um mesmo exemplo já citado acima:

```jsx
let p1 = {
    id:1,
    name:'Davi',
    getName(){
        return this.name
    },
    parent:{
        id:33,
        name:'Thanos',
        getName(){
            return this.name
        },
    }
}

let p2 = JSON.parse(JSON.stringify(p1))

p1.name = 'John Wick'

console.log("[P1] : ",p1.getName() ) // [LOG]: "[P1] : ",  "John Wick"

console.log("[P2] : ",p2.getName())  // [ERR]: p2.getName is not a function
 
p2.name="Bruce Dickinson"
p2.parent.name = "Marshall Bruce Mathers III"

console.log("[P1] : ", p1.getName() ) 

console.log("[P2] : ",p2.getName() ) 

console.log("[P1] - PARENT] : ",p1.parent.getName()) 

console.log("[P2] - PARENT] : ",p2.parent.getName())
```

Perceba que como o meu objeto possui um método, irá causar problemas na serialização, o que mostra que essa estratégia não é boa para trabalhar com funções, null, undefined e entre outros.

Um ponto importante a destacar é que *Shallow Copy* é muito mais performática do que *Deep Copy*, sendo ideal para casos em que você deseja fazer uma cópia de um objeto sem ser necessário levar em consideração objetos aninhados dentro do mesmo.

Para evitar correr riscos de atribuir em uma variável um objeto já existente sem fazer algumas das cópias apresentadas, é melhor trabalhar com o `const` pois não permite você alterar o endereço da memória de uma variável

# Memory Heap

> Onde os tipos não primitivos são armazenados
> 

Guarda endereços de memórias que podem serem apontados pela a *Call Stack* para trabalhar com dados do tipo referências (Objetos, Array, Funções...)

<aside>
🤔 Pilha para guardar dados de tipos de referências como objetos, funções, arrays, entre outros.

</aside>

Vamos para exemplos práticos, nada de ficar preso em teoria demais !

```jsx
const name = "Mr_Spin"
const nick = name 
const items_ids = [1,2]
const items_ids_copy = items_ids
```

O código acima poderá ser visto de maneira mais ilustrativa, algo semelhante como a imagem abaixo. Notem que `items_ids` e `items_ids_copy` possuem referência para o mesmo array na *memory heap*.

![Untitled](imgs/Untitled%202.png)

Sempre que cria uma valor do tipo não primitivo, o Javascript irá alocar o valor na *Memory Heap* com um identificador único da memória e irá ligá-lo com valor na *call stack*. Ou seja, cria um endereço na memória na *call stack* e o valor irá ser a referência do endereço da memória da *Memory Heap* que irá possuir o tipo não primitivo.

Quando fazemos alguma operação `push` no array, não irá mudar o endereço da memória da call stack, mas o valor na Memory Heap irá ser alterado. 

### Vamos analisar mais códigos para juntar as peças

```jsx
const obj1={counter:1}
const obj2=obj1
obj2.counter++
// obj2 => {counter:2}
// obj1 => {counter:2}
```

Note que como estamos trabalhando com objetos que são tipos não primitivos, estamos criando uma referência na memória na *memory heap* e o valor será adicionado lá. 

Porém, ao adicionarmos em uma nova variável um valor que irá ser a variável já criada, irá acabar criando um novo endereço na memória na *call stack* e para essa nova variável o valor irá apontar para a mesma referência já criada na *memory heap*, ou seja, embora sejam variáveis diferentes na call stack a referência na *memory heap* será a mesma.

O exemplo anterior é muito usado para indicar o compartilhamento de referência na memória (*Memory Heap*) em endereços de memória diferentes na *call stack*.

Para evitar ter problemas em duas variáveis diferentes apontarem para o mesmo endereço da memória existe o `Object.Create` ou outras técnicas de `deep copy` que criam novo endereço na *memory heap,* como já foi apresentado já no tópico sobre ***Call Stack***.

![Altera tanto o `obj1` como o `obj2` irá ocasionar na modificação na mesma referência da memória do objeto na *memory heap*.](imgs/Untitled%201.png)

Alterar tanto o `obj1` como o `obj2` irá ocasionar na modificação na mesma referência da memória do objeto na *memory heap*.


Feito com ❤️ por Davi Silva ou conhecido como [Spinnafre](https://github.com/Spinnafre) 😉

## Referências

[Javascript Fundamentals - Call Stack and Memory Heap](https://medium.com/@allansendagi/javascript-fundamentals-call-stack-and-memory-heap-401eb8713204)

[Understanding Call Stack and Heap Memory in JS](https://levelup.gitconnected.com/understanding-call-stack-and-heap-memory-in-js-e34bf8d3c3a4)

[Understanding the size of an object in Chrome/V8](https://www.mattzeunert.com/2017/03/29/v8-object-size.html)


