## TypeScript

### Типы данных в JS

#### Примитивные типы

number - Double IEE754

bigint

string - UTF-16

boolean - 1 байт
0000 0001 - true
0000 0000 - false

symbol

null - пустое значение
undefined - значение не было присвоено

#### Объекты

object

### Типизация

- Явная / неявная
- Слаба(возможно приведение типов) / Сильня
- Динамическая(на этапе компиляции типы неизвестны) / Статическая

### JSDoc

До TS использовали различные форматы комментариев(например, JSDoc) и IDE могла подсветить, если есть ошибка в типах

```js
/**
 * @param {number} a
 * @returns {number}
 */
function foo(a) {
    return a
}
```

### Runtime конструкции в TS

#### Enum

```ts
enum Error {
    NotFound = 404,
    ServerUnavailable = 503
}
```

#### Декораторы

```ts
class Foo {
    @depricated
    foo() {
        return 'some value'
    }
}

function depricated(target, key, desc) {
    const fn = target[key]

    return {
        ...desc,
        value: function() {
            console.log('Method is depricated');
            return fn.apply(this, arguments)
        }
    }
}
```

### Размеченное объединение типов

```ts
function foo(a: string | string[] | number) {
    if (typeof a === 'number') {
        a.toFixed()
    } 

    if (isArray(a)) {
        a.findIndex((el) => el === 'some string')
    }
}

function isArray(obj: unknown): obj is Array<string> {
    return Array.isArray(obj)
}
```

### Пересечение типов

```ts
class Foo {
    foo: string = '123'
}

class Bar {
    bar: number = 123
}

function fb(a: Foo & Bar) {
    a.bar
    a.foo
}

fb({...new Foo(), ...new Bar()})
```

### Константные типы

```ts
function insertAdjacentHTML(
    str: string,
    append: 'afterbegin' | 'afterend' | 'beforebegin' | 'beforeend'
) {
    // ...
}
```

### Типы на основе шаблонов

```ts
function insertAdjacentHTML(
    str: string,
    append: `${'after' | 'before'}${'begin' | 'end'}`
) {
    // ...
}
```

### Типы символов

```ts
let a = Symbol('123')
let b = Symbol('456')
let c = Symbol('789')

function insertAdjacentHTML(
    str: string,
    append: typeof a | typeof b
) {
    // ...
}

// будет валидный код, так как a и b переменные типа let и ts приведете свойство append к "symbol"
insertAdjacentHTML('str', c) 
```

Можно это исправить, изменив **let** на **const**

```ts
const a = Symbol('123')
const b = Symbol('456')
let c = Symbol('789')
```

Также можно добавить тип **unique symbol**, но const в данном случае уже будет достаточно

```ts
const a: unique symbol = Symbol('123')
const b: unique symbol = Symbol('456')
let c = Symbol('789')
```

### Алиасы типов

использование алисов делает код более понятным и убирает "нагромождение" типов

```ts
type AdjacentMode = `${'after' | 'before'}${'begin' | 'end'}`
type SomeType = string | number | boolean

foo(mode: AdjacentMode): SomeType
```

### Интерфейсы

```ts
interface Options {
    opt1: boolean
    opt2: boolean
}
```

> С помощью интерфейсов можно делать все тоже самое, что и при использовании type, но есть отличия

При колизии интерфейсы объединяются

```ts
interface Options {
    opt1: boolean
    opt2: boolean
}

interface Options {
    opt3: boolean
}
```

Также объединяются class и interface с одинаковыми названиями

```ts
interface Options {
    opt1: boolean
    opt2: boolean
}

class Options {
    opt3: string = '123'
}

// данный экземпляр будет иметь свойства из интерфейс и из класса 
const opts = new Options()
opts.opt1
opts.opt2
opts.opt3
```

C помощью интерфейсов нельзя делать пересечения и объединения, как это делается в **type** (операторы "&" "|")

### Тип функции

С помощью type

```ts
type Funnction = (args: unknown) => void
```

С помощью interface

