---
id: flow
title: Flow all-in
sidebar_label: Flow all-in
---

- **https://flow.org/en/docs/faq/**
- https://github.com/facebook/flow/wiki
- https://github.com/wgao19/flow-notes
- https://github.com/facebook/flow/blob/master/Changelog.md
- https://github.com/niieani/typescript-vs-flowtype
- https://github.com/vkurchatkin/typescript-vs-flow
- https://github.com/lttb/flown
- https://github.com/dustinspecker/awesome-flow
- https://gajus.github.io/flow-runtime/
- https://medium.com/flow-type/what-the-flow-team-has-been-up-to-54239c62004f#a548
- https://github.com/facebook/flow/issues/7365 (Facebook's own Flow adoption?)
- https://github.com/lessmess-dev/gen-flow-files
- Paper: [Fast and Precise Type Checking for JavaScript](https://arxiv.org/pdf/1708.08021.pdf)
- type checking for `this` annotations in functions: https://github.com/facebook/flow/commit/fb6a836ef7d8d3ae842ac9df67e4c83698cbffb4

:::tip
Flow infers the widest type that makes your code work. If you don't want inference to widen your type, the solution is always to annotate.
:::

## Oncalls in Facebook (Flow related)

> so the way it works is that the Flow team has a rotating oncall. it's relatively calm as oncalls go (we aren't getting woken up in the middle of the night), but whoever is oncall is responsible for doing support (we have an internal group where people can ask questions), and also responsible for taking the lead if something goes wrong with Flow or the various related integrations we have. near the beginning of the year we also made it so that the oncall is responsible for addressing libdef and documentation PRs, since there is usually no clear owner for those, and pretty much anyone should be able to review them

(source Discord)

## Exit codes

Flow exited with some numeric exit code and you are not sure what does it mean? You can find these exit codes here: https://github.com/facebook/flow/blob/eaffc8cdf09fb268c6f9ab9914685cbbd562e534/src/common/exit_status/flowExitStatus.ml

## Sound vs. complete

> A sound type system (or analysis) is one that rejects all erroneous programs, and a complete system accepts all correct programs.

> A sound type system is one that is not lenient, in that it rejects all invalid programs plus some number of valid programs. A complete type system accepts all valid programs, and some invalid ones to boot. Which to pick?

> A sound type system is one that ensures your program does not get into invalid states. For example, if an expression’s static type is string, at runtime, you are guaranteed only to get a string when you evaluate it.

- https://eschew.wordpress.com/2009/08/31/sound-and-complete/
- https://stackoverflow.com/a/21437375/3135248
- https://blog.logrocket.com/is-typescript-worth-it
- https://dart.dev/guides/language/sound-dart#what-is-soundness

Please note: not everything can be expressed/modeled in your type system so you have to take into account also dynamic errors (division by zero or integer overflow) when writing your program.

> Programmers dislike having the computer reject a program that would have run fine, simply because the computer couldn’t make sure it would run fine without actually running it. In short, restrictive type systems drive programmers to more flexible environments.

## New generics

TKTK

```ini
[options]
generate_tests=false
```

- https://github.com/facebook/flow/search?p=1&q=%22%5Bnew-generics%5D%22&type=commits
- https://github.com/facebook/flow/search?p=1&q=%22generate_tests%22&type=code

## Type-safe filter function

```js
declare function filter<K, V, P: $Pred<1>, T: Array<V> | $ReadOnlyArray<V>>(
  fn: P,
  xs: T,
): Array<$Refine<V, P, 1>>;

const input: $ReadOnlyArray<number | string> = [1, 'a', 2, 3, 'b'];

function isNumber(x: mixed): boolean %checks {
  return typeof x === 'number';
}

// const result: $ReadOnlyArray<number> = input.filter(isNumber);
//                                        ^ Cannot assign `input.filter(...)` to `result` because string [1] is incompatible with number [2] in array element.
const result: $ReadOnlyArray<number> = input.filter(isNumber);

const result_fixed: $ReadOnlyArray<number> = filter(isNumber, input);
```

