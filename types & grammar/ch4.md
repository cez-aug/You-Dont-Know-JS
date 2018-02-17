# You Don't Know JS: Tipos e Gramática
# Capitulo 4: Coerção

Agora que nós entendemos melhor os tipos e valores do JavaScript, vamos voltar nossa atenção para um tópico muito controverso: coerção.

Como mencionado no Capítulo 1, os debates sobre se coerção é um recurso útil ou uma falha no design da linguagem (ou algo no meio disso!) têm ocorrido desde o primeiro dia. Se você já leu outros livros sobre JS, você sabe que a *mensagem* prevalentemente esmagadora é que coerção é mágica, má, confusa, e francamente uma ideia ruim.

Seguindo o mesmo espírito dessa série de livros, ao invés de fugir da coerção porque todo mundo faz isso, ou porque você tem algum capricho, eu acho que você deve seguir em frente e procurar entender mais claramente.

Nosso objetivo é entender plenamente os prós e contras (sim, *existem* prós!) da coerção, então você poderá estar bem informado para tomar a decisão de como ela se adequa ao seu programa.

## Convertendo Valores

Converter um valor de um tipo para outro é geralmente chamado de "type casting", quando explicitamente, e "coerção" quando feito implicitamente (forçado pelas regras de como um valor é utilizado).

**Nota:** Pode não ser óbvio, mas as coerções do JavaScript sempre resultam em algum valor dos primitivos escalares (veja Capítulo 2), como `string`, `number`, ou `boolean`. Não existe coerção que resulte em um valor complexo como `object` ou `function`. O Capítulo 3 aborda "boxing", que envolve os valores primitivos em seus `object` homólogos, mas isso não é realmente coerção em seu sentido correto.

Pode-se distinguir esses ainda de outra maneira: "type casting" (ou coversão de tipos) ocorre em linguagem staticamente tipadas em tempo de compilação, enquanto "coerção" é uma conversão que ocorre em tempo de execução em linguagens dinamicamente tipadas.

Entretanto, em JavaScript, a maioria das pessoas chamam todos esse tipos de *coerção*, por isso a maneira que prefiro chamar de "coerção implícita" vs. "coerção explícita".

A diferença deve ser óbvia: "coerção explícita" é quando consegue-se olhar para o código e ver que a coversão de tipos está ocorrendo intencionalmente, enquanto que a "coerção implícita" é quando o conversão de tipos ocorre de maneira óbvia como um efeito colateral de outra operação intencional.

Por exemplo, considere essas duas abordagens de coerção:

```js
var a = 42;

var b = a + "";			// coerção implícita

var c = String( a );	// coerção explícita
```

Para `b`, a coerção ocorre implicitamente, porque o operador `+` combinado com um dos operandos sendo uma `string` (`""`) vai insistir que a operação seja uma concatenação de strings (juntando duas strings), que, *como um efeito colateral (oculto)* vai forçar o valor `42` em `a` a ser convertido a seu equivalente em `string`: `"42"`.

Em contrapartida, a função `String(..)` torna bastante óbvio que o valor de `a` é convertido em sua representação em `string`.

As duas abordagens têm o mesmo efeito: `42` vira `"42"`. Mas é o *como* que é o coração dos debates mais acalorados sobre coerção no JavaScript.

**Nota:** Tecnicamente, há pequenas nuances no comportamento que vão além da estética. Nós abordaremos isso em mais detalhes mais tarde neste capítulo, na seção "Implicitamente: Strings <--> Number".

Os termos "explícito" e "implícito", ou "óbvio" e "efeito colateral oculto", são *relativos*.

Se você sabe exatamente o que `a + ""` está fazendo e você está intencionalmente convertendo o valor para uma `string`, você pode achar a operção suficientemente "explícita". Por outro lado, se você nunca viu a função `String(..)` usada para coerção de strings, o seu comportamento pode parecer oculto o suficiente para ser "implícito" para você.

Mas nós estamos discutindo "explícito" vs. "implícito" baseados nas prováveis opiniões de um desenvolvedor *mediano, razoavelmente informado, mas não especialista ou devoto pela especificação do JS*. Se você se encaixa de alguma forma nesse grupo, você terá de ajustar a sua perspectiva para estar de acordo com as nossas observações.

Relembrando: é pouco provável que após escrevermos o nosso código, nós sejamos os únicos que vão lê-lo. Mesmo que você seja um expert em todos os prós e contras no JS, considere como um colega de trabalho com menos experiência vai ser se sentir quando ler o seu código. Vai ser "explícito" ou "implícito" para eles da mesma maneira que é para você?

## Operações de valor abstrato

Antes de explorarmos a coerção *explícita* e *implícita*, nós precisamos aprender as regras básicas que governam como os valores *tornam-se* uma `string`, `number` ou `boolean`. A seção 9 da especificação ES5 define várias "operações abstratas" (nome técnico para "operação interna") com as regras de conversão de valor. Nós vamos prestar atenção, especificamente, em: `ToString`, `ToNumber` e `ToBoolean`, e menos na extensão  `ToPrimitive`.

### `ToString`

Quando um valor não-`string` é convertido para uma representação `string`, a conversão é manipulada pela operação abstrata `ToString` na seção 9.8 da especificação.

Valores primitivos nativos têm stringficação natural: `null` torna-se `"null"`, `undefined` torna-se `"undefined"` e `true` torna-se `"true"`. `numbers` são geralmente expressos de forma natural como você esperava, mas como discutimos no Capítulo 2, `numbers` muito pequenos ou muito grandes são representados na forma expoente:

```js
// multiplicando `1.07` por `1000`, sete vezes mais
var a = 1.07 * 1000 * 1000 * 1000 * 1000 * 1000 * 1000 * 1000;

// sete vezes três dígitos => 21 digits
a.toString(); // "1.07e21"
```