```ts
interface Function {
    handler: (args: unknown) => void
}

// короткая запись
interface Function {
    (args: unknown): void
}
```

### Перегрузка функции

```ts
function widget(name: string): Widget | null
function widget(name: string, params?: {}): true

function widget(name: string, params?: {}): Widget | null | true {
    if (arguments.length === 1) {
        return getWidget()
    }

    return regWidget()
}
```

В объекте также можно сделать перегрузку

```ts
// принципиально записывать функцию не как свойство, а именно метод
interface Obj {
    foo(a: string): string
    foo(a: number): number 
}

const obj: Obj = {
    foo(a) {
        // ts не может вывести тип, поэтому приходится писать <any >
        return <any>a
    }
}
```

Перегрузка в классе

```ts
class Obj {
    foo(a: string): string
    foo(a: number): number
    foo(a: string | number): string | number {
        return a
    }
}
```

### Встроенные типы в TS

- any
- unknown
- never
- void - любой тип, но мы с ним ничего не можем сделать

> Многие ошибочно думают, что если у функции возвращаемый результат void, то функция ничего не возвращает. На самом деле функция может вернуть любой результат просто мы не можем с этим результатом ничего сделать

```ts
// функция никогда не вернут результат
function foo(): never {
    while (true) {
        // ...
    } 
}

function bar(): void {
    console.log('123')
}

function bar2(): void {
    console.log('123')
    return 12312 // ничего не сможем сделать с этим резултатом
}
```

### Коллизии типов свойств

Коллизии желательно избегать
При пересечении примитивных типов ts приводит пересечение к never

```ts
interface Foo {
    a: string
} 

interface Bar extends Foo {
    a: number
    b: string // возникнет коллизия
} 
```

```ts
type Foo = {
    a: string
} 

type Bar = Foo & {
    a: number 
    b: string // возникнет коллизия
} 
```

### Классы в TS

#### Модификаторы области видимости

**protected** - доступно для дочерних классов, но не доступно снаружи

**private** - лучше не использовать, в js приватные свойства и методы пишутся #something. Также private создает коллизии

#### Реализация интерфейсов

Класс может имплементирвать сколько угодно интерфейсов

```ts
interface Server {
    post(body: any): void
    get(): void  
}

interface Server2 {
    delete(): boolean 
}

class MyServer implements Server, Server2 {
    post(body: any): void {
        
    }

    get(): void {
        
    }

    delete(): boolean {
        return true
    }
}
```

Также класс можно использовать, как интерфейс

> P.S. следует отметить, что в интерфейсе не может быть protected и private свойств, поэтому при использовании класса, как интерфейса, такие свойства будут проигнорированы

#### Абстрактные классы

Абстрактный класс может иметь, как реализованные методы,  так и абстрактные методы и свойства, которые нужно реализовать в конкретных классах

```ts
abstract class Server {
    abstract post(body: any): void
    
    delete(): boolean {
        // ...
    } 
}
```

#### Строгоинициализированные свойства

Могут быть ситуации, когда дефолтная реализация для свойства задается в конструкторе и ts не всегда может это проанализировать

```ts
class Foo {
    a: string // ts будет ругаться, что свойство не проинициализровано(хотя это не так)
    a!: string   // можно добавить Non-null assertion operator, чтоб сказать ts, когда мы точно уверены, что свойство не null или undefined

    constructor(a: string) {
        this.initA(a)
    }

    initA(a: string) {
        this.a = a
    }
}
```

#### this полиморфизм

```ts
class Foo {
    bar(): Foo {
        return this
    }
}

class Foo2 extends Foo {
    bar2(): Foo2 {
        return this
    }
}

// ts напишет, что bar2 doesn't exist, так как мы прописали, что возвращаемый тип у bar(): Foo
new Foo().bar().bar2().bar()
```

Можно поставить this вместо конкретного типа

```ts
class Foo {
    bar(): this {
        return this
    }
}

class Foo2 extends Foo {
    bar2(): this {
        return this
    }
}

// Теперь ts не будет "ругаться"
new Foo().bar().bar2().bar()
```