[flow.org/try](https://flow.org/try/#0CYUwxgNghgTiAEAzArgOzAFwJYHtVKwgxBgB4BpAGngDVqAFALngBJ65hSBGAPmoBVmAQRgwoAT1I0e8AD6sASiCjAA8qgjiRYydJ4AKAFDwkqZvUrH4ADwDOzfpYCUw0RNIsliLKhBSG1Lw8ANyGhmB4thjwPgAOyBjMnspqGlpukqjIALYARiRy8FEwPgDmMgC88ADaXNQA5FD11ABM1ADMDbn1ALqhhijo2HgxtgByOfkw+tbM2VjWIMAu8Lk4OBDK+ACkYAAW4ADWtvAA3lZwGMgw+BjisSA4iDbwFW-w9Vl5JPWhAL5hCKoKLwOC2ZBEJJKFTqTTadxfKaVGKoeIYAB03iIJH0WHGkxITlCQJBYIhGAA+t5FsAoSlYekdKRESRkVjiNM8RNvjBqHEEkSgA)

- https://github.com/facebook/flow/issues/1414

## Force type casting

You may find yourself in a situation where you want to force cast some type despite it's originally defined differently. Example:

```js
type StringOrNumber = string | number;
const foo: StringOrNumber = 'hello';

const bar1: string = foo; // cannot assign `foo` to `bar` because number is incompatible with string
const bar2: string = (foo: string); // cannot cast `foo` to string because number is incompatible with string
```

Both errors are correct and valid. The solution is to cast the type via `any` like this:

```js
const bar: string = (foo: any);
```

> So you cast `foo` to an `any` because `any` accepts any type of value as an input. Then because the `any` type also allows you to read all possible types from it, you can assign the `any` value to `bar` because an `any` is also a string.

This obviously means that you are going against the type system and you might be doing something wrong. Did you want to use conditional refinement instead?

```js
type StringOrNumber = string | number;
const foo: StringOrNumber = 'hello';

if (typeof foo === 'string') {
  const bar: string = foo; // no errors!
}
```

> It's important to note that both suppression comments and cast-through any are **unsafe**. They override Flow completely, and will happily perform completely nonsensical "casts". To avoid this, use refinements that Flow understands whenever you can, and avoid use of the other, unsafe "casting" techniques.

Source: https://stackoverflow.com/questions/41328728/how-can-flow-be-forced-to-cast-a-value-to-another-type/45068255

## `$Compose`, `$ComposeReverse`

Compose pattern is very common among JS community. Here are some examples: [Redux](http://redux.js.org/docs/api/compose.html), [Recompose](https://github.com/acdlite/recompose/blob/master/docs/API.md#compose), [Lodash](https://lodash.com/docs/4.17.15#flow) and [Ramda](https://ramdajs.com/docs/#compose)

You can enable this common `compose` pattern like so:

```js
declare var compose: $Compose;
declare var composeReverse: $ComposeReverse;
```

More info: https://github.com/facebook/flow/commit/ab9bf44c725efd2ed6d7e1e957c5566b6eb6f688

## `$CharSet`

Handy utility for things like regexp flags or for any other case where the flags (chars) cannot repeat.

```js
type RegExpFlags = $CharSet<'gimsuy'>;

const a: RegExpFlags = 'miug'; // OK
const b: RegExpFlags = 'iii'; // not OK! ("i" is duplicated)
const c: RegExpFlags = 'abc'; // not OK! ("a", "b", "c" are not valid members)
```

- https://github.com/facebook/flow/issues/4654
- https://github.com/microsoft/TypeScript/issues/6579

## Contributing to native libdevs

First, you have to build Flow locally (check official README for updated instructions: https://github.com/facebook/flow):

```bash
brew install opam                       # http://opam.ocaml.org/
opam init
opam switch create . --deps-only -y     # install OCaml and Flow's dependencies
eval $(opam env)                        # probably not necessary, read `opam init` step
make
```

Now, you can start making changes to libdevs:

```bash
make
bash ./runtests.sh -t node_tests bin/flow
bash ./runtests.sh -t node_tests -r bin/flow    # to write snapshots
```

Please note, building like this should be faster for local development:

```bash
make build-flow-debug
```

Now, you can use this new binary from source in your application to test new features. Just use newly built `bin/flow` instead of Flow binary from NPM like so:

```bash
bin/flow status <ROOT>
```

What to do when something goes wrong during the build?

```bash
make clean
make build-flow-debug
```

## Types-first architecture

See: https://medium.com/flow-type/types-first-a-scalable-new-architecture-for-flow-3d8c7ba1d4eb

Types-first is a new architecture which significantly improves the overall performance. It requires a bunch more type annotations (which is why it wasn't recommended widely yet) but it's production-ready and everything is way faster if you add the needed type annotations. You can enable it like so:

```ini
[options]
experimental.well_formed_exports=true
experimental.types_first=true
```

It requires that every exported type be annotated and I'd certainly recommend it to anyone starting a new project in Flow.

> The reason we needed to get the enforcement right for input positions in 0.85 was to enable things like flow autofix to actually be able to infer types. With the input annotations we can infer the output ones and insert them via a codemod!

You can codeshift the codebase like so (see: https://flow.org/en/docs/cli/annotate-exports/; manual changes recommended):

```text
flow codemod annotate-exports \
  --write \
  --repeat \
  --log-level info \
  /path/to/folder \
  2> out.log
```

It is better to migrate the codebase gradually step by step like so:

```ini
experimental.types_first=false
experimental.well_formed_exports=true
experimental.well_formed_exports.includes=<PROJECT_ROOT>/src/packages/js
experimental.well_formed_exports.includes=<PROJECT_ROOT>/src/packages/relay
; ...
```

_Q_: How your FB colleagues accepted the types-first migration? It sometimes feels quite controversial :grimacing: Maybe some article would be nice - are you planning it?<br/>
_A_: some people were a little bit grumpy about the additional annotation burden at first, but everyone seems to be used to it now and there wasn't any real pushback, it was mostly just some grumbling here and there

Thanks to Flow team for a great insights on this topic!

## Trust mode

_TODO_

- https://github.com/facebook/flow/commit/959b4bad08ebf9fb2c2d4446653b8192bd0eb7d8
- https://github.com/facebook/flow/commit/fa0a8ef899925ed2b0939cb03678eceec821f5d0

## Enums

```js
const Enum = Object.freeze({
  X: 'x',
  Y: 'y',
});

type EnumT = $Values<typeof Enum>;
('a': EnumT);
```

Results in:

```text
7: ('a': EnumT);
    ^ Cannot cast `'a'` to `EnumT` because string [1] is incompatible with enum [2].
   References:
   7: ('a': EnumT);
       ^ [1]
   7: ('a': EnumT);
            ^ [2]
```

See:

- https://github.com/facebook/flow/commit/7c3390f7dcf886b0b39acfa505446614641ecb92
- https://github.com/facebook/flow/blob/369e1f93dde1ddafec7c5539a20e7b061672da6c/tests/values/object_types.js

Please note: this only works when you define the object with values inside `Object.freeze`. Similar but alternative approach: https://github.com/facebook/flow/issues/627#issuecomment-389668600

### Large unions (simple enums) performance

> I've been working on this recently so I can give you an overview. Essentially the reasons large unions are slow is that the amount of work Flow needs to do can grow exponentially with the size of the union. To determine if a union is a valid subtype of another type, we need to consider whether every single element of the union is a valid subtype, while to determine if it's a supertype we need to check if at least one of its cases is a supertype. If the union isn't actually a supertype we end up needing to check every case. Where this gets really bad is when we compare two union types, and this can easily result in an exponential case where we need to perform a lot of work for every single combination of union cases.

> Luckily we have a lot of optimizations in place for dealing with unions, especially those that can be simplified to enums (unions of strings or number literals). 450 variants is really not that large in the scheme of things; we deal with unions with upwards of 100,000 elements routinely. The only caution I would suggest is to make sure that you don't add non-literal types to your enum unions, because that will cause our optimizations to fail and leave you with the worst case peformance.

Thanks @sainati on Discord.

### Built-in enums

Please note: this feature is definitely **not** recommended to use. It's not finished, highly experimental and very controversial (+not supported in many tools). You can enable it like this:

```ini
[options]
experimental.enums=true
```

And now, you can use this new feature:

```flow js
enum E1 {
  A,
  B,
}

enum E2 of number {
  A = 1,
  B = 2,
}
```

This feature is NOT finished yet so typechecking doesn't work well (only parsing basically):

```js
const a: string = E2.A; // mmm...
const b: E2 = 1;
const c: E2 = '1'; // mmm...
const d: E2 = E2.A;
```

Basic methods support:

```flow js
enum E {
  A,
  B,
}

E.cast('A'); // string
E.members(); // Iterable<E>
E.isValid('A'); // boolean
```

- https://github.com/gkz/enums
- https://github.com/facebook/flow/issues/7837
- https://github.com/babel/babel/pull/10344
- https://github.com/prettier/prettier/pull/6833
- https://github.com/babel/babel-eslint/commit/2c754a8189d290f145e23ac962331fd1abd877bd
- https://github.com/facebook/flow/commit/bc4dcba789b3e1dc1def85787e67cf8e1e85755b

## Callable objects

This type allows you to use the function as a function and as an object at the same time.

```flow js
type MemoizedFactorial = {|
  +cache: {
    [number]: number,
  },
  (number): number,
|};

const factorial: MemoizedFactorial = (n) => {
  if (!factorial.cache) {
    factorial.cache = {};
  }
  if (factorial.cache[n] !== undefined) {
    return factorial.cache[n];
  }
  factorial.cache[n] = n === 0 ? 1 : n * factorial(n - 1);
  return factorial.cache[n];
};
```

See: https://flow.org/en/docs/types/functions/#toc-callable-objects

Alternatively, you can use so called _internal slot property_:

```flow js
type MemoizedFactorial = {|
  +cache: {
    [number]: number,
  },
  [[call]](number): number,
|};
```

- https://github.com/facebook/flow/pull/7790/files
- https://github.com/niieani/typescript-vs-flowtype/pull/55/files
- https://github.com/facebook/flow/commit/954a72704a6338778c940239573045b28c716488
- https://github.com/facebook/flow/commit/732eae55e102cdb7dfa7b6a85f0147d48c3afed7

Support of internal slot properties in tools like Babel or Eslint is not great though.

## Interesting Flow commands

```text
y flow graph cycle src/incubator/graphql/src/public/FAQ/types/outputs/FAQArticle.js

# Outputs dependency graphs of flow repositories. Subcommands:
# cycle: Produces a graph of the dependency cycle containing the input file
# dep-graph: Produces the dependency graph of a repository
```

```text
y flow dump-types src/packages/relay/src/QueryRenderer.js
```

```text
y flow check --debug
```

```text
y flow check --profile
```

```text
y flow autofix
```

[source](https://stackoverflow.com/a/40569640/3135248)

## Exact Objects by Default

TODO

- https://medium.com/flow-type/on-the-roadmap-exact-objects-by-default-16b72933c5cf
- https://github.com/facebook/flow/commit/1ac913040f38309480934ccb6717a3ffc65094a8

## Private object properties

```js
class Thing {
  prop1: string;
  #prop2: string = 'I am private!';
}

new Thing().prop1;
new Thing().prop2; // <- ERROR
```

```
7: (new Thing()).prop2;
                 ^ Cannot get `new Thing().prop2` because property `prop2` is missing in `Thing` [1].
    References:
    7: (new Thing()).prop2;
        ^ [1]
```

- https://github.com/tc39/proposal-class-fields#private-fields

## Predicate functions with `%checks`

`%checks` is an experimental predicate type. Check this code (no Flow errors):

```js
function isGreaterThan5(x: string | number): boolean {
  if (typeof x === 'string') {
    return parseInt(x) > 5;
  }
  return x > 5;
}
```

But you can slightly refactor it and you'll get unexpected errors:

```js
function isString(y): boolean {
  return typeof y === 'string';
}

function isGreaterThan5(x: string | number): boolean {
  if (isString(x)) {
    return parseInt(x) > 5;
  }
  return x > 5;
}
```

```
9:   return x > 5;
            ^ Cannot compare string [1] to number [2].
References:
5: function isGreaterThan5(x: string | number) {
                              ^ [1]
9:   return x > 5;
                ^ [2]
```

This is because the refinement information of `y` as `string` instead of `string | number` is lost when returning from the `isString` function. You have to fix it like this:

```js
function isString(y): boolean %checks {
  return typeof y === 'string';
}
```

You can also declare the predicate like this:

```js
declare function isSchema(schema: mixed): boolean %checks(schema instanceof GraphQLSchema);
```

- https://flow.org/en/docs/types/functions/#toc-predicate-functions
- https://github.com/facebook/flow/issues/3048
- https://github.com/facebook/flow/issues/34

## Object indexer properties

```js
const o: { [string]: number, ... } = {};
o['foo'] = 0;
o['bar'] = 1;
o[42] = 2; // Error!
const foo: number = o['foo'];
```

> When an object type has an indexer property, property accesses are assumed to have the annotated type, even if the object does not have a value in that slot at runtime. It is the programmer’s responsibility to ensure the access is safe, as with arrays.

This is similar to the issue with [possibly undefined array elements](#possibly-undefined-array-elements):

```js
const obj: { [number]: string, ... } = {};
obj[42].length; // No type error, but will throw at runtime!
```

Indexer properties can be also mixed with normal properties:

```js
const obj: {
  size: number,
  [id: number]: string,
  ...
} = {
  size: 0,
};

function add(id: number, name: string) {
  obj[id] = name;
  obj.size++;
}
```

---

_This information is not valid from version 0.126.0+ since indexer properties are now implemented even for exact objects, see: https://github.com/facebook/flow/commit/97e3a103227a381de0fd0be197bf25f6d6b6081a._

Please note: indexer property doesn't make any sense on exact objects:

```js
type S = {| [key: string]: string |};
const s: S = { key: 'value' }; // cannot assign object literal to `s` because property `key` is missing in `S`
```

This is not a bug - it simply doesn't make any sense with exact objects. It could be eventually repurposed though, see: https://github.com/facebook/flow/issues/7862 (related issue: https://github.com/facebook/flow/issues/3162)

However, it still somehow works when you are not trying to re-assign the whole value, see:

```js
type S = {| [key: string]: string |};

const s: S = { key: 'value' }; // Error: Cannot assign object literal to `s` because property `key` is missing in `S` but exists in object literal.

function test(x: S): string {
  x.y = 'string'; // OK (would fail with a wrong type assignment)
  return x.test; // OK
}
```

## Difference between `&` and `...`

It's easy to misunderstand the difference between intersection types (`A & B`) and spreading types (`{ ...A, b:boolean }`) in Flow.

```js
type A = { a: number };
type B = { b: boolean };
type C = { c: string };

// Intersection types are the opposite of union types!
const a: A & B & C = {
  a: 1,
  b: true,
  c: 'ok',
};

const b: $Exact<A> | $Exact<B> | $Exact<C> = {
  a: 1,
  //  b: true,
  //  c: 'ok'
};

const c: {
  ...{ a: number, b: string },
  a: string,
} = {
  a: '1', // only string, no number
  b: '2',
};

const d: {|
  ...{| a: number, b: string |},
  a: string,
|} = {
  a: '1', // works the same with exact types
  b: '2',
};

// Impossible type:
// const e: {| a: number |} & {| a: string |} = {
//   a: ???,
// }
```

No errors!

- https://www.knyz.org/blog/post/flow-union-intersection-spead/

## `@flow` pragma consequences

It is incorrect to assume that adding `@flow` pragma to your files just enables typesystem without any side-effects. There can be some unexpected changes in fact:

1. `/*:: ... */` and `/*: ... */` comments have special meaning (https://flow.org/en/docs/types/comments/)
2. `a<b>(c)` becomes a type argument, rather than `((a < b) > c)`

https://github.com/facebook/flow/issues/7928#issuecomment-511428223

## Conditions in Flow using `$Call`

- https://gist.github.com/miyaokamarina/934887ac2aff863b9c73283acfb71cf0
- https://flow.org/en/docs/types/utilities/#toc-call
- https://github.com/niieani/typescript-vs-flowtype/issues/37

> (jbrown215)
> \$Call is a shitty syntax for conditional types, but it exists. You can use overloading to simulate the cases, so:

```flow js
type Fun = ((number) => string) & ((string) => number);
type SwapNumberAndString<T: number | string> = $Call<Fun, T>;
```

Is approximately in TS:

```ts
type SwapNumberAndString<T extends number | string> = T extends number ? string : number;
```
