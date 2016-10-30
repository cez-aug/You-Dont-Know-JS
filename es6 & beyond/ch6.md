# You Don't Know JS: ES6 e além
# Chapter 6: Adições no API

De conversão de valores a cálculos matemáticos, ES6 agrega muitas propriedades estáticas e métodos a vários nativos e objetos globais para ajudar com tarefas comuns. Além disso, instâncias de alguns dos nativos têm capacitações através de vários métodos de prototipagem.

**Nota:** A maioria dessas funcionalidades podem ser fielmente polyfilled. Nós não vamos entrar em detalhes aqui, mas cheque o "ES6 Shim" (https://github.com/paulmillr/es6-shim/) para padrões compátiveis de shims/polyfills.

## `Array`

Uma das funcionalidades estendidas mais comum em JS por várias bibliotecas é o Array type. Não deveria ser surpresa que o ES6 adiciona uma quantidade de helpers para Array, tanto estático quanto prototipagem (instancia).

### `Array.of(..)` Função Estática

Há uma pegadinha bem conhecida com o construtor `Array(..)`, que é se só um argumento é passado e esse argumento é um número, ao invés de criar um array de um elemento contendo esse valor, ele constrói um array vazio com uma propriedade `length` igual ao número. Essa ação produz o infeliz e peculiar comportamento do "slot vazio" que os arrays Javascript tanto são criticados.

`Array.of(..)` substitui `Array(..)` como o formato preferido de construtor para arrays, porque `Array.of(..)` não tem aquele caso especial de argumento-numérico-único. Considere:

```js
var a = Array( 3 );
a.length;						// 3
a[0];							// undefined

var b = Array.of( 3 );
b.length;						// 1
b[0];							// 3

var c = Array.of( 1, 2, 3 );
c.length;						// 3
c;								// [1,2,3]
```

Sob essas circunstâncias, você iria querer usar `Array.of(..)` ao invés de apenas criar um array com sintaxe literal, como `c = [1,2,3]`? Há dois casos possíveis.

Se você tem um callback que deve agrupar o(s) argumento(s) passados a ele em um array, `Array.of(..)` encaixa perfeitamente. Isso não é muito comum, mas pode quebrar seu galho.

O outro cenário é se você criar uma subclasse de `Array` (veja "Classes" no Capítulo 3) e quiser criar e inicializar elementos na instância do seu objeto, como por exemplo:

```js
class MyCoolArray extends Array {
	sum() {
		return this.reduce( function reducer(acc,curr){
			return acc + curr;
		}, 0 );
	}
}

var x = new MyCoolArray( 3 );
x.length;						// 3 -- oops!
x.sum();						// 0 -- oops!

var y = [3];					// Array, not MyCoolArray
y.length;						// 1
y.sum();						// `sum` is not a function

var z = MyCoolArray.of( 3 );
z.length;						// 1
z.sum();						// 3
```

Você não pode criar (facilmente) um construtor para `MyCoolArray` que sobrescreve o comportamento do construtor pai `Array`, porque esse construtor é necessário para criar um valor de array que se comporte bem (inicializando o `this`). O método "herdado" estático `of(..)` na subclasse `MyCoolArray` provê uma boa solução.

### `Array.from(..)` Função Estática

Um objeto array-like em JavaScript é um objeto que tem uma propriedade `length`, especificamente com um valor inteiro maior que zero.
An "array-like object" in JavaScript is an object that has a `length` property on it, specifically with an integer value of zero or higher.

Esses valores têm sido notóriamente frustrantes de se trabalhar com JS; É bem comum que seja preciso transformá-los em um verdadeiro array, assim os vários métodos do `Array.prototype` (`map(..)`, `indexOf(..)` etc) podem ser usados. Esse processo geralmente é assim:

```js
// array-like object
var arrLike = {
	length: 3,
	0: "foo",
	1: "bar"
};

var arr = Array.prototype.slice.call( arrLike );
```

Outra tarefa comum onde `slice(..)` geralmente é usado é em duplicar um array real:

```js
var arr2 = arr.slice();
```

Em ambos os casos, o novo método do ES6 `Array.from(..)` pode ter uma abordagem mais compreensivel e elegante -- e também menos verbosa:

```js
var arr = Array.from( arrLike );

var arrCopy = Array.from( arr );
```

`Array.from(..)` verifica se o primeiro argumento é um iterável (veja "Iteráveis" no capítulo 3), e então, usa o iterador para produzir valores para "copiar" para o array retornado. Por conta dos arrays reais terem um iterador para esses valores, o iterador é automaticamente usado.

Mas se você passar um objeto array-like como primeiro argumento ao `Array.from(..)`, ele se comporta basicamente da mesma forma que o `slice()` (sem argumentos!) ou `apply(..)` faz, que é simplesmente percorrer o valor, acessando propriedades nomeadas numericamente desde `0` até o valor de `length`.

Considere:

```js
var arrLike = {
	length: 4,
	2: "foo"
};

Array.from( arrLike );
// [ undefined, undefined, "foo", undefined ]
```

Por conta das posições `0`, `1`, e `3` não existirem no `arrLike`, o resultado foi um valor `undefined` para cada um desses espaços.

Você pode produzir uma saída similar assim:

```js
var emptySlotsArr = [];
emptySlotsArr.length = 4;
emptySlotsArr[2] = "foo";

Array.from( emptySlotsArr );
// [ undefined, undefined, "foo", undefined ]
```

#### Evitando Espaços Vazios

Há uma diferença sutil mas importante no fragmento anterior entre o `emptySlotsArr` e o resultado da chamada do `Array.from(..)`. `Array.from(..)` nunca produz espaços vazios.

Antes do ES6, se você quisesse produzir um array inicializado com um certo tamanho e valores `undefined` em cada espaço (e não espaços vazios!), você tinha que fazer um trabalho extra:

```js
var a = Array( 4 );								// four empty slots!

var b = Array.apply( null, { length: 4 } );		// four `undefined` values
```

Mas `Array.from(..)` agora torna isso mais fácil:

```js
var c = Array.from( { length: 4 } );			// four `undefined` values
```

**Atenção:** Usar um espaço vazio como `a` no fragmento anterior pode funcionar com algumas funções de array, mas outras ignoram espaços vazios (como `map(..)`, etc). Você não deve nunca trabalhar intencionalmente com espaços vazios, já que é quase certo que vai resultar em comportamentos estranhos/imprevisíveis nos seus programas.

#### Mapeamento

O utilitário `Array.from(..)` tem outra carta na manga. O segundo argumento, se passado, é um mapping callback (quase igual ao que o `Array#map(..)` regular espera) que é chamado para mapear/transformar cada valor do original ao alvo retornado. Considere:

```js
var arrLike = {
	length: 4,
	2: "foo"
};

Array.from( arrLike, function mapper(val,idx){
	if (typeof val == "string") {
		return val.toUpperCase();
	}
	else {
		return idx;
	}
} );
// [ 0, 1, "FOO", 3 ]
```

**Nota:** Assim como outros métodos de array que aceitam callbacks, `Array.from(..)` aceita um terceiro argumento opicional que se for definido, vai especificar o `this` para o callback passado no segundo argumento. Caso contrário, `this` vai ser `undefined`.

Veja "Arrays tipados" no Capítulo 5 para um exemplo de uso do `Array.from(..)` traduzindo valores de um array de valores 8-bit para um array de valores 16-bit.

### Criando Arrays e Subtipos

Nas duas últimas seções, nós discutimos `Array.of(..)` e `Array.from(..)`, ambos para criar um novo array de forma similar a um construtor. Mas o que eles fazem nas subclasses? Eles criam instâncias do `Array` base ou de subclasses derivadas?

```js
class MyCoolArray extends Array {
	..
}

MyCoolArray.from( [1, 2] ) instanceof MyCoolArray;	// true

Array.from(
	MyCoolArray.from( [1, 2] )
) instanceof MyCoolArray;							// false
```

Ambos `of(..)` e `from(..)` usam o construtor que eles são acessados para construir o array. Então se você usa a base `Array.of(..)` você vai ter uma instância de `Array`, mas se você usa `MyCollArray.of(..)`, você vai ter uma instância de `MyCoolArray`.

Em "Classes" no Capítulo 3, nós cobrimos a configuração `@@species` que todas as classes nativas (como `Array`) têm definidas, que são usadas por qualquer método prototipado se eles criam uma nova instância. `slice(..)` é um ótimo exemplo:

```js
var x = new MyCoolArray( 1, 2, 3 );

x.slice( 1 ) instanceof MyCoolArray;				// true
```

Geralmente, o comportamento padrão provavelmente vai ser o desejado, mas como discutimos no Capítulo 3, você *pode* sobrescrevê-lo se quiser:

```js
class MyCoolArray extends Array {
	// force `species` to be parent constructor
	static get [Symbol.species]() { return Array; }
}

var x = new MyCoolArray( 1, 2, 3 );

x.slice( 1 ) instanceof MyCoolArray;				// false
x.slice( 1 ) instanceof Array;						// true
```

É importante notar que a configuração de `@@species` só é usada para métodos prototipados, como `slice(..)`. Não é usado por `of(..)` e `from(..)`; ambos somente usam o this (qualquer que seja o construtor usado para fazer referência). Considere:

```js
class MyCoolArray extends Array {
	// force `species` to be parent constructor
	static get [Symbol.species]() { return Array; }
}

var x = new MyCoolArray( 1, 2, 3 );

MyCoolArray.from( x ) instanceof MyCoolArray;		// true
MyCoolArray.of( [2, 3] ) instanceof MyCoolArray;	// true
```

### `copyWithin(..)` Método Prototipado

`Array#copyWithin(..)` é um novo método mutador disponível para todos os arrays (incluindo Arrays Tipados; veja o capítulo 5). `copyWithin(..)` copia uma porção de um array a outro local no mesmo array, sobrescrevendo o que quer que estivesse lá antes.

Os argumentos são *target* (o índice a ser copiado), *start* (o índice inclusivo que vamos partir a cópia), e opcionalmente *end* (o índice exclusivo que vamos parar de copiar). Se algum dos argumentos for negativo, eles passam a ser relativos ao final do array.

Considere:

```js
[1,2,3,4,5].copyWithin( 3, 0 );			// [1,2,3,1,2]

[1,2,3,4,5].copyWithin( 3, 0, 1 );		// [1,2,3,1,5]

[1,2,3,4,5].copyWithin( 0, -2 );		// [4,5,3,4,5]

[1,2,3,4,5].copyWithin( 0, -2, -1 );	// [4,2,3,4,5]
```

O método `copyWithin(..)` não estende o tamanho do array, como o primeiro trecho do exemplo mostra. A cópia simplesmente para quando chega no final do array.

Ao contrário do que você provavelmente pensa, a cópia nem sempre vai da ordem direita-para-esquerda (índice ascendente). É possível que isso possa resultar em repetidamente copiar algum valor já copiado se o intervalo do alvo e do início sobrepor, o que se presume ser um comportamento não esperado.

Então internamente, o algoritmo evita esse caso copiando na ordem inversa para evitar essa pegadinha. Considere:

```js
[1,2,3,4,5].copyWithin( 2, 1 );		// ???
```

Se o algoritmo foi movido estritamente da direita pra esquerda, então o `2` deve ser copiado para sobrescrever o `3`, então *esse* `2` copiado deve ser copiado para sobrescrever o `4`, então *esse* `2` copiado deve ser copiado para sobrescrever o `5`, e você terminaria com `[1,2,2,2,2]`.

Ao invés disso, o algorítmo de cópia inverte a direção e copia o `4` para sobrescrever o `5`, então copia o `3` para sobrescrever o `4`, então copia o `2` para sobrescrever o `3`, e o resultado final é `[1,2,2,3,4]`. Esse é provavelmente o mais "correto" em termos de expectativa, mas pode ser confuso se você está apenas pensando no algorítimo de cópia na moda indênua direita-para-esquerda.

### Método Prototipado `fill(..)`

Preencher um array existente por completo (ou parcialmente) com um valor específico é nativamente suportado em ES6 com o método `Array#fill(..)`:

```js
var a = Array( 4 ).fill( undefined );
a;
// [undefined,undefined,undefined,undefined]
```

`fill(..)` opcionalmente aceita os parâmetros *início* e *fim*, que indicam o subconjunto do array para preencher, tal como:

```js
var a = [ null, null, null, null ].fill( 42, 1, 3 );

a;									// [null,42,42,null]
```

### Método Prototipado `find(..)`

A forma mais comum de procurar por um valor em um array, geralmente tem sio o método `indexOf(..)`, que retorna o índice se o valor for encontrado, ou `-1` se não for encontrado:

```js
var a = [1,2,3,4,5];

(a.indexOf( 3 ) != -1);				// true
(a.indexOf( 7 ) != -1);				// false

(a.indexOf( "2" ) != -1);			// false
```

A comparação `indexOf` requer uma equivalência estrita com `===`, então a procura por `"2"` falha ao encontrar o valor `2` e vice-versa. Não tem como sobrescrever o algoritmo de equivalência para `indexOf(..)`. E também é infeliz/deselegante ter que fazer uma comparação manual com o valor `-1`.

**Dica:** Veja o título *Tipos e Gramática* dessa série para uma técnica interessante (e controversamente confusa) para trabalhar com a feiura do `-1` com o operador `~`.

Desde o ES5, a solução alternativa para ter controle da lógica de equivalência tem sido o método `some(..)`. Ele funciona chamando uma função callback para cada elemento, até que um deles retorne um valor `true`/verdadeiro, e então para. Por conta de você ter que definir uma função callback, você tem total controle de como a equivalência é feita:

```js
var a = [1,2,3,4,5];

a.some( function matcher(v){
	return v == "2";
} );								// true

a.some( function matcher(v){
	return v == 7;
} );								// false
```

Mas o lado negativo dessa abordagem é que você só tem o `true`/`false` indicando se uma equivalência adequada foi encontrada, mas não qual o valor dela.

O `find(..)` do ES6 aborda isso. Funciona basicamente da mesma maneira que o `some(..)`, exceto que uma vez que o callback retorna `true`/valor verdadeiro, o real valor do array é retornado:

```js
var a = [1,2,3,4,5];

a.find( function matcher(v){
	return v == "2";
} );								// 2

a.find( function matcher(v){
	return v == 7;					// undefined
});
```

Usar uma função customizada `matcher(..)` também te deixa testar a equivalência contra valores complexos, como objetos:

```js
var points = [
	{ x: 10, y: 20 },
	{ x: 20, y: 30 },
	{ x: 30, y: 40 },
	{ x: 40, y: 50 },
	{ x: 50, y: 60 }
];

points.find( function matcher(point) {
	return (
		point.x % 3 == 0 &&
		point.y % 4 == 0
	);
} );								// { x: 30, y: 40 }
```

**Nota:** Assim como outros métodos de arrays que aceitam callbacks, `find(..)` aceita um argumento opcional que, se definido, vai especificar o `this` para o callback passado no primeiro argumento. Caso contrário, `this` será indefinido.

### Método Prototipado `findIndex(..)`

Enquanto a seção anterior ilustra como o `some(..)` produz um resultado booleano para uma procura em um array, `find(..)` produz o próprio valor combinadodo array que buscamos, mas ainda há a necessidade de saber o índice da posição desse valor.

`indexOf(..)` faz isso, mas não há controle sobre a sua lógica de equivalência; ele sempre usa igualdade restrita `===`. Então o `findIndex` do ES6 é a resposta:

```js
var points = [
	{ x: 10, y: 20 },
	{ x: 20, y: 30 },
	{ x: 30, y: 40 },
	{ x: 40, y: 50 },
	{ x: 50, y: 60 }
];

points.findIndex( function matcher(point) {
	return (
		point.x % 3 == 0 &&
		point.y % 4 == 0
	);
} );								// 2

points.findIndex( function matcher(point) {
	return (
		point.x % 6 == 0 &&
		point.y % 7 == 0
	);
} );								// -1
```

Não use o `findIndex(..) != -1` (do jeito que é feito com `indexOf(..)`) para pegar um valor booleano da busca, porque `some(..)` já produz o valor `true`/`false` que você quer. E não faça `a[ a.findIndex(..) ]` para pegar o valor combinado, porque é o que o `find(..)` faz. E finalmente, use `indexOf(..)` se você precisa do índice de uma igualdade restrita, ou `findIndex(..)` se você precisa do índice de uma equivalência mais customizada.

**Nota:** Assim como outros métodos de arrays que aceitam callbacks, `find(..)` aceita um argumento opcional que, se definido, vai especificar o `this` para o callback passado no primeiro argumento. Caso contrário, `this` será indefinido.

### Métodos Prototipados `entries()`, `values()`, `keys()`

No Capítulo 3, nós ilustramos como estruturas de dados podem prover uma enumeração modelada item-por-item dos seus valores, via um iterador. Nós então expusemos essa abordagem no Capítulo 5, quando exploramos como as novas collections do ES6 (Map, Set, etc.) provêem vários métodos para produzir diferentes tipos de iterações.

Por conta de isso não ser novo no ES6, `Array` pode não ser pensado tradicionalmente como uma "coleção", mas é se pensarmos que ele fornece os mesmos métodos para iterar: `entries()`, `values()`, e `keys()`. Considere:

```js
var a = [1,2,3];

[...a.values()];					// [1,2,3]
[...a.keys()];						// [0,1,2]
[...a.entries()];					// [ [0,1], [1,2], [2,3] ]

[...a[Symbol.iterator]()];			// [1,2,3]
```

Assim como com `Set`, o iterador padrão `Array` do array retorna o mesmo que `values()`.

Em "Evitando Espaços Vazios" mais acima nesse capítulo, nós ilustramos como `Array.from(..)` trata espaçoes vazios em um array somente colocando `undefined`. Isso é na verdade porque, por baixo dos panos, os iteradores de array se comportam desse jeito:

```js
var a = [];
a.length = 3;
a[1] = 2;

[...a.values()];		// [undefined,2,undefined]
[...a.keys()];			// [0,1,2]
[...a.entries()];		// [ [0,undefined], [1,2], [2,undefined] ]
```

## `Object`

Alguns outros helpers estáticos foram adicionados ao `Object`. Tradicionalmente, funções desse tipo têm sido focadas nos comportamentos/capacidades dos valores do objeto.

Contudo, iniciando com ES6, funções estáticas de `Object` vão servir também para o propósito geral de APIs globais de qualquer tipo que ainda não pertença naturalmente a loutro local (como `Array.from(..)`).

### Função Estática `Object.is(..)`

A função estática `Object.is(..)` faz comparação de valores de uma maneira ainda mais estilosamente estrita do que a comparação com `===`.

`Object.is(..)` invoca o algoritmo subjacente `SameValue` (ES6 spec, seção 7.2.9). O algoritmo `SameValue` é basicamente o mesmo que `===` Algoritmo de Comparação de Igualidade Estrita (ES6 spec, seção 7.2.13), com duas exceções importantes.

Considere:

```js
var x = NaN, y = 0, z = -0;

x === x;							// false
y === z;							// true

Object.is( x, x );					// true
Object.is( y, z );					// false
```

Você deveria continuar usando `===` para comparações de igualidade estritas; `Object.is(..)` não deveria ser pensado como um substituto para o operador. Entretanto, em casos onde você está tentando estritamente identificar um `NaN` ou o valor `-0`, `Object.is(..)` é agora a opção preferida.

**Nota:** ES6 também adiciona um utilitário `Number.isNaN(..)` (discutido mais adiante nesse capítulo) que pode ser levemente mais conveniente; você pode preferir `Number.isNaN(x)` ao invés de `Object.is(x,NaN)`. Você *pode* precisamente testar para `-0` com um desajeitado `x == 0 && 1 / x === -Infinity`, mas nesse caso `Object.is(x,-0)` é muito melhor.

### Função Estática `Object.getOwnPropertySymbols(..)`

A seção "Símbolos" no Capítulo 2 discute o novo tipo de valor primitivo Symbols em ES6.

Símbolos provavelmente vão ser os mais usados como propriedades especiais (meta) em objetos. Então o utilitário `Object.getOwnPropertySymbols(..)` foi introduzido, que recupera apenas as propriedades símbolo diretamente de um objeto:

```js
var o = {
	foo: 42,
	[ Symbol( "bar" ) ]: "hello world",
	baz: true
};

Object.getOwnPropertySymbols( o );	// [ Symbol(bar) ]
```

### Função Estática `Object.setPrototypeOf(..)`

Também no Capítulo 2, nós mencionamos o utilitário `Object.setPrototypeOf(..)`, que (sem surpresa) seta o `[[Prototype]]` de um objeto para fins de *delegação de comportamento* (veja o título *this & Object Prototypes* dessa série). Considere:

```js
var o1 = {
	foo() { console.log( "foo" ); }
};
var o2 = {
	// .. o2's definition ..
};

Object.setPrototypeOf( o2, o1 );

// delegates to `o1.foo()`
o2.foo();							// foo
```

Alternativamente:

```js
var o1 = {
	foo() { console.log( "foo" ); }
};

var o2 = Object.setPrototypeOf( {
	// .. o2's definition ..
}, o1 );

// delegates to `o1.foo()`
o2.foo();							// foo
```

Em ambos os trechos anteriores, a relação entre `o2` e `o1` aparece no fim da definição do `o2`. Mais comumente, a relação entre um `o2` e `o1` é especificada no topo da definição do `o2`, assim como com classes, e também com `__proto__` em objetos literais (veja "Setting `[[Prototype]]` no Capítulo 2").

**Atenção:** Configurar um `[[Prototype]]` logo após a criação do objeto é razoável, como mostrado. Mas mudá-lo muito depois de criá-lo não é uma boa ideia e geralmente acaba mais em confusão do que clareza.

### Função Estática `Object.assign(..)`

Muitas bibliotecas/frameworks JavaScript provêem utilitários para copiar/misturar propriedades de um objeto ao outro (por exemplo, `extend(..)` do jQuery). Tem diferenças de nuances entre esses diferentes utilitários, como se a propriedade com valor `undefined` é ignorada ou não.

ES6 adiciona `Object.assign(..)`, que é uma versão simplificada desses algoritmos. O primeiro argumento é o *alvo*, e quaisquer outros argumentos passados são *origens*, que vão ser processadas em uma ordem listada. Para cada origem, seu enumerável e suas próprias chaves (exemplo, não "herdadas"), incluindo símbolos, são copiadas como se fosse uma atribuição com `=`. `Object.assign(..)` retorna o objeto alvo.

Considere essa configuração de objeto:

```js
var target = {},
	o1 = { a: 1 }, o2 = { b: 2 },
	o3 = { c: 3 }, o4 = { d: 4 };

// setup read-only property
Object.defineProperty( o3, "e", {
	value: 5,
	enumerable: true,
	writable: false,
	configurable: false
} );

// setup non-enumerable property
Object.defineProperty( o3, "f", {
	value: 6,
	enumerable: false
} );

o3[ Symbol( "g" ) ] = 7;

// setup non-enumerable symbol
Object.defineProperty( o3, Symbol( "h" ), {
	value: 8,
	enumerable: false
} );

Object.setPrototypeOf( o3, o4 );
```

Somente as propriedades `a`, `b`, `c`, `e`, e o  `Symbol("g")` vão ser copiados ao `target`:

```js
Object.assign( target, o1, o2, o3 );

target.a;							// 1
target.b;							// 2
target.c;							// 3

Object.getOwnPropertyDescriptor( target, "e" );
// { value: 5, writable: true, enumerable: true,
//   configurable: true }

Object.getOwnPropertySymbols( target );
// [Symbol("g")]
```

As propriedades `d`, `f`, e `Symbol("h")` são omitidas da cópia; Propriedades não-enumeráveis e não-próprias são todas excluidas da atribuição. Também, `e` é copiada como uma atribuição normal de propriedade, não duplicada como uma propriedade read-only.

Em uma seção anterior, nós mostramos o uso de `setPrototypeOf(..)` para configurar uma relação `[[Prototype]]` entre os objetos `o2` e `o1`. Tem outra forma que se aproveita do `Object.assign(..)`:

```js
var o1 = {
	foo() { console.log( "foo" ); }
};

var o2 = Object.assign(
	Object.create( o1 ),
	{
		// .. o2's definition ..
	}
);

// delegates to `o1.foo()`
o2.foo();							// foo
```

**Nota:** `Object.create(..)` é o utilitário padrão do ES5 que cria um objeto vazio que é `[[Prototype]]`-linked. Veja o título *this & Object Prototypes* dessa série para mais informação.

## `Math`

ES6 adicional diversos utilitários matemáticos que preenchem buracos ou ajudam com operações comuns. Todos podem ser manualmente calculados, mas a maioria está agora definido nativamente, então em alguns casos a engine do JS pode otimizar a performance dos cálculos e ser mais performático com mais precisão de números decimais do que a solução manual.

É provavel que asm.js/código JS transpilado (veja o título *Async & Performance* dessa série) é o consumidor mais provável de muitos desses utilitários, ao invés de desenvolvedores diretos.

Trigonometria:

* `cosh(..)` - Coseno hiperbólico
* `acosh(..)` - Arco-cosseno hiperbólico
* `sinh(..)` - Seno hiperbólico
* `asinh(..)` - Arco-seno hiperbólico
* `tanh(..)` - Tangente hiperbólica
* `atanh(..)` - Arco-tangente hiperbólica
* `hypot(..)` - A raíz quadrada da soma dos quadrados (ou seja, o generalizado Teorema de Pitágoras)

Aritmética:

* `cbrt(..)` - Raíz cúbica
* `clz32(..)` - Conta os zeros à esquerda em uma representação binária de 32-bit
* `expm1(..)` - O mesmo que `exp(x) - 1`
* `log2(..)` - Logarítimo binário (log de base 2)
* `log10(..)` - Log de base 10
* `log1p(..)` - O mesmo que `log(x + 1)`
* `imul(..)` - Multiplicação de dois números inteiros de 32-bit

Meta:

* `sign(..)` - Retorna o sinal de um número
* `trunc(..)` - Retorna apenas a parte inteira de um número
* `fround(..)` - Arrendonda para o valor float mais próximo de 32-bit (precisão única)

## `Number`

Importantly, for your program to properly work, it must accurately handle numbers. ES6 adds some additional properties and functions to assist with common numeric operations.

Two additions to `Number` are just references to the preexisting globals: `Number.parseInt(..)` and `Number.parseFloat(..)`.

### Static Properties

ES6 adds some helpful numeric constants as static properties:

* `Number.EPSILON` - The minimum value between any two numbers: `2^-52` (see Chapter 2 of the *Types & Grammar* title of this series regarding using this value as a tolerance for imprecision in floating-point arithmetic)
* `Number.MAX_SAFE_INTEGER` - The highest integer that can "safely" be represented unambiguously in a JS number value: `2^53 - 1`
* `Number.MIN_SAFE_INTEGER` - The lowest integer that can "safely" be represented unambiguously in a JS number value: `-(2^53 - 1)` or `(-2)^53 + 1`.

**Note:** See Chapter 2 of the *Types & Grammar* title of this series for more information about "safe" integers.

### `Number.isNaN(..)` Static Function

The standard global `isNaN(..)` utility has been broken since its inception, in that it returns `true` for things that are not numbers, not just for the actual `NaN` value, because it coerces the argument to a number type (which can falsely result in a NaN). ES6 adds a fixed utility `Number.isNaN(..)` that works as it should:

```js
var a = NaN, b = "NaN", c = 42;

isNaN( a );							// true
isNaN( b );							// true -- oops!
isNaN( c );							// false

Number.isNaN( a );					// true
Number.isNaN( b );					// false -- fixed!
Number.isNaN( c );					// false
```

### `Number.isFinite(..)` Static Function

There's a temptation to look at a function name like `isFinite(..)` and assume it's simply "not infinite". That's not quite correct, though. There's more nuance to this new ES6 utility. Consider:

```js
var a = NaN, b = Infinity, c = 42;

Number.isFinite( a );				// false
Number.isFinite( b );				// false

Number.isFinite( c );				// true
```

The standard global `isFinite(..)` coerces its argument, but `Number.isFinite(..)` omits the coercive behavior:

```js
var a = "42";

isFinite( a );						// true
Number.isFinite( a );				// false
```

You may still prefer the coercion, in which case using the global `isFinite(..)` is a valid choice. Alternatively, and perhaps more sensibly, you can use `Number.isFinite(+x)`, which explicitly coerces `x` to a number before passing it in (see Chapter 4 of the *Types & Grammar* title of this series).

### Integer-Related Static Functions

JavaScript number values are always floating point (IEEE-754). So the notion of determining if a number is an "integer" is not about checking its type, because JS makes no such distinction.

Instead, you need to check if there's any non-zero decimal portion of the value. The easiest way to do that has commonly been:

```js
x === Math.floor( x );
```

ES6 adds a `Number.isInteger(..)` helper utility that potentially can determine this quality slightly more efficiently:

```js
Number.isInteger( 4 );				// true
Number.isInteger( 4.2 );			// false
```

**Note:** In JavaScript, there's no difference between `4`, `4.`, `4.0`, or `4.0000`. All of these would be considered an "integer", and would thus yield `true` from `Number.isInteger(..)`.

In addition, `Number.isInteger(..)` filters out some clearly not-integer values that `x === Math.floor(x)` could potentially mix up:

```js
Number.isInteger( NaN );			// false
Number.isInteger( Infinity );		// false
```

Working with "integers" is sometimes an important bit of information, as it can simplify certain kinds of algorithms. JS code by itself will not run faster just from filtering for only integers, but there are optimization techniques the engine can take (e.g., asm.js) when only integers are being used.

Because of `Number.isInteger(..)`'s handling of `NaN` and `Infinity` values, defining a `isFloat(..)` utility would not be just as simple as `!Number.isInteger(..)`. You'd need to do something like:

```js
function isFloat(x) {
	return Number.isFinite( x ) && !Number.isInteger( x );
}

isFloat( 4.2 );						// true
isFloat( 4 );						// false

isFloat( NaN );						// false
isFloat( Infinity );				// false
```

**Note:** It may seem strange, but Infinity should neither be considered an integer nor a float.

ES6 also defines a `Number.isSafeInteger(..)` utility, which checks to make sure the value is both an integer and within the range of `Number.MIN_SAFE_INTEGER`-`Number.MAX_SAFE_INTEGER` (inclusive).

```js
var x = Math.pow( 2, 53 ),
	y = Math.pow( -2, 53 );

Number.isSafeInteger( x - 1 );		// true
Number.isSafeInteger( y + 1 );		// true

Number.isSafeInteger( x );			// false
Number.isSafeInteger( y );			// false
```

## `String`

Strings already have quite a few helpers prior to ES6, but even more have been added to the mix.

### Unicode Functions

"Unicode-Aware String Operations" in Chapter 2 discusses `String.fromCodePoint(..)`, `String#codePointAt(..)`, and `String#normalize(..)` in detail. They have been added to improve Unicode support in JS string values.

```js
String.fromCodePoint( 0x1d49e );			// "𝒞"

"ab𝒞d".codePointAt( 2 ).toString( 16 );		// "1d49e"
```

The `normalize(..)` string prototype method is used to perform Unicode normalizations that either combine characters with adjacent "combining marks" or decompose combined characters.

Generally, the normalization won't create a visible effect on the contents of the string, but will change the contents of the string, which can affect how things like the `length` property are reported, as well as how character access by position behave:

```js
var s1 = "e\u0301";
s1.length;							// 2

var s2 = s1.normalize();
s2.length;							// 1
s2 === "\xE9";						// true
```

`normalize(..)` takes an optional argument that specifies the normalization form to use. This argument must be one of the following four values: `"NFC"` (default), `"NFD"`, `"NFKC"`, or `"NFKD"`.

**Note:** Normalization forms and their effects on strings is well beyond the scope of what we'll discuss here. See "Unicode Normalization Forms" (http://www.unicode.org/reports/tr15/) for more information.

### `String.raw(..)` Static Function

The `String.raw(..)` utility is provided as a built-in tag function to use with template string literals (see Chapter 2) for obtaining the raw string value without any processing of escape sequences.

This function will almost never be called manually, but will be used with tagged template literals:

```js
var str = "bc";

String.raw`\ta${str}d\xE9`;
// "\tabcd\xE9", not "	abcdé"
```

In the resultant string, `\` and `t` are separate raw characters, not the one escape sequence character `\t`. The same is true with the Unicode escape sequence.

### `repeat(..)` Prototype Function

In languages like Python and Ruby, you can repeat a string as:

```js
"foo" * 3;							// "foofoofoo"
```

That doesn't work in JS, because `*` multiplication is only defined for numbers, and thus `"foo"` coerces to the `NaN` number.

However, ES6 defines a string prototype method `repeat(..)` to accomplish the task:

```js
"foo".repeat( 3 );					// "foofoofoo"
```

### String Inspection Functions

In addition to `String#indexOf(..)` and `String#lastIndexOf(..)` from prior to ES6, three new methods for searching/inspection have been added: `startsWith(..)`, `endsWidth(..)`, and `includes(..)`.

```js
var palindrome = "step on no pets";

palindrome.startsWith( "step on" );	// true
palindrome.startsWith( "on", 5 );	// true

palindrome.endsWith( "no pets" );	// true
palindrome.endsWith( "no", 10 );	// true

palindrome.includes( "on" );		// true
palindrome.includes( "on", 6 );		// false
```

For all the string search/inspection methods, if you look for an empty string `""`, it will either be found at the beginning or the end of the string.

**Warning:** These methods will not by default accept a regular expression for the search string. See "Regular Expression Symbols" in Chapter 7 for information about disabling the `isRegExp` check that is performed on this first argument.

## Review

ES6 adds many extra API helpers on the various built-in native objects:

* `Array` adds `of(..)` and `from(..)` static functions, as well as prototype functions like `copyWithin(..)` and `fill(..)`.
* `Object` adds static functions like `is(..)` and `assign(..)`.
* `Math` adds static functions like `acosh(..)` and `clz32(..)`.
* `Number` adds static properties like `Number.EPSILON`, as well as static functions like `Number.isFinite(..)`.
* `String` adds static functions like `String.fromCodePoint(..)` and `String.raw(..)`, as well as prototype functions like `repeat(..)` and `includes(..)`.

Most of these additions can be polyfilled (see ES6 Shim), and were inspired by utilities in common JS libraries/frameworks.
