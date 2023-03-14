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

> Интересная особенность: если перед enum поставить const, то в рантайме все использования enum превратяться в константные значения и при разработке у нет возможности использовать enum в обе стороны

```ts
const enum Error {
    NotFound = 404,
    ServerUnavailable = 503
}

console.log(Error[404]) // теперь так делать нельзя 
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

#### Ассоциативные типы

В typescript нет ассоциативных типов, но с помощью this полиморфизма можно добиться похожего поведения

```ts
class FormWidget {
    readonly Value!: unknown

    protected value!: this['Value'] 

    getValue(): this['Value'] {
        return this.value
    }
}

class InputWidget extends FormWidget {
    override readonly Value!: string // переписали Value с новым типом, теперь все остальные свойства и методы автоматически поменяли тип на string
}

new InputWidget().getValue().trim() // ts понял, что getValue() вернет строку, а не unknown
```

#### readonly свойства

readonly свойство нельзя можно изменять в конструкторе

```ts
class Foo {
    readonly prop: string

    constructor() {
        this.prop = 'some string'
    }
}
```

Нужно помнить, что readonly делает проверку только на уровне статического анализа. В исходящем js файле свойство readonly просто убирается(свойство не замораживается - freez)

Также в ts есть следующие типы для структур только на чтение

- ReadonlyArray
- ReadonlyMap
- ReadonlySet

#### Кортежи

> Кортеж - массив фиксированной длины, каждому элементу которого можно задать тип.

```ts
const arr: [string, number, boolean] = ['', 1, true]
```

Также в кортеже можно использовать rest оператор "..."

```ts
const arr: [string, ...number] = ['', 1, 1, 1 ... и тд ]

// кортеж заканчивается и начинается строковым элементом
const arr2: [string, ...number, string] = ['', 1, 1, 1, '']

// несколько кортежей делать нельзя
const arr3: [string, ...number, ...string]
```

#### Оператор in keyof typeof

```ts
type A = {
    [key in 'a' | 'b']: number
}

const a: A = {
    a: 1,
    b: 2
}
```

```ts
interface Obj {
    a: string
    b: number
    c: boolean
}

type A = {
    [K in keyof Obj]: Obj[K]
}

const a: A = {
    a: '',
    b: 2,
    c: true
}
```

```ts
const obj = {
    a: '',
    b: 2,
    c: true,
}

type A = {
    [K in keyof typeof obj]: typeof obj[K]
}

const a: A = {
    a: '',
    b: 2,
    c: true
}
```

#### Оператор readonly с индексным типом

Делаем все поля readonly

 ```ts
interface Obj {
    a: string,
    b: number,
    c: boolean,
}

type ReadonlyObj = {
    readonly [K in keyof Obj]: Obj[K]
}
```

Обратная ситуация.

 ```ts
interface Obj {
    readonly a: string,
    readonly b: number,
    readonly c: boolean,
}

// так как мы берем значения из интерфейса по ключу, то readonly свойство тоже будет наследоваться
// что это убрать нужно добавить "-"
type ReadonlyObj = {
    -readonly [K in keyof Obj]: Obj[K]
}
```

#### Параметрические типы и обобщенные функции(generic)

 ```ts
function fn<T>(value: T): T {
    return value
}

// ts сам вычисляет, какой тип будет на выходе
fn(1).toFixed()
fn('').trim()
```

также можно задать значение параметрического типа по умолчанию

 ```ts
function fn<T = number>(value: string): T {
    // ...
    return value
}

fn('').toFixed()
```

Можно также указать, что тип должен принадлежать какому-то подтипу

 ```ts
function fn<T extends Array<any>>(value: string): T {
    // ...
    return value
}

fn<number[]>('')
```

#### Выведение параметрических типов

 ```ts
function map<
    A extends [],
    F extends (el: A extends Array<infer V> ? V : unknown) => any
>(arr: A, fn: F): F extends (...args: any[]) => infer R ? R[] : unknown {
    return <any>arr.map(fn)
}
```

### Касты типов

```ts
interface O {
    o: string
}

// downcast
const a = <O>{} // будет работать так как O подтип объекта
a.o // ts не будет ругаться(хотя свойства "o" не существует)

// downcast
const b = <O>{a: 'aaa'} // будет ругаться так как справа от <O> не родительский тип, а вообще другой объект

const c = <O><any>{a: 'aaa'} // можно обмануть ts и поставить каст <any>

// upcast
const d = <Object>{o: 'asdf'} // все ок
b.o // ts будет говорить, что "о" не существует в Object
```

### Параметрические типы методов

Параметрические типы могут указываться не только, как входные параметры какого-то типа/интерфейса/класса, но и для отдельного метода/функции
Это позволяет сделать вывод типа из контекста

```ts
type Fn<T extends string | number> = (val: T) => T
// TS будет ругаться, что generic тип должен иметь входной параметр 
// Так же в таком случае TS не может вывести тип у val и будет ругаться, что он имеет тип any
const fn: Fn = (val) => val

// вот так все будет работат
const fn: Fn<string> = (val) => val
```

Укажем параметрически тип теперь для самой функции

```ts
type Fn = <T extends string | number>(val: T) => T

// Теперь TS выведет тип для val - string | number 
const fn: Fn = (val) => val
```

### Стандартная библиотека TS

#### Awaited

Позволяет достать значение, которо возвращает Promise

```ts
let a: Awaited<Promise<1>> // а будет иметь тип 1
let b: Awaited<Promise<string>> // b будет иметь тип string
```

#### NonNullable

Исключает из типа null и undefined

```ts
type A = NonNullable<string | null | undefined> // A имеет тип string
```

Все утилиты [в официальной доке](https://www.typescriptlang.org/docs/handbook/utility-types.html#nonnullabletype)

### Файлы d.ts

Позволяет описывать типы для различных функций/методов и тд в подключаемых библиотеках
