# Capítulo 1: O que estamos fazendo ?

## Introdução

Olá eu sou o Professor Franklin Risby, prazer em conhece-lo! Iremos passar um tempo juntos para que eu lhe ensine um pouco sobre programação funcional. Eu já falei sobre mim, me conte sobre você! Espero que você já esteja familiarizado com JavaScript, tenha um pouco de experiência com Orientação a Objetos and fancy yourself a working class programmer. Você não precisa ter um PHD em Entomologia, você apenas precisa saber como encontrar e matar alguns ``Bugs``.

Não suponho que você tenha algum conhecimento anterior com programação funcional, mas suponho que tenha conhecimento de passar por situações desfavoráveis que aparecem quando trabalhamos com estados mutáveis, efeitos colaterais irrestritos e unprincipled design. Agora que estamos apresentados, vamos começar.


O propósito deste capítulo é lhe mostrar aquilo que procuramos quando escrevemos programas funcionais. Devemos ter a idéia sobre o que faz um programa ser **funcional**, ou iremos apenas escrever código a toa sem rumo algum, inclusive evitando a todo custo objetos - Um esforço inútil. We need a bullseye to hurl our code toward, some celestial compass for when the waters get rough.


Existem alguns principios de programação, vários credos que nos guia através dos vales sombrios de qualquer aplicação: DRY (don't repeat yourself), loose coupling high cohesion, YAGNI (ya ain't gonna need it), principle of least surprise, única responsabilidade, e assim por diante.

Não irei abusar usando cada princípio que tenho visto em todos esses anos... o caso
é que eles se aplicam no ambiente funcional, mesmo sendo de uma forma superficial para aquilo que queremos.
What I'd like you to get a feel for now, before we get any further, is our intention when we poke and prod at the keyboard; our functional Xanadu.

<!--BREAK-->

## Um encontro casual

Vamos começar com um toque de insanidade.

Let's start with a touch of insanity. Here is a seagull application. When flocks conjoin they become a larger flock and when they breed they increase by the number of seagulls with whom they're breeding. Now this is not intended to be good Object-Oriented code, mind you, it is here to highlight the perils of our modern, assignment based approach. Behold:

```js
var Flock = function(n) {
  this.seagulls = n;
};

Flock.prototype.conjoin = function(other) {
  this.seagulls += other.seagulls;
  return this;
};

Flock.prototype.breed = function(other) {
  this.seagulls = this.seagulls * other.seagulls;
  return this;
};

var flock_a = new Flock(4);
var flock_b = new Flock(2);
var flock_c = new Flock(0);

var result = flock_a.conjoin(flock_c)
    .breed(flock_b).conjoin(flock_a.breed(flock_b)).seagulls;
//=> 32
```

Who on earth would craft such a ghastly abomination? It is unreasonably difficult to keep track of the mutating internal state. And, good heavens, the answer is even incorrect! It should have been `16`, but `flock_a` wound up permanently altered in the process. Poor `flock_a`. This is anarchy in the I.T.! This is wild animal arithmetic!

Se você não entende este programa, tudo bem, eu também não! O ponto é que *estado* e *valores mutáveis* são difíceis de serem acompanhados mesmo em pequenos exemplos.

Vamos tentar novamente com uma abordagem funcional:

```js
var conjoin = function(flock_x, flock_y) { return flock_x + flock_y };
var breed = function(flock_x, flock_y) { return flock_x * flock_y };

var flock_a = 4;
var flock_b = 2;
var flock_c = 0;

var result = conjoin(
  breed(flock_b, conjoin(flock_a, flock_c)), breed(flock_a, flock_b)
);
//=>16
```

Bem, dessa vez temos o valor certo. Temos muito menos código. *function nesting* é um pouco confuso...[Vamos remediar isso no capítulo5]. Dessa forma é melhor, mas vamos mais fundo. Existem benefícios em chamar um por um. Had we done so, we might have seen we're just working with simple addition (`conjoin`) and multiplication (`breed`).

Não há nada especial nessas duas funções além de seus nomes. Vamos renomea-las para revelar sua verdadeira indentidade.

```js
var add = function(x, y) { return x + y };
var multiply = function(x, y) { return x * y };

var flock_a = 4;
var flock_b = 2;
var flock_c = 0;

var result = add(
  multiply(flock_b, add(flock_a, flock_c)), multiply(flock_a, flock_b)
);
//=>16
```
E assim, nós ganhamos o conhecimento dos antigos:

```js
// associative
add(add(x, y), z) == add(x, add(y, z));

// commutative
add(x, y) == add(y, x);

// identity
add(x, 0) == x;

// distributive
multiply(x, add(y,z)) == add(multiply(x, y), multiply(x, z));
```

Ah yes, those old faithful mathematical properties should come in handy. Don't worry if you didn't know them right off the top of your head. For a lot of us, it's been a while since we've reviewed this information. Let's see if we can use these properties to simplify our little seagull program.

```js
// Original line
add(multiply(flock_b, add(flock_a, flock_c)), multiply(flock_a, flock_b));

// Apply the identity property to remove the extra add
// (add(flock_a, flock_c) == flock_a)
add(multiply(flock_b, flock_a), multiply(flock_a, flock_b));

// Apply distributive property to achieve our result
multiply(flock_b, add(flock_a, flock_a));
```

Brilliant! We didn't have to write a lick of custom code other than our calling function. We include `add` and `multiply` definitions here for completeness, but there is really no need to write them - we surely have an `add` and `multiply` provided by some previously written library.

You may be thinking "how very strawman of you to put such a mathy example up front". Or "real programs are not this simple and cannot be reasoned about in such a way". I've chosen this example because most of us already know about addition and multiplication so it's easy to see how math can be of use to us here.

Don't despair, throughout this book, we'll sprinkle in some category theory, set theory, and lambda calculus to write real world examples that achieve the same simplicity and results as our flock of seagulls example. You needn't be a mathematician either, it will feel just like using a normal framework or api.

It may come as a surprise to hear that we can write full, everyday applications along the lines of the functional analog above. Programs that have sound properties. Programs that are terse, yet easy to reason about. Programs that don't reinvent the wheel at every turn. Lawlessness is good if you're a criminal, but in this book, we'll want to acknowledge and obey the laws of math.

We'll want to use the theory where every piece tends to fit together so politely. We'll want to represent our specific problem in terms of generic, composable bits and then exploit their properties for our own selfish benefit. It will take a bit more discipline than the "anything goes" approach of imperative[^We'll go over the precise definition of imperative later in the book, but for now it's anything other than functional programming] programming, but the payoff of working within a principled, mathematical framework will astound you.

We've seen a flicker of our functional north star, but there are a few concrete concepts to grasp before we can really begin our journey.

[Chapter 2: First Class Functions](ch2.md)