Para objetos regulares, a menos que você mesmo especifique, o padrão `toString()` (localizado em Object.prototype.toString()`) vai retornar uma *`[[Class]]` interna* (veja o capítulo 3), como por exemplo `"[object Object]"`.

Mas como mostrado anteriormente, se um objeto tem seu próprio método `toString()`, e se você usa esse objeto em um tipo `string`, o `toString()` é o que será chamado automaticamente, e o resultado da `string` dessa chamada é o que vai ser usado no lugar.

**Observação** A forma que um objeto é convertido em uma `string` tecnicamente passa através da operação abstrata `toPrimitive` (seção 9.1 da especificação ES5), mas essas nuances serão abordadas com mais detalhes na seção `ToNumber`, mais tarde nesse capítulo, então vamos pulá-lo aqui.

Arrays têm um padrão `toString()` substitutível que stringfica a (string) concatenação de todos esses valores (cada um stringficando a si mesmo), com `","` entre cada valor:

```js
var a = [1,2,3];

a.toString(); // "1,2,3"
```

Novamente, `toString()` pode tanto ser chamada explicitamente, ou ela vai ser chamada automaticamente se uma não-`string` for usada em um contexto de `string`.

#### Stringficação do JSON

Outra tarefa que parece terrível relacionada a `ToString` é quando você usa a funcionalidade  `JSON.stringify(..)` para serializar um valor para um valor `string` compatível com JSON.

É importante notar que essa stringficação não é exatamente a mesma que a coerção. Mas como ela é relacionada às regras de `ToString` acima, nós faremos uma leve diversificação para abordar os comportamentos de stringficação JSON aqui.

Por mais simples que sejam o valores, a stringficação JSON se comporta basicamente da mesma forma que conversões `toString()`, exceto que a serialização resulta *sempre como uma `string`*:

```js
JSON.stringify( 42 );	// "42"
JSON.stringify( "42" );	// ""42"" (uma string com um valor dentro de aspas)
JSON.stringify( null );	// "null"
JSON.stringify( true );	// "true"
```

Qualuqer valor *seguro para JSON* pode ser stringficada com `JSON.stringify(..)`. Mas o que é *seguro para JSON (JSON-safe)* ? Qualquer valor que pode ser representado em uma representação JSON válida.

Pode ser mais fácil considerar valores que **não** são seguros para JSON. Alguns exemplos: `undefined`s, `function`s, (ES6+) `symbol`s, e `object`s com referências circulares (onde as referências de propriedade em uma estrutura de objeto criam um ciclo interminável entre si). Todos esses são valores ilegais para uma estrutura JSON padrão, principalmente porque ela não têm portabilidade para outras linguagens que consumem valores JSON.

A funcionalidade `JSON.stringify(..)` vai omitir automaticamente valores `undefined`, `function` e `symbol` quando cruzar com eles. Se o valor em questão for encontrado em um `array`, esse valor é substituído por `null` (então aquela posição da informação do array não é alterada). Se for encontrado como propriedade de um objeto, essa propriedade vai simplesmente ser excluída.

Considere:

```js
JSON.stringify( undefined );					// undefined
JSON.stringify( function(){} );					// undefined

JSON.stringify( [1,undefined,function(){},4] );	// "[1,null,null,4]"
JSON.stringify( { a:2, b:function(){} } );		// "{"a":2}"
```

Mas se você tentar fazer um `JSON.stringify(..)` em um `object` com referência(s) circular nele, um erro vai ser lançado.

Stringficação JSON tem um comportamente especial, que se o valor de um `object` tem um método `toJSON()` definido, esse método vai ser chamado primeiro para pegar um valor a ser usado para serialização.

Você tem a intenção de stringficar um objeto JSON que pode conter valores JSON ilegais, ou se você apenas têm, valores no `object` que não são apropriados para a serialização, você deveria definir um método `toJSON()`para que ele retorne à uma versão *segura para JSON* do `object`.

Por exemplo:

```js
var o = { };

var a = {
	b: 42,
	c: o,
	d: function(){}
};

// cria uma referência circular dentro de `a`
o.e = a;

// vai lançar um erro na referência circular
// JSON.stringify( a );

// define um valor de serialização JSON personalizado
a.toJSON = function() {
	// apenas inclui a propriedade `b` para serialização
	return { b: this.b };
};

JSON.stringify( a ); // "{"b":42}"
```

É bem comum o equívoco que `toJSON()` deveria retornar uma representação stringficada de JSON. Isso está provavelmente incorreto, a menos que você queira realmente stringficar a própria `string` (geralmente não!). `toJSON()` deve retornar o valor regular atual (de qualquer tipo) seria apropriado, e o próprio `JSON.stringify(..)` vai manipular a stringficação.

Em outras palavras, `toJSON()` deve ser interpretado como "adequado para stringficação para um valor seguro para JSON", não "para uma string JSON" como muitos desenvolvedores assumem errôneamente.

Considere:

```js
var a = {
	val: [1,2,3],

	// provavelmente correto!
	toJSON: function(){
		return this.val.slice( 1 );
	}
};

var b = {
	val: [1,2,3],

	// provavelmente incorreto!
	toJSON: function(){
		return "[" +
			this.val.slice( 1 ).join() +
		"]";
	}
};

JSON.stringify( a ); // "[2,3]"

JSON.stringify( b ); // ""[2,3]""
```

Na segunda chamada, nós stringficamos o retorno `string` ao invés do próprio `array`, o que provavelmente não é o que queríamos fazer.

Enquanto estamos falando de `JSON.stringify(..)`, vamos discutor algumas funcionalidades pouco conhecidas que continuam a ser bem úteis.

Um segundo argumento opcional pode ser passado para `JSON.stringify(..)` que é chamado *substituto (replacer)*. Esse argumento pode tanto ser um `array` ou uma `function`. É usado para personalizar a serialização recursiva de um `object` fornecendo um mecanismo de filtro no qual propriedades podem ou não serem incluídas, em uma maneira similar de como o `toJSON()` pode preparar um valor para serialização.

Se um *substituto* é um `array`, ele deve ser um `array` de `strings`, no qual cada um vai especificar uma nome de propriedade que é permitida para ser incluída na serialização do `object`. Se uma propriedade que existe não está nessa lista, ela será ignorada.

Se o *substituto* é uma `function`, ele será chamado uma vez pelo próprio `object`, e então uma vez para cada propriedade no `object`, e cada vez que ele passar dois argumentos, *chave* e *valor*. Para ignorar uma *chave* na serialização, retorne `undefined`. Do contrário, retorne o *valor* fornecido.

```js
var a = {
	b: 42,
	c: "42",
	d: [1,2,3]
};

JSON.stringify( a, ["b","c"] ); // "{"b":42,"c":"42"}"

JSON.stringify( a, function(k,v){
	if (k !== "c") return v;
} );
// "{"b":42,"d":[1,2,3]}"
```

**Observação:** No caso do *substituto* da `function`, o argumento chave `k` é `undefined` na primeira chamada (onde o próprio objeto `a` está sendo passado). A declaração `if` **filtra** a propriedade nomeada de `"c"`. Stringficação é recursiva, então o array `[1,2,3]` tem cada um dos seus valores (`1`, `2`, e `3`) passados como `v` para o *substituto*, com índices (`0`, `1`, and `2`) como `k`.

Um terceiro argumento opcional também pode ser passado para `JSON.stringify(..)`, chamado *espaço (space)*, no qual é usado como indentação para deixar a saída mais bonita e amigável. *espaço* pode ser um intermediador positivo para indicar quantos espaços de caracteres devem ser usados em cada nível de identação. Ou, *espaço* pode ser uma `string`, que nesse caso até os primeiros dez caracteres do seu valor serão usados para cada nível de identação.

```js
var a = {
	b: 42,
	c: "42",
	d: [1,2,3]
};

JSON.stringify( a, null, 3 );
// "{
//    "b": 42,
//    "c": "42",
//    "d": [
//       1,
//       2,
//       3
//    ]
// }"

JSON.stringify( a, null, "-----" );
// "{
// -----"b": 42,
// -----"c": "42",
// -----"d": [
// ----------1,
// ----------2,
// ----------3
// -----]
// }"
```

Lembre-se, `JSON.stringify(..)` não é uma forma direta de coerção. Nós o abordamos aqui, porém, por duas razões que seu comportamento está relacionado com coerção `ToString`:

1. Valores `string`, `number`, `boolean`, e `null` todos podem ser stringficados para JSON basicamente o mesmo como a forma que eles convertem valores `string` através das regras da operação abstrata `ToString`.
2. Se você passou o uma valor de `object` para `JSON.stringify(..)`, e esse `object` tem um método `toJSON()` nele, `toJSON()` é chamado automaticamente para (tipo que) "converter" o valor para ser *seguro para JSON* antes da stringficação.

### `ToNumber`

Se qualquer valor não-`number` é usado de uma forma que que exige que seja um `number`, como uma operação matemática, a especificação ES5 define, na seção 9.3, a operação abstrata `ToNumber`.

Por exemplo, `true` torna-se `1` e `false` torna-se `0`. `undefined` torna-se `NaN`, mas (curiosamente) `null` torna-se `0`.

`ToNumber` para valor `string` essencialmente funciona para a maioria das partes como regras/sintaxe para numéricos literais (Veja o Capítulo 3). Se isso falhar, o resultado é `NaN` (ao invés de um erro de sintaxe com `numbers` literais). Um exemplo da diferençã é que `0`-números octais pré fixados não são manipulados como octais (apenas como decimais normais) nessa operação, portanto esses octais são válidos como `numbers` literais (veja o Capítulo 2).

**Observação:** As diferenças entre a gramática de `number` literal  e `ToNumber` em um valor de uma `string` são sutis e altamente matizados, e por isso não serão mais abordados aqui. Consulte a seção 9.3.1 da especificação ES5 para mais informações.

Objetos (e arrays) vão primeiro ser convertidos para seus valores primitivos equivalentes, e o valor resultado (se for primitivo mas ainda não um `number`) é convertido para um `number` de acordo com as regras de `ToNumber` mencionadas.

Para converter para seu valor primitivo equivalente, a operação abstrata`ToPrimitive` (seção 9.1 da especificação ES5) irá consultar o valor (usando a operação interna `DefaultValue` -- seção 8.12.8 da especificação ES5) em questão para ver se ele tem um método `valueOf()`. Se o `valueOf()` estiver disponível e ele retornar um valor primitivo, *aquele* valor é usado para coerção. Do contrário, mas se `toString()` está disponível, ele vai fornecer o valor para a coerção.

Se nenhuma das operações pode fornecer um valor primitivo, um `TypeError` é lançado.

A partir de ES5, você pode criar certos objetos não coercivos -- um sem `valueOf()` e `toString()` -- se ele tiver um valor `null` para seu `[[Prototype]]`, geralmente criado com `Object.create(null)`. Veja o título *this & Object Prototypes* desa série para mais informações de `[[Prototype]]`s.

**Observação:** Nós abordamos como converter para `number`s em detalhes mais tarde nesse capítulo, mas para esse próximo trecho de código, apenas suponha que a função `Number(..)` faz isso.

Considere:

```js
var a = {
	valueOf: function(){
		return "42";
	}
};

var b = {
	toString: function(){
		return "42";
	}
};

var c = [4,2];
c.toString = function(){
	return this.join( "" );	// "42"
};

Number( a );			// 42
Number( b );			// 42
Number( c );			// 42
Number( "" );			// 0
Number( [] );			// 0
Number( [ "abc" ] );	// NaN
```

### `ToBoolean`

A seguir, vamos ter uma pequena conversa sobre como `boolean`s se comportam em JS. Há **muita confusão e equívoco** em torno desse tópico, então preste bastante atenção!

Em primeiro lugar, JS tem as palavras-chave atuais `true` e `false`, e elas se comportam exatamente como você esperaria de valores `boolean`. É um equívoco comum que os valores `1` e `0` sejam idênticos à `true/false`. Enquanto isso pode ser verdadeiro em outras linguagens, em JS os `number`s são `number`s e os `boolean`s são `boolean`s. Você pode converter `1` para `true` (e vice-versa) ou `0` para `false` (e vice versa). Mas eles não são os mesmos.

#### Valores Falsos (Falsy)

Mas esse não é o fim da história. Nós precisamos discutir como outros valores além dos dois `boolean`s se comportam independentemente de você converter *para* seus equivalentes `boolean`.

Todos os valores JavaScript podem ser divididos em duas categorias:

1. Valores que irão se tornar `false` se convertidos para `boolean`
2. Todo o resto (o que vai obviamente se tornar `true`)

Eu não estou apenas sendo engraçado. A especificação JS define uma específica e estreita lista dos valores que poderão tornar-se `false` quando convertidos para um valor `boolean`.

Como sabemos qual é essa lista de valores? Na seção 9.2 da especificação ES5, é definido uma operação abstrata `ToBoolean`, na que diz exatamente o que aconteceria para todos os valores possíveis quando você tenta converte-los "para boolean".

A partir dessa tabela, obtemos o seguinte da chamada lista de valores "falsos":

* `undefined`
* `null`
* `false`
* `+0`, `-0`, e `NaN`
* `""`

É isso. Se um valor não está nessa lista, é um valor "falso" (falsy), e ele não vai ser convertido para `false` se você forçar uma coerção `boolean` nele.

Por conclusão lógica, se um valor *não* está nessa lista, ele deve estar em *outra lista*, na qual nós chamamos de lista de valores "verdadeiros". Mas o JS realmente não define uma lista de valores "verdadeiros" por si só. Ele dá alguns exemplos, assim como dizemos explicitamente que todos os objetos são verdadeiros, mas principalmente a especificação apenas implica que: **qualquer coisa que não esteja explicitamente na lista falsa, é portanto, verdadeira.**

#### Objetos Falsos (Falsy Objects)

Espere um minuto, aquele títulos de seção soa até contraditório. Eu *apenas disse* literalmente que a especificação chama todos os objetos de verdadeiro, certo? Não deveria existir tal coisa como um "objeto falso".

O que isso possivelmente pode significar?

Você deve estar tentado a pensar que isso significa um *object wrapper* (veja o capítulo 3) em torno de um valor falso (como `""`, `0` ou `false`). Mas não caia nessa *armadilha*.

**Observação:** 

Wait a minute, that section title even sounds contradictory. I literally *just said* the spec calls all objects truthy, right? There should be no such thing as a "falsy object."

What could that possibly even mean?

You might be tempted to think it means an object wrapper (see Chapter 3) around a falsy value (such as `""`, `0` or `false`). But don't fall into that *trap*.

Essa é uma piada de especificação sutil que alguns de vocês podem ter sacado.

Considere:

```js
var a = new Boolean( false );
var b = new Number( 0 );
var c = new String( "" );
```

Nós sabemos que todos os três valores são *objects wraper* (veja o capítulo 3) em torno de valores obviamente falsos. Mas esses objetos se comportam como `true` ou como `false`? Essa é fácil de responder:

```js
var d = Boolean( a && b && c );

d; // true
```

Então, todos os três se comportam como `true`, como essa é a única maneira de `d` acabar como `true`.

**Dica** Note que o wrapped `Boolean(..)` está em torno da expressão `a && b && c` -- você deve estar se perguntando porque isso está ali. Nós vamos voltar mais tarde nesse capítulo, então faça um nota mental disso. Para um pequeno exercício, procure por si mesmo o que `d` será se você apenas fizer `d = a && b && c` sem a chamada `Boolean(..)`!

Então, se "objetos falsos" **não são apenas objetos embrulhados em torno de valores falsos**, o que diabos eles são?

A parte complicada é que eles podem aparecer no seu programa JS, mas eles na verdade não são parte do próprio JavaScript.

**O quê?!**

Há certos casos em que navegadores criam seus próprios tipos de comportamentos *exóticos* de valores, nomeando essa ideia de "objetos falsos", no topo da semântica regular do JS.

Um "objeto falso" é um valor que parece e age como um objeto normal (propriedades, etc.), mas quando você converte eles para um `boolean`, ele faz a coerção para um valor `false`.

**Por quê?!**

O caso mais conhecido é `document.all`: um tipo array (objeto) fornecido pelo seu programa JS *pelo DOM* (não pelo próprio motor JS), que expoêm elementos na sua página para seu programa JS. Ele *costuma* se comportar como um objeto normal -- isso seria verdadeiro. Mas não mais.

O próprio `document.all` nunca foi realmente "padrão" e há muito tempo ficou obsoleto/abandonado.

"Eles não podem apenas remover isso então?" Desculpe, boa tentativa. Gostaria que pudessem. Mas há muita base de código JS legado por aí que dependem desse uso.

Então, porque fazer ele agir como falso? Porque coerções de `document.all` para `boolean` (assim como nas declarações `if`) foram quase sempre usadas como um meio de detectar o IE antigo e não padronizado.

O IE há muito tempo vem se aproximando dos padrões e, em muitos casos, vem empurrando a Web para frente tanto ou mais do que qualquer outro navegador. Mas todos aqueles códigos `if` antigos (document.all){ /* it's IE */ }` antigos contiuam por aí, e muito deles, provavelmente, nunca irão embora. Todos esses códigos legados continuam supondo que estão sendo executados em IE antigos, o que leva a más experiências de navegação para usuários IE.

Então, nós não podemos remover `document.all` completamente, mas o IE não quer que códigos `if (document.all) { .. }` funcionem mais, então esses usuários em IE modernos terão novas lógicas de código compatível com os padrões.

"O que devemos fazer?" **"Já sei! Vamos degradar o sistema de tipo do JS e fingir que `document.all` é falso!"

Eca. Isso é uma merda. É um macete louco que a maioria dos desenvolvedores não entendem. Mas a alternativa (fazer nada sobre os problemas sem solução acima) fede *um pouquinho mais*.

Então...é isso que temos: "objetos falsos" loucos e fora do padrão adicionados ao JS pelos navegadores. Ebaa!

#### Valores verdadeiros (truthy)

De volta para a lista verdadeira. O que exatamente são valores verdadeiros? Lembre-se: ** um valor é verdadeiro se ele não está na lista falsa **.

Considere:

```js
var a = "false";
var b = "0";
var c = "''";

var d = Boolean( a && b && c );

d;
```

Qual valor você espera que `d` tenha aqui? Deve ser ou `true` ou `false`.

É `true`. Porquê? Porque apesar dos conteúdos dos valores daquelas `string` parecerem como valores falsos, os próprios valores da `string` são verdadeiros, porque `""` é o único valor de `string` na lista falsa.

E esses?

```js
var a = [];				// array vazio -- verdadeiro ou falso?
var b = {};				// object vazio --  verdadeiro ou falso?
var c = function(){};	// function vazio --  verdadeiro ou falso?

var d = Boolean( a && b && c );

d;
```

Sim, você acertou, `d` continua `true` aqui. Porque? Mesma razão de antes. Apesar do que possa parecer, `[]`, `{}`, e `function(){}` *não* estão na lista falsa, e, portanto, são valores verdadeiros.

Em outras palavras, a lista de verdadeiros é infinitamente longa. É impossível de fazer tal lista. Você apenas pode fazer uma lista falsa e *cosultá-la*.

Pegue cinco minutos, escreva a lista falsa em um post-it para o monitor do seu computador, ou memorize-a se preferir. De qualquer forma, você facilmente poderá construir uma lista falsa virtual sempre que precisar simplesmente perguntando se está na lista falsa ou não.

A importância de verdadeiro e falso no entendimento de como um valor vai se comportar quando você coverte-lo (explicitamente ou implicitamente) para uma valor `boolean`. Agora que você tem essas duas listas em mente, nós podemos mergulhar nos exemplos de coerção.

## Explicit Coercion

*Explicit* coercion refers to type conversions that are obvious and explicit. There's a wide range of type conversion usage that clearly falls under the *explicit* coercion category for most developers.

The goal here is to identify patterns in our code where we can make it clear and obvious that we're converting a value from one type to another, so as to not leave potholes for future developers to trip into. The more explicit we are, the more likely someone later will be able to read our code and understand without undue effort what our intent was.

It would be hard to find any salient disagreements with *explicit* coercion, as it most closely aligns with how the commonly accepted practice of type conversion works in statically typed languages. As such, we'll take for granted (for now) that *explicit* coercion can be agreed upon to not be evil or controversial. We'll revisit this later, though.

### Explicitly: Strings <--> Numbers

We'll start with the simplest and perhaps most common coercion operation: coercing values between `string` and `number` representation.

To coerce between `string`s and `number`s, we use the built-in `String(..)` and `Number(..)` functions (which we referred to as "native constructors" in Chapter 3), but **very importantly**, we do not use the `new` keyword in front of them. As such, we're not creating object wrappers.

Instead, we're actually *explicitly coercing* between the two types:

```js
var a = 42;
var b = String( a );

var c = "3.14";
var d = Number( c );

b; // "42"
d; // 3.14
```

`String(..)` coerces from any other value to a primitive `string` value, using the rules of the `ToString` operation discussed earlier. `Number(..)` coerces from any other value to a primitive `number` value, using the rules of the `ToNumber` operation discussed earlier.

I call this *explicit* coercion because in general, it's pretty obvious to most developers that the end result of these operations is the applicable type conversion.

In fact, this usage actually looks a lot like it does in some other statically typed languages.

For example, in C/C++, you can say either `(int)x` or `int(x)`, and both will convert the value in `x` to an integer. Both forms are valid, but many prefer the latter, which kinda looks like a function call. In JavaScript, when you say `Number(x)`, it looks awfully similar. Does it matter that it's *actually* a function call in JS? Not really.

Besides `String(..)` and `Number(..)`, there are other ways to "explicitly" convert these values between `string` and `number`:

```js
var a = 42;
var b = a.toString();

var c = "3.14";
var d = +c;

b; // "42"
d; // 3.14
```

Calling `a.toString()` is ostensibly explicit (pretty clear that "toString" means "to a string"), but there's some hidden implicitness here. `toString()` cannot be called on a *primitive* value like `42`. So JS automatically "boxes" (see Chapter 3) `42` in an object wrapper, so that `toString()` can be called against the object. In other words, you might call it "explicitly implicit."

`+c` here is showing the *unary operator* form (operator with only one operand) of the `+` operator. Instead of performing mathematic addition (or string concatenation -- see below), the unary `+` explicitly coerces its operand (`c`) to a `number` value.

Is `+c` *explicit* coercion? Depends on your experience and perspective. If you know (which you do, now!) that unary `+` is explicitly intended for `number` coercion, then it's pretty explicit and obvious. However, if you've never seen it before, it can seem awfully confusing, implicit, with hidden side effects, etc.

**Note:** The generally accepted perspective in the open-source JS community is that unary `+` is an accepted form of *explicit* coercion.

Even if you really like the `+c` form, there are definitely places where it can look awfully confusing. Consider:

```js
var c = "3.14";
var d = 5+ +c;

d; // 8.14
```

The unary `-` operator also coerces like `+` does, but it also flips the sign of the number. However, you cannot put two `--` next to each other to unflip the sign, as that's parsed as the decrement operator. Instead, you would need to do: `- -"3.14"` with a space in between, and that would result in coercion to `3.14`.

You can probably dream up all sorts of hideous combinations of binary operators (like `+` for addition) next to the unary form of an operator. Here's another crazy example:

```js
1 + - + + + - + 1;	// 2
```

You should strongly consider avoiding unary `+` (or `-`) coercion when it's immediately adjacent to other operators. While the above works, it would almost universally be considered a bad idea. Even `d = +c` (or `d =+ c` for that matter!) can far too easily be confused for `d += c`, which is entirely different!

**Note:** Another extremely confusing place for unary `+` to be used adjacent to another operator would be the `++` increment operator and `--` decrement operator. For example: `a +++b`, `a + ++b`, and `a + + +b`. See "Expression Side-Effects" in Chapter 5 for more about `++`.

Remember, we're trying to be explicit and **reduce** confusion, not make it much worse!

#### `Date` To `number`

Another common usage of the unary `+` operator is to coerce a `Date` object into a `number`, because the result is the unix timestamp (milliseconds elapsed since 1 January 1970 00:00:00 UTC) representation of the date/time value:

```js
var d = new Date( "Mon, 18 Aug 2014 08:53:06 CDT" );

+d; // 1408369986000
```

The most common usage of this idiom is to get the current *now* moment as a timestamp, such as:

```js
var timestamp = +new Date();
```

**Note:** Some developers are aware of a peculiar syntactic "trick" in JavaScript, which is that the `()` set on a constructor call (a function called with `new`) is *optional* if there are no arguments to pass. So you may run across the `var timestamp = +new Date;` form. However, not all developers agree that omitting the `()` improves readability, as it's an uncommon syntax exception that only applies to the `new fn()` call form and not the regular `fn()` call form.

But coercion is not the only way to get the timestamp out of a `Date` object. A noncoercion approach is perhaps even preferable, as it's even more explicit:

```js
var timestamp = new Date().getTime();
// var timestamp = (new Date()).getTime();
// var timestamp = (new Date).getTime();
```

But an *even more* preferable noncoercion option is to use the ES5 added `Date.now()` static function:

```js
var timestamp = Date.now();
```

And if you want to polyfill `Date.now()` into older browsers, it's pretty simple:

```js
if (!Date.now) {
	Date.now = function() {
		return +new Date();
	};
}
```

I'd recommend skipping the coercion forms related to dates. Use `Date.now()` for current *now* timestamps, and `new Date( .. ).getTime()` for getting a timestamp of a specific *non-now* date/time that you need to specify.

#### O curioso caso do `~`

Um operador coercivo JS que é frequentemente negligenciado e geralmente muito confudido é o operador til `~` (também conhecido como "operador bit a bit NOT"). Muitos dos que até compreendem o que ele faz, vão muitas vezes continuar a evitá-lo. Mas se mantendo no espírito do nossa abordagem nesse livro e série, vamos cavar isso e descobrir se o `~` tem algo de útil para nos dar.

Na seção "inteiros de 32-bit (signed)" do Capítulo 2, nśo abordamos como operadores bit a bit em JS são definidos apenas por operações de 32-bit, o que sifnifica que eles forçam seus operando a entrarem em conformidade com representações de valores 32-bit. As regras para como isso acontece são controladas pela operação abstrata `ToInt32` (Especificação ES5, seção 9.5).

`ToInt32` primeiro faz uma coerção para um `ToNumber`, o que significa que se o valor é `"123"`, ele vai primeiro se tornar `123` antes das regras de `ToInt32` serem aplicadas.

Enquanto não é *tecnicamente* uma coerção em si (desde que o type não mude!), o uso de operadores bit a bit (como `|` ou `~`) com um certo valor `number` especial produz um efeito coercivo que resulta em um valor `number` diferente.

Por exemplo, primeiro vamos considerar o `|` "operador bit a bir OU" usado de outra forma em um idioma não-op `0 | x`, que (como o capítulo 2 mostrou) essencialmente apenas faz a conversão `ToInt32`:

```js
0 | -0;			// 0
0 | NaN;		// 0
0 | Infinity;	// 0
0 | -Infinity;	// 0
```

Esse números especiais não são representações 32-bit (desde que eles venham do padrão 64-bit IEEE 754 -- veja o Capítulo 2), então `ToInt32` apenas especifica `0` como resultado para esses valores.

É discutível se `0 | __` é uma forma *explícita* dessa operação coerciva `ToInt32` ou se ela é mais *implícita*. Pela perspectiva da especificação, é inquestionavelmente *explícita*, mas se você não compreende operações bit a bit nesse nível, isso pode parecer uma mágica mais *implícita*. No entanto, de acordo com outras afirmações nesse capítulo, nós vamos chamá-la de *explícita*.

Então vamos voltar nossa atenção para o `~`. O operador `~` primeiro faz a "coerção" para um valor `number` de 32-bit , e então realiza uma negativa bit a bit (lançando a paridade de cada bit).

**Observação** Isso é bem similar em como `!` não apenas faz a coerção de seus valores para `boolean` mas também lança sua paridade (veja a discussão dos "unários `!`" depois).

Mas...o quê!? Porquê nos importamos com bits sendo lançados? Isso é algo bem específico, algo com muitas nuances. É bem raro que os desenvolvedores JS precisem raciocinar sobre bits individuais.

Outra forma de pensar sobre a definição de `~` vem da ciência da computação/Matemática old-school: `~` realiza dois complementos. Ótimo, obrigado, isso está totalmente claro!

Vamos tentar de novo: `~x` é aproximadamente o mesmo que `-(x+1)`. Isso é estranho, mas um pouco mais fácil de racionalizar. Então:

```js
~42;	// -(42+1) ==> -43
```

Você provavelmente continua imaginando o que diabos é toda essa coisa com o `~`, ou porque isso realmente importa para uma discussão sobre coerção. Vamos chegar ao ponto rapidamente.

Considere `-(x+1)`. Qual é o único valor que você pode realizar essa operação no qual ele irá produzir um resultado `0` (ou, tecnicamente, `-0`)? `-1`. Em outras palavras, `~` usado com uma gama de valores `number` produzirá um valor falso (facilmente coercível para `false`) `0` para o valor de entrada `-1`, e, de outra forma, para qualquer outro valor verdadeiro.

Por que isso é relevante?

`-1` é comumente chamado de um "sentinel value", o que basicamente significa um valor no qual é dado um significado semântico arbitrário dentro do conjunto maior de valores do primeiro tipo (`number`s). A linguagem C usa o sentinel value `-1` para muitas funções que retornam valores `>=0` para "sucesso" e `-1` para "falha".

O JavaScript adotou esse precedente ao definir a operação `string` de `indexOf(..)`, que busca por uma substring e, se encontrada, retorna sua posição de índice inicial, ou `-1` se não encontrada.

É bem comum tentar usar `indexOf(..)` não apenas como uma operação para pegar a posição, mas como uma verificação `boolean` que verifica a presença/ausência de uma sibstring em outra `string`. Veja agora como como desenvolvedores realizam essas verificações:

```js
var a = "Hello World";

if (a.indexOf( "lo" ) >= 0) {	// true
	// encontrado!
}
if (a.indexOf( "lo" ) != -1) {	// true
	// encontrado!
}

if (a.indexOf( "ol" ) < 0) {	// true
	// não encontrado!
}
if (a.indexOf( "ol" ) == -1) {	// true
	// não encontrado!
}
```

Eu acho um pouco grosseiro olhar para `>= 0` ou `== -1`. É basicamente uma "abstração vazada", na medida em que está vazando o comportamento de implementação subjacente -- o uso da sentinela `-1` para "falha" -- no meu código. Eu preferiria esconder tal detahe.

E agora, finalmente, nós vemos porque `~` pode nos ajudar! Usar `~` com `indexOf()` realiza a "coerção" (na verdade apenas transforma) o valor **para ser um `boolean` apropriado para coerção**:

```js
var a = "Hello World";

~a.indexOf( "lo" );			// -4   <-- verdadeiro!

if (~a.indexOf( "lo" )) {	// true
	// encontrado!
}

~a.indexOf( "ol" );			// 0    <-- falso!
!~a.indexOf( "ol" );		// true

if (!~a.indexOf( "ol" )) {	// true
	// não encontrado!
}
```

`~` pega o valor retornado de `indexOf(..)` e o transforma: em caso de "falha" `-1` nós teremos o falso `0`, e todo outro valor é verdadeiro.

**Observação** O pseudo-algoritmo `-(x+1)` para `~` implicaria que `~-1` é `-0`, mas na verdade ele produz `0` porque a operação subjacente é na verdade bit a bit, não matemática.

Tecnicamente, `if (~a.indexOf(..))` ainda está confiando na coerção *implícita* da resultante `0` para `false` ou diferente de zero para `true`. Mas no geral, `~` ainda me parece mais como um mecanismo de coerção *explícita*, desde que você saiba o que pretende fazer nessa linguagem.

Eu acho esse é um código mais limpo do que o desorganizado `>= 0` / `== -1`.

##### Truncando bits

Há mais um lugar que `~` pode aparecer em um código: alguns desenvolvedores usam o til duplo `~~` para truncar a parte decimal de um `number` (aplicar "coerção" para um número "inteiro"). É comum (embora erroneamente) dizer que este é o mesmo resultado que chamar `Math.floor(..)`.

Como `~~` funciona, é que o primeiro `~` aplica a coerção `ToInt32` e faz o lançamento do bit e, em seguida, o segundo` ~ `faz outro lançamento de bit a bit, folheando todos os bits de volta para o estado original. O resultado final é apenas a "coerção" `ToInt32` (também conhecida como truncamento).

**Observação** O lançamento bit a bit duplo de `~~` é muito parecido com o comportamento de paridade da negativa dupla `!!`, explicada mais tarde na seção "Explicitamente: * --> Boolean.

Porém, `~~` precisa de algum cuidado/esclarecimento. Primeiro, ele apenas funciona dependente de valores em 32-bit. Mas, mais importante, ele não funciona da mesma forma em números negativos como o `Math.floor(..)` faz!

```js
Math.floor( -49.6 );	// -50
~~-49.6;				// -49
```

Definindo o `Math.floor(..)`, apesar das diferenças, `~~x` pode truncar para um inteiro (32-bit). Mas o `x | 0` também faz, e aparentemente com (ligeiramente) *menos esforço*.

Então, porque você escolheria `~~x` em vez de `x | 0`? Precedência de operador (veja o Capítulo 5):

```js
~~1E20 / 10;		// 166199296

1E20 | 0 / 10;		// 1661992960
(1E20 | 0) / 10;	// 166199296
```

Assim como todos os outros conselhos aqui, use `~` e `~~`` como mecanismos explícitos para "coerção" e transformação de valores somente se todos que lêem/escrevem o código em questão estão propriamente cientes de como esses operadores funcionam!

### Explicitamente: Parseando strings numéricas

Um resultado semelhante para coagir uma `string` para um` number` pode ser conseguido parseando um `number` de um conteúdo de caracteres de uma `string`

Considere:

```js
var a = "42";
var b = "42px";

Number( a );	// 42
parseInt( a );	// 42

Number( b );	// NaN
parseInt( b );	// 42
```

Parsear um valor numérico de uma string é *tolerante* para caracteres não numéricos -- isso apenas para de parsear da esquerda para a direita quando encontrado -- enquanto a coerção é *não tolerante* e falha, resultando no valor 'NaN`.

Perseamente deve ser visto como um substituto para coerção. Essas duas tarefas, mesmo similares, têm propósitos diferentes. Parsear uma `string` como um `number` quando você não sabe/se importa qual ordem caracteres não-numéricos podem estar no lado da mão-direita. Fazer a coerção de um `string` (para um `number`) quando os únicos valores aceitáveis são numéricos e algo como "42px" deve ser rejeitado como um `number`.

**Dica** `parseInt(..)` tem um irmão gêmeo, `parseFloat(..)`, que (como parece) tira um número de ponto flutuante de uma string.

Não esqueça que `parseInt(..)` opera em valores `string`. Não faz absolutamente nenhum sentido passar um valor `number` para `parseInt(..)`. Nem faria sentido passar nenhum outro tipo de valor, como `true`, `function(){..}` ou `[1,2,3]`.

Se você passar uma não `string`, o valor que você passar vai automaticamente sofrer coerção para uma `string` primeiro (veja "`ToString`" anteriormente), o que vai claramente se um tipo de coerção *implícita* oculta. É realmente uma péssima ideia confiar em tal comportamento no seu programa, então nunca use `parseInt(..)` em um valor que não seja uma `string`.

Antes do ES5, outra pegadinha existia com `parseInt(..)`, a qual era fonte de muitos bugs de programas JS. Se você não passasse um segundo argumento para indicar qual base numérica (conhecida como radix) usar para interpretar o conteúdo numérico da `string`, `parseInt(..)` iria olhar para os caracteres iniciais e adivinhar.

Se os primeiros dois caracteres fossem `"0x"` ou `"0X"`, o palpite (por convenção) era que você queria interpretar a `string` como um `number` de base hexadecimal(base-16). Por outro lado, se o primeiro caractere fosse `"0"`, o palpite (novamente, por convenção) era que você queria interpretar a `string` como um `number` de base octal (base-8).

`string`s hexadecimais (com iniciais `0x` ou `0X`) não são extremamente fáceis de se misturar. Mas a adivinhação do número octal mostrou-se diabolicamente comum. Por exemplo:

```js
var hour = parseInt( selectedHour.value );
var minute = parseInt( selectedMinute.value );

console.log( "The time you selected was: " + hour + ":" + minute);
```

Parece inofensivo, certo? Tente selecionar `08` para hora e `09` para os minutos. Você vai ter `0:0`. Por quê? porque nem `8` nem `9` são caracteres válidos em octais base-8.

A correção pré-ES5 foi simples, mas muito fácil de esquecer: **sempre passar `10` como o segundo argumento**. Isso era totalmente seguro:

```js
var hour = parseInt( selectedHour.value, 10 );
var minute = parseInt( selectedMiniute.value, 10 );
```

A partir da ES5, `parseInt(..)` não adivinhava mais octais. A menos que você diga o contrário, ele supõe caracteres base-10 (ou base-16 para prefixos `"0"`). Isso é muito melhor. Apenas tenha cuidado se seu código tenha que rodar em ambientes pré-ES5, que nesse caso você ainda vai precisar passar `10` para o radix.

#### Parseando não-Strings

Um exemplo um pouco infame do comportamento do `parseInt(..)` é destacado em uma publicação com uma piada sarcástica alguns anos atrás, tirando sarro desse comportamento JS:

```js
parseInt( 1/0, 19 ); // 18
```
A afirmação pretensiosa (mas totalmente inválida) foi: "Se eu passar infinito e parsear um número inteiro disso, eu deveria recuperar o infinito, não 18." Certamente, JS deve estar louco por esse resultado, certo?

Embora este exemplo seja obviamente artificial e irreal, vamos entrar na loucura por um momento e examinar se JS realmente é tão louco.

Primeiramente, o pecado mais óbvio cometido aqui é passar uma não-`string` para `parseInt(..)`. Não, não, não. Faça isso e você estará pedindo por problemas. Mas mesmo se você fizer, o JS, educadamente, faz a coerção o que você passa em uma `string` que pode tentar parsear.

Alguns poderão argumentar que esse é um comportamento irracional, e que `parseInt(..)` deveria operar em um valor não-`string`. Isso deveria lançar um erro? Isso seria muito a cara do Java, francamente. Eu estremeço ao pensar que JS deveria começar a lançar erros em todo o lugar para que o `try..catch` seja necessário em quase todas as linhas.

Ele deveria retornar `NaN`? Talvez. Mas... que tal:

```js
parseInt( new String( "42") );
```

Isso deveria falhar também? É um valor não-`string`. Se você quer que o wrapper de objeto `String` seja desenpacotado para `"42"`, então é realmente tão incomum que o `42` se torne primeiro `"42"` para que `42` possa ser analisado de volta?

Eu argumentaria que essa coerção meio *explícita*, meio *implícita* que pode ocorrer pode ser uma coisa muito útil. Por Exemplo:

```js
var a = {
	num: 21,
	toString: function() { return String( this.num * 2 ); }
};

parseInt( a ); // 42
```

O fato de que `parseInt (...)` forçe a coerção de seu valor para um `string` para realizar um parse é bastante sensato. Se você passar lixo, e você receber lixo de volta, não culpe a lata de lixo -- ela só fez seu trabalho fielmente.

Então, se você passar um valor como `Infinity` (o resultado de `1 / 0` obviamente), que tipo de representação de `string` você faria mais sentido para essa coerção? Apenas duas escolhas racionais vêm à mente: `"Infinity"` and `"∞"`. O JS escolhe `"Infinity"`. E sou grato por ele escolher isso.

Eu acho que é uma coisa boa que **todos os valores** em JS tenham algum tipo de representação de `string` padrão, assim eles não são misteriosas caixas preta que nós não podemos debugar e pensar sobre.

Agora, e sobre caracteres base-19? Obviamente, completamente falso e artificial. Nenhum programa JS real usa base-19. É um absurdo. Mas, de novo, vamos curtir o ridículo. Em base-19, os caracteres numéricos válidos são `0` - `9` e `a` - `i` (case insensitive).

Então, de volta para nosso exemplo `parseInt( 1/0, 19 )`. Isso é essencialmente `parseInt( "Infinity", 19 )`. Como ele irá parsear? O primeiro caractere é o `"I"`, no qual é valor `18` na boba base-19. O segundo caractere `"n"` não está no conjuntos de caracteres válidos, e como tal, o parse simplesmente pára, assim como quando ele cruzar com `"p"` em `"42px"`.

O resultado? `18`. Exatamente como ele, sensatamente, deve ser. Os comportamentos envolvidos para nos trazer até aqui, e não para um próprio erro `Infitnity`, são **muito importantes** para o JS, e não devem ser descartados tão facilmente.

Outros examplos desse comportamento com `parseInt(..)` que podem ser surpreendentes mas são bastante sensatos incluem:

```js
parseInt( 0.000008 );		// 0   ("0" de "0.000008")
parseInt( 0.0000008 );		// 8   ("8" de "8e-7")
parseInt( false, 16 );		// 250 ("fa" de "false")
parseInt( parseInt, 16 );	// 15  ("f" de "function..")

parseInt( "0x10" );			// 16
parseInt( "103", 2 );		// 2
```

Na verdade `parseInt(..)` é bem previsível e consistente em seu comportamento. Se voce usa-lo corretamente, você terá resultados sensatos. Se você usa-lo incorretamente, o resultado maluco que você terá não é culpa do JavaScript.

### Explicitamente: * --> Boolean

Agora, vamos examinar a coerção de qualquer outro valor não `boolean` para um `boolean`.

Assim como com `String(..)` e `Number(..)` acima, `Boolean(..)` (sem o `new`, claro!) é uma forma explícita de forçar a coerção `ToBoolean`:

```js
var a = "0";
var b = [];
var c = {};

var d = "";
var e = 0;
var f = null;
var g;

Boolean( a ); // true
Boolean( b ); // true
Boolean( c ); // true

Boolean( d ); // false
Boolean( e ); // false
Boolean( f ); // false
Boolean( g ); // false
```

Enquanto `Boolean(..)` é claramente explícita, ela não é tão comum ou idiomática.

Assim como o operador unário `+` faz a coerção de um valor para um `number` (veja acima), o operador unário de negação `!` faz a coerção explicitamente de um valor para um `boolean`. O *problema* é que ele também inverte o valor de verdadeiro para falso ou vice versa. Então a forma mais comum em que desenvolvedores JS fazem a coerção explícita para `boolean` é usando o operador de negação duplo `!!`, porque o segundo `!` vai inverter a paridade de volta à original:

```js
var a = "0";
var b = [];
var c = {};

var d = "";
var e = 0;
var f = null;
var g;

!!a;	// true
!!b;	// true
!!c;	// true

!!d;	// false
!!e;	// false
!!f;	// false
!!g;	// false
```

Qualquer uma dessas coerções `ToBoolean` podem acontecer *implicitamente* sem o `Boolean(..)` ou `!!`, se usado em um contexto `boolean` assim como uma declaração `if(..) ..`. Mas o objeto é forçar explicitamente o valor para um `boolean` para deixar claro que a coerção `ToBoolean` é intencional.

Outro caso de uso para coerção explícita `ToBoolean` é se você quer forçar uma coerção de valor `true`/`false` em uma serialização JSON de uma estrutura de dados:

```js
var a = [
	1,
	function(){ /*..*/ },
	2,
	function(){ /*..*/ }
];

JSON.stringify( a ); // "[1,null,2,null]"

JSON.stringify( a, function(key,val){
	if (typeof val == "function") {
		// força a coerção `ToBoolean` da função
		return !!val;
	}
	else {
		return val;
	}
} );
// "[1,true,2,true]"
```

Se você veio para o JavaScript do Java, você deve reconhecer essa linguagem:

```js
var a = 42;

var b = a ? true : false;
```

O operador ternário `? :` vai testar `a` para verdadeiro, e baseado nesse teste atribuirá `true` ou `false` para `b`, em conformidade.

Nessa superfície, essa linguagem é uma forma *explícita* de coerção do tipo `ToBoolean`, uma vez que é óbvio que apenas `true` ou `false` saem da operação.

Porém, há uma coerção *implícita* oculta, aquela expressão `a` deve primeiro sofrer a coerção para `boolean` para executar o teste de verdade. Eu chamaria essa linguagem de "explicitamente implícita". Além disso, eu sugiro que **você evite essa linguagem completamente** no JavaScript. Ela não oferece benefícios reais, e pior, mascara algo que não é.

`Boolean(a)` e `!!a` são de longe melhores as opções para coerção *explícita*.

## Coerção Implícita

Coerção *implícita* se refere à tipos de conversões que são ocultas, com efeitos colaterais não óbvios que implicitamente ocorrem por outras ações. Em outras palavras, *coerções implicitas* são qualquer tipo de conversões que não são óbvias (para você).

Enquanto está claro qual é o objetivo de coerção *explícita* (tornar o código explícito e compreensível), pode ser *muito* óbvio que coerção *implícita* tenha o objetivo oposto: tornar o código mais difícil de se entender.

Confiar de olhos fechados, acredito que é aí que grande parte da raiva de coerções vêm. A maioria das reclamações sobre "coerções JavaScript" têm, na verdade, como alvo (eles percebendo ou não) coerções *implícitas*.

**Observação** Douglas Crockford, autor de *"JavaScript: The Good Parts"*, afirmou em muitas palestras de conferências e artigos que coerção JavaScript deve ser evitada. Mas o que ele pareceu falar é que coerção *implícita* é ruim (na opnião dele). Porém, se você ler seu próprio código, você irá achar muitos exemplos de coerção, ambas *implícita* e *explícita*! Na verdade, a raiva dele parece ser primeiramente destinada para a operação `==`, mas você você verá nesse capítulo, essa é apenas uma parte do mecanismo de coerção.

Então, a **coerção implícita é** maligna? Ela é perigosa? É uma falha no design do JavaScript? Nós devemos evitá-la a todo custo?

**Não tão rápido**. Me dê ouvidos.

Vamos assumir uma perspectiva diferente do que é coerção *implícita*, e pode ser, do que apenas que é "o oposto do bom tipo explícito de coerção". Isso é muito estreiro e perde uma nuance importante.

Vamos definir o objetivo de coerção *implícita* como: reduzir a verbosidade, boilerplate, e/ou detalhes de implentação desnecessários que encobre nosso código com ruído que nos distrai da intenção mais importante.

### Simplificando a Implicidade

Antes até de nós chegarmos ao JavaScript, deixe-me sugerir algum pseudo-código de uma linguagem teórica fortemente tipada para ilustrar:

```js
SomeType x = SomeType( AnotherType( y ) )
```

Nesse exemplo, eu tenho tenho um tipo de valor arbitrário em `y` que eu quero converter para o tipo `SomeType`. O problema é, essa linguagem não pode ir diretamente de qualquer coisa que `y` é pra `SomeType`. Ele precisa de um passo intermediário, onde ele primeiro converte para `AnotherType`, e então de `AnotherType` para `SomeType`.

Agora, e se a linguagem (ou definição que você mesmo pode criar com a linguagem) *fez*, digamos:

```js
SomeType x = SomeType( y )
```

Você não concorda que nós simplificamos o tipo de conversão aqui para reduzir o "ruído" desnecessário do passo de conversão intermediária? Quero dizer, isso é *realmente* tão importante, aqui mesmo nesse ponto do código, para ver e lidar com o fato que `y` vai primeiro para `AnotherType` antes e então vai para `SomeType`?

Alguns argumentariam, pelo menos em algumas circunstâncias, sim. Mas eu acho que um argumento equivalente pode ser feito de várias outras cinscunstâncias que aqui, a simplificação **na verdade ajuda na legibilidade do código** absorvendo ou escondendo tais detalhes, tanto na própria linguagem ou nas suas próprias abstrações.

Sem dúvidas, nos bastidores, em algum lugar, a conversão intermediária continua acontecendo. Mas se esse detalhe é oculto da view, nós apenas podemos raciocinar sobre pegar `y` para o tipo `SomeType` como uma operação genérica e enconder os detalhes bangunçados.

Embora não seja uma analogia perfeita, o que eu vou argumentar em todo o resto desse capítulo é que coerção *implícita* JS pode ser considerada como forncecedora de uma ajuda similar para seu código.

Mas, **e isso é muito importante**, essa não é uma declaração absoluta e ilimitada. Há definitivamente uma abundância de *males* que espreitam a coerção *implícita*, que prejidicará seu código muito mais do que qualquer potencial melhoria de legigibilidade. Claramente, nós teremos que aprender como evitar certos construtos para que não envenenemos nosso código com todas as formas de bugs.

Muitos desenvolvedores acreditam que se um mecanismo pode fazer algo últil **A** mas também pode ser abusado ou mal usado para fazer algo terrível **Z**, então nós devemos descartar completamente esse mecanismo, apenas por segurança.

Meu conselho para você é: não se conforme com isso. Não "mate uma mosca com uma bala de canhão". Não assuma coerção *implícita* é de toda ruim porque tudo que você acha que já viu são "partes ruins". Eu penso que há "partes boas" aqui, e eu quero ajudar e inspirar você para acha-las e absorve-las.

### Implicitamente: Strings <--> Numbers

Mais cedo nesse capítulo, nós exploramos a coerção *implícita* entre valores `string` e `number`. Agora, vamos explorar a mesma tarefa mas com abordagem de coerção *implícita*. Mas antes, nós temos que examinar algumas nuances de operações que vão forçar a coerção *implícita*.

O operador `+` é encarregado de servir propósitos tanto de adição de `number` como concatenação de `string`. Então como o JS sabe qual tipo de operação você quer usar? Considere:

```js
var a = "42";
var b = "0";

var c = 42;
var d = 0;

a + b; // "420"
c + d; // 42
```

Qual a diferença que causa `"420"` vs `42`? É um equívoco comum que a diferença é se um ou ambos os operadores são uma `string`, pois isso significa que `+` assumirá a concatenação `string`. Enquando isso é parcialmente verdade, é mais complicado que isso.

Considere:

```js
var a = [1,2];
var b = [3,4];

a + b; // "1,23,4"
```

Nenhum desses operandos é uma `string`, mas claramente ambos sofrem coerção para `string`s e então a concatenação `string` pula dentro. Então o que realmente stá acontecendo?

(**Atenção** terrível e profunda linguagem de especificação abaixo, então pule os próximos dois parágrafos se isso intimida você!)

-----

De acordo com  a seção 11.6.1 da especificação ES5, o algoritmo `+` (quando um valor `object` é um operando) vai concatenar se um dos operandos já for uma `string`, ou se os passos seguintes produzirem uma representação `string`. Então quando o `+` recebe um `object` (incluindo `array`) para ambos operandos, ele primeiro chama a operação abstrata `ToPrimitive` (seção 9.1) no valor, o que então chama o algoritmo `[[DefaultValue]]` (section 8.12.8) com um contexto `number`.

Se você está prestando bastante atenção, você irá notar que essa operação é agora indêntica a como a operação abstrata `ToNumber` maneja `object` (veja a selção anterior "`ToNumber`"). A operação `valueOf()` no `array` vai falhar em produzir um primitivo simples, então ela cai na representação `toString()`. Os dois `array`s irão então se tornar `"1,2"` and `"3,4"`, respectivamente. Agora, `+` concatena as duas `string` como você espera: `"1,23,4"`.

-----

Vamos olhar além desses detalhes confusos e voltar para uma explicação simplificada: se o operando para `+` é uma `string` (ou torna-se uma com os passos acima!), a operação será concatenação de `string`. Do contrário, ela sempre será adição numérica.

**Observação** uma pegadinha de coerção comumente citada é `[] + {}` vs. `{} + []`, como essas duas expressões resultam, repectivamente, `"[object Object]"` e `0`. Há ainda mais, e cobrimos esses detalhes em "Blocos" no capítulo 5.

O que isso significa para coerção *implícita*?

Voce pode fazer a coerção de um `number` para uma `string` simplismente "adicionando" o `number` e a `string` vazia `""`:

```js
var a = 42;
var b = a + "";

b; // "42"
```

**Dica** Adições numéricas com o operador `+` é comutativa, o que significa que `2 + 3` é o mesmo que `3 + 2`. Concatenação de String com `+` obviamente não é comutativa geralmente, **mas** com o caso específico do `""`, ela é efetivamente comutativa, assim como `a + ""` e `"" + a` irão produzir o mesmo resultado.

É extremamente comum/idiomático fazer a coerção (*implicitamente*) de `number` para `string` com uma operação `+""`. De fato, é interessante que, mesmo alguns dos maiores críticos da coerção *implícita* ainda usam essa abordagem em seu próprio código, em vez de uma das suas alternativas *explícitas*.

**Eu acho que esse é um grande exemplo** de formas úteis na coerção *implícita*, apesar do quão frequentemente o mecanismo recebe críticas.

Comparando essa coerção *implícita* de `a + ""` com nosso exemplo anterior de coerção *explícita* `String(a)`, há uma peculiaridade adicional para ter cuidado. Por causa de como a operação abstrata `ToPrimitive` funciona, `a + ""` invoca `valueOf()` no valor de `a`, no qual o valor de retorno é então finalmente convertido para uma `string` via operação abstrata interna `ToString`. Mas `String(a)` apenas invoca `toString()` diretamente.

Ambas abordagens vão resultar em uma `string` no final, mas se você está usando um `object` em vez de um valor primitivo `number` normal, você pode, não necessariamente, ter o *mesmo* valor de `string`!

Considere:

```js
var a = {
	valueOf: function() { return 42; },
	toString: function() { return 4; }
};

a + "";			// "42"

String( a );	// "4"
```

Geralmente, esse tipo de pegadina não vai te pegar a mesmo que você realmente esteja tentando criar estruturas de dados e operações confusas, mas você deve ter cuidado se você está definindo métodos próprios `valueOf()` e `toString()` para algum `object`, como a forma de fazer a coerção pode afetar o resultado.

E a outra direção? Como podemos fazer a *coerção implícita* de `string` para `number`?

```js
var a = "3.14";
var b = a - 0;

b; // 3.14
```

O operador `-` é definido apenas para subtrações numéricas, então `a - 0` força o valor `a` a sofrer coerção para `number`. Embora muito menos comum, `a * 1` or `a / 1` realizarão o mesmo resultado, já que esses operadores também são definidos apenas para operações numéricas.

E os valores `object` com o operador `-`? Mesma história que para o `+` acima:

```js
var a = [3];
var b = [1];

a - b; // 2
```

Ambos valores `array` precisam se tornar `number`s, mas eles terminam primeiro sofrendo a coerção para `string` (usando a serialização esperada `toString()`), e então sofrem a coerção para `number`s, para a substração `-` seja aplicada.

Então, a coerção *implícita* de valores `string` e `number` são tão maléficas sobre a qual você sempre ouviu histórias de terror? Pessoalmente, eu não acho.

Compare `b = String(a)` (*explícita*) com `b = a + ""` (*implícita*). Eu acho que casos podem ser feitos para que ambas abordagens sejam úteis para seu código. Certamente `b = a + ""` é um pouco mais comum em programs JS, provendo sua própria utilidade independentemente de *sentimentos* sobre os méritos e perigos da coerção *implícita* em geral.

### Implicitamente: Booleans --> Numbers

Eu acho que um caso onde coerção *implícita* pode realmente brilhar é em simplificar certos tipos de lógicas `boolean` complicadas em simples adições numéricas. Claro, essa não é uma técnica com propósito geral, mas uma solução específica para casos específicos.

Considere:

```js
function onlyOne(a,b,c) {
	return !!((a && !b && !c) ||
		(!a && b && !c) || (!a && !b && c));
}

var a = true;
var b = false;

onlyOne( a, b, b );	// true
onlyOne( b, a, b );	// true

onlyOne( a, b, a );	// false
```

Essa utilidade `onlyOne(..)` apenas deve retornar `true` se exatamente um dos argumentos for `true` / verdadeiro. Ela está usando coerção *implícita* nas validações verdadeiras e coerção *explícita* nas outras, incluindo o valor final retornado.

Mas e se precisamos que essa utilidade seja capaz de gerenciar quatro, cinco ou vinte flags da mesma forma? è bem difícil imaginar implementar um código que seja capaz de gerenciar todas essas permutações de cimparações.

Mas aqui está onde fazer a coerção de valores `boolean` para `number`s (`0` ou `1`, obviamente) pode ajudar muito:

```js
function onlyOne() {
	var sum = 0;
	for (var i=0; i < arguments.length; i++) {
		// pula os valores falsos. mesmo que tratar
		// eles como 0's, mas evita os NaN's.
		if (arguments[i]) {
			sum += arguments[i];
		}
	}
	return sum == 1;
}

var a = true;
var b = false;

onlyOne( b, a );				// true
onlyOne( b, a, b, b, b );		// true

onlyOne( b, b );				// false
onlyOne( b, a, b, b, b, a );	// false
```

**Obeservação** Claro, em vez do loop `for` em `onlyOne(..)`, você pode usar a tarefa do ES5 `reduce(..)`, mas eu não queria obscurecer os conceitos.

O que estamos fazendo aqui é relacionado com coerção de `1` para `true`/verdadeiro, e adionando todos numericamente. `sum += arguments[i]` usa coerção *implícita* para fazer isso acontecer. Se um e apenas um valor na lista de `arguments` é `true`, então a soma numérica vai ser `1`, do contrário a soma não será `1` e portanto a condição desejada não será atendida.

Nós podemos claro fazer isso com coerção *implícita* no lugar:

```js
function onlyOne() {
	var sum = 0;
	for (var i=0; i < arguments.length; i++) {
		sum += Number( !!arguments[i] );
	}
	return sum === 1;
}
```

Nós primeiro usamos `!!arguments[i]` para forçar a coerção dos valores para `true` ou `false`. Só assim você poderia passar os valores `boolean`, como `onlyOne( "42", 0 )`, e isso ainda continuará funcionando como esperado (do contrário você vai terminar com uma concatenação `string` e a lógica será incorreta).

Uma vez que temos certeza que é um `boolean`, nós fazemos outra coerção *explícita* com `Number(..)` para ter certeza que os valores são `0` ou `1`.

As formas de coerção *explícita* dessa utilidade são "melhores"? Ela evita o `NaN` como explicado nos comentários do código. Mas, utimamente, isso depende da sua necessidade. Eu pessoalmente acho que a forma anterior, confiando em coerção *implícita* é mais elegante (se você não tiver passando `undefined` ou `NaN`), e a versão *explícita* é desnecessariamente mais verbosa.

Mas assim como tudo o que discutimos aqui, é uma escolha.

**Observação** Independentemente de abordagem *implícita* ou *explícita*, você pode facilmente fazer variações `onlyTwo(..)` ou `onlyFive(..)` simplismente mudando a comparação final de `1`, para `2` ou `5`, respectivamente. Isso é drasticamente mais fácil do que adicionar um monte de expressões `&&` e `||`. Então, geralmente, coerção é muito útil nesse caso.

### Implicitamente: * --> Boolean

Agora, vamos voltar nossa atenção para coerção *implícita* de valores `boolean`, como isso é de longe o mais comum e também de longe o mais potencialmente problemático.

Lembre-se, coerção *implícita* é o que entra quando você usa um valor de tal forma que ele força o valor a ser convertido. Para operações numéricas e de `string`, é bem fácil de ver como as coerções podem acontecer.

Mas, que tipo de expressões de operação requerem/forçam (*implicitamente*) uma coerção `boolean`?

1. A expressão test em uma declaração `if(..)`.
2. A expressão test (segunda cláusula) em um header `for ( .. ; .. ; .. )`.
3. A expressão test em loops `while (..)` e `do..while(..)`.
4. A expressão test (primeira cláusula) em expressões ternárias `? :`.
5. O operando *left-hand* (que serve uma expressão test -- veja abaixo!) para os operadores `||` ("lógico OU") e `&&` ("lógico E").

Qualquer valor usado nesse contexto que já não seja um `boolean` vai sofrer coerção *implícita* para um `boolean` usando as regras da operação abstrata `ToBoolean` abordada anteriormente nesse capítulo.

Vamos ver alguns exemplos:

```js
var a = 42;
var b = "abc";
var c;
var d = null;

if (a) {
	console.log( "yep" );		// yep
}

while (c) {
	console.log( "nope, never runs" );
}

c = d ? a : b;
c;								// "abc"

if ((a && d) || c) {
	console.log( "yep" );		// yep
}
```

Em todos estes contextos, os valores não `boolean`s sofrem coerção *implícita* para seus equivalentes `boolean` para fazer decisões de teste.

### Operadores `||` e `&&`

É bem provável que você já tenha visto os operadores `||` ("lógico OU") e `&&` ("lógico E") na maioria ou em todas outras linguagens que você já usou. Então seria natural presumir que eles trabalham basicamente da mesma forma no JavaScript como nas outras linguagens similares.

Há aqui uma nuance pouco conhecida, mas muito importante.

Na verdade, eu argumentaria que esses operadores nem sequer deveriam ser chamados de "operadores___lógicos", pois esse nome é incompleto ao descrever o que eles fazem. Se eu fosse dar à eles um nome mais preciso (se mais desajeitado), eu os chamaria de "operadores de seletores", ou mais completo, "operadores de seletor de operandos".

Por quê? Porque eles, na verdade, não resultam em um valor *lógico* (também conhecido como `boolean`) no JavaScript, como eles fazem em algumas outras linguagens.

Então qual o *resultado* deles? Eles retornam o valor de um (e apenas um) de seus dois operandos. Em outras palavras, **eles selecionam um dos dois valores de operandos**.

Citação da seção 11.11 da especificação ES5:

> O valor produzido pelo operador && ou || não pe necessariamente do tipo Boolean. O valor prodizido sempre será o valor de uma das duas empressões de operandos.

Vamos ilustrar:

```js
var a = 42;
var b = "abc";
var c = null;

a || b;		// 42
a && b;		// "abc"

c || b;		// "abc"
c && b;		// null
```

**Espera, o quê?** Pense nisso. Em liunguagens como C e PHP, essas expressões resultam em `true` ou `false`, mas em JS (e Python e Ruby, aliás!), o resultado vem dos próprios valores.

Ambos operadores, `||` e `&&` fazem um teste `boolean` no **primeiro operando** (`a` ou `c`). Se o operando já não for um `boolean` (que no caso, não é), uma coerção `ToBoolean` normal acontece, então o teste pode ser feito.

Para o operador `||`, se o teste é `true`, a expressão `||` resulta no valor do *primeiro operando* (`a` ou `c`). Se o teste é `false`, a expressão `||` resulta no valor do *segundo operando* (`b`).

Iversamente, para o operador `&&`, se o teste é `true`, a expressão `&&` resulta no valor do *segundo operando* (`b`) . Se o teste é `false`, a expressão `&&` resulta no valor do *primeiro operando* (`a` ou `c`).

O resultado das expressões `||` ou `&&` é sempre o valor de um dos operandos, **não** o resultado (possivelmente convertido) do teste. Em `c && b`, `c` é `null`, e portanto falso. Mas a própria expressão `&&` resulta em `null` (o valor em `c`), não no `false` convertido usado no teste.

Viu como esses operaodres agem como "seletores de operandos" agora?

Outra forma de pensar sobre esses operadores:

```js
a || b;
// aproximadamente equivalente à:
a ? a : b;

a && b;
// aproximadamente equivalente à:
a ? b : a;
```

**Observação** Eu chamo `a || b` de "aproximadamente equivalente" à `a ? a : b` porque a saída é idêntica, mas há uma diferença de nuance. Em `a ? a : b`, se `a` era uma expressão mais complexa (como por exemplo uma que pode ter efeitos colaterais como chamar uma `function`, etc..), então a expressão `a` vai possivelmente ser avaliada duas vezes (se a primeira avaliação for verdadeira). Por contraste, para `a || b`, a expressão `a` é avalidada apenas uma vez, e esse valor é usado tanto para o teste coercivo como para o valor do resultado (se apropriado). A mesma nuance se aplica para as expressões `a && b` and `a ? b : a`.

Um uso extremamente comum e útil desse comportamento, em que há uma grande chance de você já ter usado isso antes e não entendido completamente é:

```js
function foo(a,b) {
	a = a || "hello";
	b = b || "world";

	console.log( a + " " + b );
}

foo();					// "hello world"
foo( "yeah", "yeah!" );	// "yeah yeah!"
```

O idioma `a = a || "hello"` () testa `a` e se ele não tem valor (ou apenas um valor falso indesejável), provê um valor padrão de backup (`"hello"`).

No entanto, **tenha cuidado!**

The `a = a || "hello"` idiom (sometimes said to be JavaScript's version of the C# "null coalescing operator") acts to test `a` and if it has no value (or only an undesired falsy value), provides a backup default value (`"hello"`).

**Be careful**, though!

```js
foo( "That's it!", "" ); // "That's it! world" <-- Opa!
```

Viu o problema? `""` como segundo argumento é uma valor falso (veja `ToBoolean` anteriormente nesse capítulo), então o teste `b = b || "world"` falha, e o valor padrão `"world"` é substituído, mesmo quando a intenção era, provavelmente, passar explicitamente que `""` seja o valor atribuído para `b`.

Essa linguagem `||` é extremamente comum, e bem útil, mas você tem que usá-la somente em casos onde *todos os valores falsos* devem ser ignorados. Do contrário, você precisará ser mais explícito no seu teste, e provalmente usar um ternário `? :` no lugar.

Essa *atribuição de valor padrão* é tão comum (e útil!) que até mesmo aqueles que veementemente e publicamente condenam a coerção JavaScript, freuqnetemente a utilizam em seu pŕoprio código!

E o `&&`?

Esse é outra linguagem que é bem menso comum, mas que é usanda por minificadores JS frequentemente. O operador `&&` "seleciona" o segundo operando se, e apenas se, o primeiro teste  do operando for verdadeiro, e esse uso é chamado algumas vezes de "operador guarda" (veja também "Circuito curto" no capítulo 5) -- o primeiro teste de expressão "guarda" a segunda expressão:

```js
function foo() {
	console.log( a );
}

var a = 42;

a && foo(); // 42
```

`foo()` é chamada apenas porque o teste de `a` é verdadeiro. Se esse teste falha, essa declaração de expressão `a && foo()` vai apenas parar silenciosamente -- isso é conhecido como "circuito curto" -- e nunca chamar `foo()`.

De novo, não é muito comum que as pessoas criem essas coisas. Normalmente, elas fazem `if (a) { foo(); }` no lugar. Mas os minificadores JS escolhem `a && foo()` porque é muito mais curto. Então, agora, se você alguma vez tiver que decifrar tal código, você saberá o que ele está fazendo e porque.

Ok, então `||` e `&&` têm alguns truques na manga, com tanto que você queira permitir a coerção *implícita* nessa mistura.

**Observação** Ambos, `a = b || "something"` e `a && b()` referen-se ao comportamento de circuitos curtos, que nos abordamos com mais detalhes no capítulo 5.

O fato desses operadores. na verdade, não resultarem em `true` e `false` possivelmente mexerá um pouco com a sua cabeça agora. Você provavelmente está se perguntando como todos suas declarações `if` e seus loops `for` funcionavam, se eles inclíram expressões lógicas compostas como `a && (b || c)`.

Não se preocupe! o céu não está desabando. Seu código está (provavelmente) bem. É que você provavelmente nunca percebeu antes que havia uma coerção *implícita* para `boolean` acontecendo **depois** que a expresão composta era analisada.

Considere:

```js
var a = 42;
var b = null;
var c = "foo";

if (a && (b || c)) {
	console.log( "yep" );
}
```

Esse código continua funcionando da forma que você sempre achou que funcionava, exceto por um detalhe sutil. A expressão `a && (b || c)` *na verdade* resulta em `"foo"`, não `true`. Portanto, a declaração `if` *então* força o valor `"foo"` a sofrer coerção para `boolean`, o que é claro será `true`.

Viu? não há razão para entrar em pânico. Seu código, provavelmente, está à salvo. Mas agora você sabe mais sobre como isso faz o que faz.

E agora você também percebeu que tal código usa coerção *implícita*. Se você ainda está no time "evite coerção (implícita)", você terá que voltar e fazer todos aqueles testes *explicitamente*:

```js
if (!!a && (!!b || !!c)) {
	console.log( "yep" );
}
```

Boa sorte com isso! ... Desculpe, apenas provocando.

### Coerção de symbols

Até esse ponto, não houve quase nenhuma diferença de resultado observável entre coerção *explícita* e *implícita* -- apenas a legibilidade do código está em jogo.

Mas símbolos do ES6 introduzem uma pegadinha no sistema de coerção que nśo precisamos discutir brevemente. Por razões que vão bem além do escopo do que nós vamos discutir nesse livro, coerção *explícita* de um `symbol` para uma `string` é permitida, mas coerção *implícita* do mesmo não é permitida e lançará um erro.

Considere:

```js
var s1 = Symbol( "cool" );
String( s1 );					// "Symbol(cool)"

var s2 = Symbol( "not cool" );
s2 + "";						// TypeError
```

Valores `symbol` não fazem coerção para `number` de nenhuma forma (lança um erro de qualquer jeito), mas estranhamente ambas podem fazer coerção *explícita* e *implícita* para `boolean` (sempre `true`).

Consistências são sempre fáceis de aprender, e exceções nunca são divertidas de lidar, mas nós apenas precisamos ter cuidado com os novos valores `symbol` do ES6 e como nós fazemos coerção nelas.

A boa notícia: provavelmente será extremamente raro você precisar fazer coerção de uma valor `symbol`. A maneira como eles são normalmente usados (veja o capítulo 3), provavelmente não exigirá coerção em uma base normal.

## Igualdade Ampla vs. Igualdade Estrita

Igualdade ampla é o operador `==`, e igualdade estrita é o operador `===`. Ambos operadores são usados para comparar dois valores para "igualdade", mas o "amplo" vs. "estrito" indica uma diferença de comportamento **muito importante** entre os dois, especificamente em como eles decidem a "igualdade".

Um equívico muito comum sobre esses dois operadores é: `==` verifica igualdade de valores e `===` verifica igualdade de ambos, valores e tipos. Enquanto isso parece sensato, é impreciso. Incontáveis livros, bem respeitados, de JavaScript e blogs disseram exatamente isso, mas infelizmente eles estão todos *errados*.

A descrição correta é: "`==` permite coerção na comparação da igualdade e `===` não permite."

### Desempenho da Igualdade

Pare e pense sobre a diferença entre a primeira explicação (imprecisa) e esta segunda (precisa).

Na primeira explicação, parece óbvio que `===` está *fazendo mais trabalho* que `==`, porque ele precisa *também* verificar o tipo. Na segunda explicação, `==` é a que está *fazendo mais trabalho* porque ele precisa seguir através dos passos da coerção se os tipos são diferentes.

Não caia na armadilha, como muitos fazem, de pensar que isso tem alguma coisa a ver com performance, como se `==` fosse ser mais lento que `===` de qualquer maneira relevante. Embora seja mensurável que a coerção tome *um pouco mais* de tempo de processamento, são meros microsegundos (sim, isso é um milionésimo de segundo!).

Se você está comprando dois valores do mesmo tipo, `==` e `===` usam o algoritmo idêntico, e algumas outras diferenças mínimas na implementação do motor, eles devem fazer o mesmo trabalho.

Se você está comparando dois valores de tipos diferentes, a performance não é o fator importante. O que você deveria se perguntar é: ao comparar esses dois valores, eu quero a coerção ou não?

Se você quer a coerção, use `==` igualdade ampla, mas se você não quer coerção, use `===` igualdade estrita.

**Observação** a implicação aqui é que ambos `==` e `===` verifiquem os tipos dos seus operandos. A diferença é em como eles irão responder se os tipos não coincidem.

### Igualdade abstrata

O comportamento do operador `==` é definido como "O algoritmo de comparação de igualdade abstrata" na seção 11.9.3 da especificação ES5. O que está listado lá é um algoritmo abrangente, mas simples, que declara explicitamente todas as combinações possíveis de tipos, e como as coerções (se necessárias) devem acontecer para cada combinação.

**Atenção** Quando (*implicitamente*) a coerção é vista como sendo muito complicada e também defeituosa para ser uma *boa parte útil*, são essas regras de "igualdade abstrata" que estão sendo condenadas. Geralmente, elas são ditas como muito complexas e não intuitivas para desenvolvedores aprenderem e usá-las na prática, e que elas mais causam bugs nos programas JS do que provêm grande legibilidade do código. Eu acredito que essa é uma premissa defeituosa -- que vocês leitores são desenvolvedores competentes que escrevem (e leêm e entendem!) algoritmos (códigos) durante todo o dia. Então o que se segue é uma plano de exposição das "igualdades abstratas" em termos simples. Mas eu imploro para que você também leia a seção 11.9.3 da especificação ES5. Eu acho qe você ficará surpreso do quão sensata ela é.

Basicamente, a primeira cláusula (11.9.3.1) diz, se os dois valores que estão sendo comparados são do mesmo tipo, eles são simplesmente e naturalmente comparados via identidade como você esperava. Por exemplo, `42` só é igual a `42`, e `"abc"` é apenas igual à `"abc"`.

Algumas pequenas exceções à expectativa normal para estar ciente são:

* `NaN` nunca é igual a ela mesma (veja Capítulo 2)
* `+0` e `-0` são iguais entre si (veja Capítulo 2)

A última provisão na cláusula 11.9.3.1 é para comparação de igualdade ampla `==` com `object`s (incluindo `function`s e `array`s). Tais valores são apenas *iguais* se ambos referências para *exatamente o mesmo valor*. Não ocorre coerção aqui.

**Observação** A comparação de igualdade estrita `===` é definida identicamente para 11.9.3.1, incluindo a provisão sobre dois valores de `objects`. É um fato pouco conhecido que **`==` e `===` se comportam de forma idêntica** no caso inde dois `objects`s estão sendo comparados.

O resto do algoritmo em 11.9.3. especifica qur se você usar igualdade ampla `==` para comparar dois valores de tipos diferentes, um ou ambos os valores precisarão sofrer coerção *implícita*. Essa coerção acontece para que ambos valores eventualmente terminem com o mesmo tipo, no qual possam ser comparados pela igualdade usando valores de identidade simples.

**Observação** A operação de não-igualdade ampla `!=` é definida exatamente como você esprava, na medida em que é literalmente a comparação da operação `==` realizada na sua totalidade, e então a negação do resultado. O mesmo vale para a operação de não-igualdade estrita `!==`.

#### Comparing: `string`s to `number`s

To illustrate `==` coercion, let's first build off the `string` and `number` examples earlier in this chapter:

```js
var a = 42;
var b = "42";

a === b;	// false
a == b;		// true
```

As we'd expect, `a === b` fails, because no coercion is allowed, and indeed the `42` and `"42"` values are different.

However, the second comparison `a == b` uses loose equality, which means that if the types happen to be different, the comparison algorithm will perform *implicit* coercion on one or both values.

But exactly what kind of coercion happens here? Does the `a` value of `42` become a `string`, or does the `b` value of `"42"` become a `number`?

In the ES5 spec, clauses 11.9.3.4-5 say:

> 4. If Type(x) is Number and Type(y) is String,
>    return the result of the comparison x == ToNumber(y).
> 5. If Type(x) is String and Type(y) is Number,
>    return the result of the comparison ToNumber(x) == y.

**Warning:** The spec uses `Number` and `String` as the formal names for the types, while this book prefers `number` and `string` for the primitive types. Do not let the capitalization of `Number` in the spec confuse you for the `Number()` native function. For our purposes, the capitalization of the type name is irrelevant -- they have basically the same meaning.

Clearly, the spec says the `"42"` value is coerced to a `number` for the comparison. The *how* of that coercion has already been covered earlier, specifically with the `ToNumber` abstract operation. In this case, it's quite obvious then that the resulting two `42` values are equal.

#### Comparing: anything to `boolean`

One of the biggest gotchas with the *implicit* coercion of `==` loose equality pops up when you try to compare a value directly to `true` or `false`.

Consider:

```js
var a = "42";
var b = true;

a == b;	// false
```

Wait, what happened here!? We know that `"42"` is a truthy value (see earlier in this chapter). So, how come it's not `==` loose equal to `true`?

The reason is both simple and deceptively tricky. It's so easy to misunderstand, many JS developers never pay close enough attention to fully grasp it.

Let's again quote the spec, clauses 11.9.3.6-7:

> 6. If Type(x) is Boolean,
>    return the result of the comparison ToNumber(x) == y.
> 7. If Type(y) is Boolean,
>    return the result of the comparison x == ToNumber(y).

Let's break that down. First:

```js
var x = true;
var y = "42";

x == y; // false
```

The `Type(x)` is indeed `Boolean`, so it performs `ToNumber(x)`, which coerces `true` to `1`. Now, `1 == "42"` is evaluated. The types are still different, so (essentially recursively) we reconsult the algorithm, which just as above will coerce `"42"` to `42`, and `1 == 42` is clearly `false`.

Reverse it, and we still get the same outcome:

```js
var x = "42";
var y = false;

x == y; // false
```

The `Type(y)` is `Boolean` this time, so `ToNumber(y)` yields `0`. `"42" == 0` recursively becomes `42 == 0`, which is of course `false`.

In other words, **the value `"42"` is neither `== true` nor `== false`.** At first, that statement might seem crazy. How can a value be neither truthy nor falsy?

But that's the problem! You're asking the wrong question, entirely. It's not your fault, really. Your brain is tricking you.

`"42"` is indeed truthy, but `"42" == true` **is not performing a boolean test/coercion** at all, no matter what your brain says. `"42"` *is not* being coerced to a `boolean` (`true`), but instead `true` is being coerced to a `1`, and then `"42"` is being coerced to `42`.

Whether we like it or not, `ToBoolean` is not even involved here, so the truthiness or falsiness of `"42"` is irrelevant to the `==` operation!

What *is* relevant is to understand how the `==` comparison algorithm behaves with all the different type combinations. As it regards a `boolean` value on either side of the `==`, a `boolean` always coerces to a `number` *first*.

If that seems strange to you, you're not alone. I personally would recommend to never, ever, under any circumstances, use `== true` or `== false`. Ever.

But remember, I'm only talking about `==` here. `=== true` and `=== false` wouldn't allow the coercion, so they're safe from this hidden `ToNumber` coercion.

Consider:

```js
var a = "42";

// bad (will fail!):
if (a == true) {
	// ..
}

// also bad (will fail!):
if (a === true) {
	// ..
}

// good enough (works implicitly):
if (a) {
	// ..
}

// better (works explicitly):
if (!!a) {
	// ..
}

// also great (works explicitly):
if (Boolean( a )) {
	// ..
}
```

If you avoid ever using `== true` or `== false` (aka loose equality with `boolean`s) in your code, you'll never have to worry about this truthiness/falsiness mental gotcha.

#### Comparing: `null`s to `undefined`s

Another example of *implicit* coercion can be seen with `==` loose equality between `null` and `undefined` values. Yet again quoting the ES5 spec, clauses 11.9.3.2-3:

> 2. If x is null and y is undefined, return true.
> 3. If x is undefined and y is null, return true.

`null` and `undefined`, when compared with `==` loose equality, equate to (aka coerce to) each other (as well as themselves, obviously), and no other values in the entire language.

What this means is that `null` and `undefined` can be treated as indistinguishable for comparison purposes, if you use the `==` loose equality operator to allow their mutual *implicit* coercion.

```js
var a = null;
var b;

a == b;		// true
a == null;	// true
b == null;	// true

a == false;	// false
b == false;	// false
a == "";	// false
b == "";	// false
a == 0;		// false
b == 0;		// false
```

The coercion between `null` and `undefined` is safe and predictable, and no other values can give false positives in such a check. I recommend using this coercion to allow `null` and `undefined` to be indistinguishable and thus treated as the same value.

For example:

```js
var a = doSomething();

if (a == null) {
	// ..
}
```

The `a == null` check will pass only if `doSomething()` returns either `null` or `undefined`, and will fail with any other value, even other falsy values like `0`, `false`, and `""`.

The *explicit* form of the check, which disallows any such coercion, is (I think) unnecessarily much uglier (and perhaps a tiny bit less performant!):

```js
var a = doSomething();

if (a === undefined || a === null) {
	// ..
}
```

In my opinion, the form `a == null` is yet another example where *implicit* coercion improves code readability, but does so in a reliably safe way.

#### Comparing: `object`s to non-`object`s

If an `object`/`function`/`array` is compared to a simple scalar primitive (`string`, `number`, or `boolean`), the ES5 spec says in clauses 11.9.3.8-9:

> 8. If Type(x) is either String or Number and Type(y) is Object,
>    return the result of the comparison x == ToPrimitive(y).
> 9. If Type(x) is Object and Type(y) is either String or Number,
>    return the result of the comparison ToPrimitive(x) == y.

**Note:** You may notice that these clauses only mention `String` and `Number`, but not `Boolean`. That's because, as quoted earlier, clauses 11.9.3.6-7 take care of coercing any `Boolean` operand presented to a `Number` first.

Consider:

```js
var a = 42;
var b = [ 42 ];

a == b;	// true
```

The `[ 42 ]` value has its `ToPrimitive` abstract operation called (see the "Abstract Value Operations" section earlier), which results in the `"42"` value. From there, it's just `42 == "42"`, which as we've already covered becomes `42 == 42`, so `a` and `b` are found to be coercively equal.

**Tip:** All the quirks of the `ToPrimitive` abstract operation that we discussed earlier in this chapter (`toString()`, `valueOf()`) apply here as you'd expect. This can be quite useful if you have a complex data structure that you want to define a custom `valueOf()` method on, to provide a simple value for equality comparison purposes.

In Chapter 3, we covered "unboxing," where an `object` wrapper around a primitive value (like from `new String("abc")`, for instance) is unwrapped, and the underlying primitive value (`"abc"`) is returned. This behavior is related to the `ToPrimitive` coercion in the `==` algorithm:

```js
var a = "abc";
var b = Object( a );	// same as `new String( a )`

a === b;				// false
a == b;					// true
```

`a == b` is `true` because `b` is coerced (aka "unboxed," unwrapped) via `ToPrimitive` to its underlying `"abc"` simple scalar primitive value, which is the same as the value in `a`.

There are some values where this is not the case, though, because of other overriding rules in the `==` algorithm. Consider:

```js
var a = null;
var b = Object( a );	// same as `Object()`
a == b;					// false

var c = undefined;
var d = Object( c );	// same as `Object()`
c == d;					// false

var e = NaN;
var f = Object( e );	// same as `new Number( e )`
e == f;					// false
```

The `null` and `undefined` values cannot be boxed -- they have no object wrapper equivalent -- so `Object(null)` is just like `Object()` in that both just produce a normal object.

`NaN` can be boxed to its `Number` object wrapper equivalent, but when `==` causes an unboxing, the `NaN == NaN` comparison fails because `NaN` is never equal to itself (see Chapter 2).

### Edge Cases

Now that we've thoroughly examined how the *implicit* coercion of `==` loose equality works (in both sensible and surprising ways), let's try to call out the worst, craziest corner cases so we can see what we need to avoid to not get bitten with coercion bugs.

First, let's examine how modifying the built-in native prototypes can produce crazy results:

#### A Number By Any Other Value Would...

```js
Number.prototype.valueOf = function() {
	return 3;
};

new Number( 2 ) == 3;	// true
```

**Warning:** `2 == 3` would not have fallen into this trap, because neither `2` nor `3` would have invoked the built-in `Number.prototype.valueOf()` method because both are already primitive `number` values and can be compared directly. However, `new Number(2)` must go through the `ToPrimitive` coercion, and thus invoke `valueOf()`.

Evil, huh? Of course it is. No one should ever do such a thing. The fact that you *can* do this is sometimes used as a criticism of coercion and `==`. But that's misdirected frustration. JavaScript is not *bad* because you can do such things, a developer is *bad* **if they do such things**. Don't fall into the "my programming language should protect me from myself" fallacy.

Next, let's consider another tricky example, which takes the evil from the previous example to another level:

```js
if (a == 2 && a == 3) {
	// ..
}
```

You might think this would be impossible, because `a` could never be equal to both `2` and `3` *at the same time*. But "at the same time" is inaccurate, since the first expression `a == 2` happens strictly *before* `a == 3`.

So, what if we make `a.valueOf()` have side effects each time it's called, such that the first time it returns `2` and the second time it's called it returns `3`? Pretty easy:

```js
var i = 2;

Number.prototype.valueOf = function() {
	return i++;
};

var a = new Number( 42 );

if (a == 2 && a == 3) {
	console.log( "Yep, this happened." );
}
```

Again, these are evil tricks. Don't do them. But also don't use them as complaints against coercion. Potential abuses of a mechanism are not sufficient evidence to condemn the mechanism. Just avoid these crazy tricks, and stick only with valid and proper usage of coercion.

#### False-y Comparisons

The most common complaint against *implicit* coercion in `==` comparisons comes from how falsy values behave surprisingly when compared to each other.

To illustrate, let's look at a list of the corner-cases around falsy value comparisons, to see which ones are reasonable and which are troublesome:

```js
"0" == null;			// false
"0" == undefined;		// false
"0" == false;			// true -- UH OH!
"0" == NaN;				// false
"0" == 0;				// true
"0" == "";				// false

false == null;			// false
false == undefined;		// false
false == NaN;			// false
false == 0;				// true -- UH OH!
false == "";			// true -- UH OH!
false == [];			// true -- UH OH!
false == {};			// false

"" == null;				// false
"" == undefined;		// false
"" == NaN;				// false
"" == 0;				// true -- UH OH!
"" == [];				// true -- UH OH!
"" == {};				// false

0 == null;				// false
0 == undefined;			// false
0 == NaN;				// false
0 == [];				// true -- UH OH!
0 == {};				// false
```

In this list of 24 comparisons, 17 of them are quite reasonable and predictable. For example, we know that `""` and `NaN` are not at all equatable values, and indeed they don't coerce to be loose equals, whereas `"0"` and `0` are reasonably equatable and *do* coerce as loose equals.

However, seven of the comparisons are marked with "UH OH!" because as false positives, they are much more likely gotchas that could trip you up. `""` and `0` are definitely distinctly different values, and it's rare you'd want to treat them as equatable, so their mutual coercion is troublesome. Note that there aren't any false negatives here.

#### The Crazy Ones

We don't have to stop there, though. We can keep looking for even more troublesome coercions:

```js
[] == ![];		// true
```

Oooo, that seems at a higher level of crazy, right!? Your brain may likely trick you that you're comparing a truthy to a falsy value, so the `true` result is surprising, as we *know* a value can never be truthy and falsy at the same time!

But that's not what's actually happening. Let's break it down. What do we know about the `!` unary operator? It explicitly coerces to a `boolean` using the `ToBoolean` rules (and it also flips the parity). So before `[] == ![]` is even processed, it's actually already translated to `[] == false`. We already saw that form in our above list (`false == []`), so its surprise result is *not new* to us.

How about other corner cases?

```js
2 == [2];		// true
"" == [null];	// true
```

As we said earlier in our `ToNumber` discussion, the right-hand side `[2]` and `[null]` values will go through a `ToPrimitive` coercion so they can be more readily compared to the simple primitives (`2` and `""`, respectively) on the left-hand side. Since the `valueOf()` for `array` values just returns the `array` itself, coercion falls to stringifying the `array`.

`[2]` will become `"2"`, which then is `ToNumber` coerced to `2` for the right-hand side value in the first comparison. `[null]` just straight becomes `""`.

So, `2 == 2` and `"" == ""` are completely understandable.

If your instinct is to still dislike these results, your frustration is not actually with coercion like you probably think it is. It's actually a complaint against the default `array` values' `ToPrimitive` behavior of coercing to a `string` value. More likely, you'd just wish that `[2].toString()` didn't return `"2"`, or that `[null].toString()` didn't return `""`.

But what exactly *should* these `string` coercions result in? I can't really think of any other appropriate `string` coercion of `[2]` than `"2"`, except perhaps `"[2]"` -- but that could be very strange in other contexts!

You could rightly make the case that since `String(null)` becomes `"null"`, then `String([null])` should also become `"null"`. That's a reasonable assertion. So, that's the real culprit.

*Implicit* coercion itself isn't the evil here. Even an *explicit* coercion of `[null]` to a `string` results in `""`. What's at odds is whether it's sensible at all for `array` values to stringify to the equivalent of their contents, and exactly how that happens. So, direct your frustration at the rules for `String( [..] )`, because that's where the craziness stems from. Perhaps there should be no stringification coercion of `array`s at all? But that would have lots of other downsides in other parts of the language.

Another famously cited gotcha:

```js
0 == "\n";		// true
```

As we discussed earlier with empty `""`, `"\n"` (or `" "` or any other whitespace combination) is coerced via `ToNumber`, and the result is `0`. What other `number` value would you expect whitespace to coerce to? Does it bother you that *explicit* `Number(" ")` yields `0`?

Really the only other reasonable `number` value that empty strings or whitespace strings could coerce to is the `NaN`. But would that *really* be better? The comparison `" " == NaN` would of course fail, but it's unclear that we'd have really *fixed* any of the underlying concerns.

The chances that a real-world JS program fails because `0 == "\n"` are awfully rare, and such corner cases are easy to avoid.

Type conversions **always** have corner cases, in any language -- nothing specific to coercion. The issues here are about second-guessing a certain set of corner cases (and perhaps rightly so!?), but that's not a salient argument against the overall coercion mechanism.

Bottom line: almost any crazy coercion between *normal values* that you're likely to run into (aside from intentionally tricky `valueOf()` or `toString()` hacks as earlier) will boil down to the short seven-item list of gotcha coercions we've identified above.

To contrast against these 24 likely suspects for coercion gotchas, consider another list like this:

```js
42 == "43";							// false
"foo" == 42;						// false
"true" == true;						// false

42 == "42";							// true
"foo" == [ "foo" ];					// true
```

In these nonfalsy, noncorner cases (and there are literally an infinite number of comparisons we could put on this list), the coercion results are totally safe, reasonable, and explainable.

#### Sanity Check

OK, we've definitely found some crazy stuff when we've looked deeply into *implicit* coercion. No wonder that most developers claim coercion is evil and should be avoided, right!?

But let's take a step back and do a sanity check.

By way of magnitude comparison, we have *a list* of seven troublesome gotcha coercions, but we have *another list* of (at least 17, but actually infinite) coercions that are totally sane and explainable.

If you're looking for a textbook example of "throwing the baby out with the bathwater," this is it: discarding the entirety of coercion (the infinitely large list of safe and useful behaviors) because of a list of literally just seven gotchas.

The more prudent reaction would be to ask, "how can I use the countless *good parts* of coercion, but avoid the few *bad parts*?"

Let's look again at the *bad* list:

```js
"0" == false;			// true -- UH OH!
false == 0;				// true -- UH OH!
false == "";			// true -- UH OH!
false == [];			// true -- UH OH!
"" == 0;				// true -- UH OH!
"" == [];				// true -- UH OH!
0 == [];				// true -- UH OH!
```

Four of the seven items on this list involve `== false` comparison, which we said earlier you should **always, always** avoid. That's a pretty easy rule to remember.

Now the list is down to three.

```js
"" == 0;				// true -- UH OH!
"" == [];				// true -- UH OH!
0 == [];				// true -- UH OH!
```

Are these reasonable coercions you'd do in a normal JavaScript program? Under what conditions would they really happen?

I don't think it's terribly likely that you'd literally use `== []` in a `boolean` test in your program, at least not if you know what you're doing. You'd probably instead be doing `== ""` or `== 0`, like:

```js
function doSomething(a) {
	if (a == "") {
		// ..
	}
}
```

You'd have an oops if you accidentally called `doSomething(0)` or `doSomething([])`. Another scenario:

```js
function doSomething(a,b) {
	if (a == b) {
		// ..
	}
}
```

Again, this could break if you did something like `doSomething("",0)` or `doSomething([],"")`.

So, while the situations *can* exist where these coercions will bite you, and you'll want to be careful around them, they're probably not super common on the whole of your code base.

#### Safely Using Implicit Coercion

The most important advice I can give you: examine your program and reason about what values can show up on either side of an `==` comparison. To effectively avoid issues with such comparisons, here's some heuristic rules to follow:

1. If either side of the comparison can have `true` or `false` values, don't ever, EVER use `==`.
2. If either side of the comparison can have `[]`, `""`, or `0` values, seriously consider not using `==`.

In these scenarios, it's almost certainly better to use `===` instead of `==`, to avoid unwanted coercion. Follow those two simple rules and pretty much all the coercion gotchas that could reasonably hurt you will effectively be avoided.

**Being more explicit/verbose in these cases will save you from a lot of headaches.**

The question of `==` vs. `===` is really appropriately framed as: should you allow coercion for a comparison or not?

There's lots of cases where such coercion can be helpful, allowing you to more tersely express some comparison logic (like with `null` and `undefined`, for example).

In the overall scheme of things, there's relatively few cases where *implicit* coercion is truly dangerous. But in those places, for safety sake, definitely use `===`.

**Tip:** Another place where coercion is guaranteed *not* to bite you is with the `typeof` operator. `typeof` is always going to return you one of seven strings (see Chapter 1), and none of them are the empty `""` string. As such, there's no case where checking the type of some value is going to run afoul of *implicit* coercion. `typeof x == "function"` is 100% as safe and reliable as `typeof x === "function"`. Literally, the spec says the algorithm will be identical in this situation. So, don't just blindly use `===` everywhere simply because that's what your code tools tell you to do, or (worst of all) because you've been told in some book to **not think about it**. You own the quality of your code.

Is *implicit* coercion evil and dangerous? In a few cases, yes, but overwhelmingly, no.

Be a responsible and mature developer. Learn how to use the power of coercion (both *explicit* and *implicit*) effectively and safely. And teach those around you to do the same.

Here's a handy table made by Alex Dorey (@dorey on GitHub) to visualize a variety of comparisons:

<img src="fig1.png" width="600">

Source: https://github.com/dorey/JavaScript-Equality-Table

## Comparação Relacional Abstrata

Enquanto essa parte da coerção implícita geralmente recebe bem menos atenção, é importante pensar no que acontece com comparações `a < b` (similar à `a == b` que já examinamos em profundidade).

O algoritmo da "Comparação relacional abstrata" na seção 11.8.5 do ES5 essencialmente se divide em duas partes: o que fazer se a comparação involve ambos valores `string` (segunda metade), ou qualquer outra coisa (primeira metade).

**Observação** O algoritmo é apenas definido por  `a < b`. Então, `a > b` é manipulado como `b < a`.

O algoritmo primeiro chama a coerção `ToPrimitive` de ambos os valores, e se o resultado de qualquer uma das chamadas não for uma `string`, então ambos valores são convertidos para valores `number` usando as regras de operação `ToNumber`, e comparado numericamente.

Por exemplo:

```js
var a = [ 42 ];
var b = [ "43" ];

a < b;	// true
b < a;	// false
```

**Observação** As ressalvas semelhantes para `-0` e `NaN` aplicam-se aqui como feitas no algoritmo `==` discutido anteriormente.

Entretanto, se ambos os valores são `string` para a comparação `<`, a comparação lexográfica simples (alfabético natural) é performada nos caracteres:

```js
var a = [ "42" ];
var b = [ "043" ];

a < b;	// false
```

`a` e `b` não são convertidos para `number`, porque ambos terminam como `string` depois da conversão `ToPrimitive` nos dois `array`s. Então, `"42"` é comparado caractere por caractere com `"043"`, começando com os primeiros caracteres `"4"` e `"0"`, respectivamente. Desde que `"0"` seja lexicograficamente *menor que* `"4"`, a comparação retorna `false`.

Exatamente o mesmo comportamento e objetivo acontece para:

```js
var a = [ 4, 2 ];
var b = [ 0, 4, 3 ];

a < b;	// false
```

Aqui, `a` torna-se `"4,2"` e `b` torna-se `"0,4,3"`, e aqueles que se relacionam lexicograficamente de forma idêntica ao trecho anterior.

E sobre:

```js
var a = { b: 42 };
var b = { b: 43 };

a < b;	// ??
```

`a < b` também é `false`, porque `a` torna-se `[object Object]` e `b` torna-se `[object Object]`, e então claramente `a` não é lexograficamente menor que `b`.

Mas estranhamente:

```js
var a = { b: 42 };
var b = { b: 43 };

a < b;	// false
a == b;	// false
a > b;	// false

a <= b;	// true
a >= b;	// true
```

Porque `a == b` não é `true`? Eles são o mesmo valor de `string` (`"[object Object]"`), então parece que eles deveriam ser iguais, certo? Não. Relembre a discussão anterior sobre como `==` funciona com referêcias de `object`.

Mas então como `a <= b` e `a >= b` resultam em `true`, se `a < b` **e** `a == b` **e** `a > b` são todos `false`?

Porque a especificação diz que para `a <= b`, ele vai na verdade avaliar primeiro `b < a`, e então negar esse resultado. Desde que `b < a` seja *também* `false`, o resultado de `a <= b` é `true`.

Isso provavelmente é muito contrário de como você teria explicado o que o `<=` faz até agora, o que provavelmente teria sido o literal: "menor que *ou* igual a." O JS mais precisamente considera `<=` como "não é maior que" (`!(a > b)`, que o JS trata como `!(b < a)`). Além disso, `a >= b` é explicado primeiro considerando-o como `b <= a`, e então aplicando o mesmo reciocícnio.

Infelizmente, não existe "comparação relacional estrita" como é para a igualdade. Em outras palavras, não há como prevenir coerção *implícita* de ocorrer com comparações relacionais como `a < b`, além de garantir que `a` e `b` são explicitamente do mesmo tipo antes de ser feita a comparação.

Use o mesmo raciocínio para nossa discussão anterior sobre teste de sanidade `==` vs. `===`. Se a coerção é útil e razoavelmente segura, como em uma comparação de `42 < "43"`, **use-a**. Por outro lado, se você precisa ter certeza sobre uma comparação relacional, faça a *coerção explícita* dos valores primeiro, antes de usar `<` (ou suas contrapartes).

```js
var a = [ 42 ];
var b = "043";

a < b;						// false -- comparação de string!
Number( a ) < Number( b );	// true -- comparação de number!
```

## Revisão

Nesse capítulo, nós voltamos nossa atenção para como as conversões de tipos acontecem no JavaScript, chamadas **coerção**, na qual pode ser caracterizada como *explícita* ou *implícita*.

Coerção tem uma má reputação, mas ela é na verdade bastante útil em muitos casos. Uma tarefa importante para um desenvolvedor JavaScript responsável é tirar um tempo para aprender todas as entradas e saídas da coerção para decidir quais partes irão ajudar a melhorar seu código, e quais partes ele realmente devem ser evitadas.

Coerção *explícita* é o código que a intenção é converter um valor de um tipo para outro é obvia. O benefício é melhora na legibilidade e manutenabilidade do código reduzindo a confusão.

Coerção *implícita* é a coerção que está "escondida" como um efeito colateral de alguma outra operação, onde não é tão óbvio o tipo de conversão que vai acontecer. Enquanto parece que a coerção *implícita* é o oposto da *explícita*, e portanto, é ruim (e de fato, muitos pensam que sim!), na verdade, coerção *implícita* é também sobre melhorar a legibilidade do código.

Especialmente para *implícita*, coerção deve ser usada com responsabilidade e conscientemente. Saber porque você esté escrevendo o código que está escrevendo, e como ele funciona. Esforce-se para escrever códigos que outros facilmente possam aprender e entender também.