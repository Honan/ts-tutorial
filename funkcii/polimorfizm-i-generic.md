# Полиморфизм и Generic

До сих пор мы рассматривали конкретные типы и функции, их использующие. Что же такое конкретный тип?

* boolean;
* string;
* {a: number} | {b: string};
* (numbers: number\[]) => number.

Конкретные типы полезны, когда вы точно знаете, какой тип ожидаете, и хотите сверить его с переданным типом. Но иногда нецелесообразно ограничивать поведение функции конкретным типом.

Представьте функцию filter для итерации по массиву и его очистки. В JavaScript она может выглядеть так:

```
function filter(array, f) {
    let result = []
    for (let i = 0; i < array.length; i++) {
        let item = array[i]
        if (f(item)) {
            result.push(item)
        }
    }
    return result
}
```

Предположим, у нас будет массив чисел. Объявим сигнатуру:

```
type Filter = {
    (array: number[], f: (item: number) => boolean): number[]
}
```

Эта сигнатура будет работает для массива чисел, но не для массивов строк, объектов, массивов и т. д. Попробуем использовать перегрузку для ее расширения, чтобы она работала для массива строк:

```
type Filter = {
    (array: number[], f: (item: number) => boolean): number[]
    (array: string[], f: (item: string) => boolean): string[]
}
```

Так можно расширять до бесконечности и кодовая база будет расти. Возникает вопрос. Что же делать?

Использовать Generics (обобщенные типы).

```
type Filter = {
    <T>(array: T[], f: (item: T) => boolean): T[]
}
```

Для объявления обобщенных типов используются угловые скобки (<>) (воспринимайте их как ключевое слово type). Место размещения скобок определяет диапазон охватываемых типов (есть всего несколько мест, где вы можете их поставить). TypeScript, в свою очередь, убеждается, что внутри этого диапазона все экземпляры параметров обобщенных типов привязаны к одному реальному типу. В текущем примере при вызове filter TypeScript привяжет конкретные типы к обобщенному типу T в зависимости от обозначенного скобками диапазона. Какой именно тип привязывать к T, он решит исходя из того, с каким типом мы вызовем filter. Между угловых скобок вы можете объявить столько обобщений, сколько пожелаете, разделив их точкой с запятой.

{% hint style="info" %}
T — это просто имя типа, такое же, как A, Zebra или 133t. Принято использовать имена, состоящие из одной заглавной буквы T или U, V, W и т. д., в зависимости от того, сколько обобщенных типов требуется. Если вы объявляете множество таких типов подряд или используете их сложным образом, рассмотрите вариант более описательных имен вроде Value или WidgetType.
{% endhint %}

```
// T привязан к number
filter([1, 2, 3], item => item > 2)

// T привязан к строке
filter(['a', 'b'], item  => item !== 'b')

// (c) T привязан к {firstName: string}
let names = [
    {firstName: 'beth'},
    {firstName: 'caitlyn'},
    {firstName: 'xin'}
]
filter(names, item => item.firstName.startsWith('b'))
```

Обобщенные типы — это эффективный способ выразить более обширное действие функции, чем это позволяют конкретные типы. Воспринимать же их стоит в виде ограничений. Как аннотирование параметра функции в виде n: number ограничивает значение параметра n типом number, так и использование обобщенного типа T ограничивает тип любого привязываемого к T условием типа быть одинаковым в каждом T.

### Когда привязывать конкретные типы к обобщенным

Место объявления обобщенного типа не только определяет его диапазон, но также указывает TypeScript, когда нужно привязать к нему конкретный тип. Из предыдущего примера:

```
type Filter = {
    <T>(array: T[], f: (item: T) => boolean): T[]
}

let filter: Filter = (array, f) =>
// ...
```

Мы объявили как часть сигнатуры вызова (перед открывающимися скобками), и TypeScript привяжет конкретный тип к T, когда мы вызовем функцию типа Filter.

Если бы мы вместо этого ограничили диапазон T псевдонимом типа Filter, TypeScript потребовал бы от нас при использовании Filter привязать тип явно:

```
type Filter<T> = {
    (array: T[], f: (item: T) => boolean): T[]
}

let filter: Filter = (array, f) => // Ошибка TS2314: обобщенный тип
// 'Filter' требует 1 аргумент типа.

let filter: Filter<number> = (array, f) =>
```

Чаще всего TypeScript будет привязывать конкретные типы к обобщенным, когда вы используете их для функций при их вызове, для классов при их инстанцировании или для псевдонимов типов и интерфейсов при их использовании или реализации.

### Где можно объявлять обобщенные типы

Полная сигнатура вызова с диапазоном T, включающим только одну сигнатуру. Поскольку диапазон T ограничен одной сигнатурой, TypeScript привяжет T к реальному типу в этой сигнатуре при вызове функции типа filter. Каждый вызов filter будет получать свою собственную привязку для T.

```
type Filter = {
    <T>(array: T[], f: (item: T) => boolean): T[]
}
let filter: Filter = // ...
```



Полная сигнатура вызова с диапазоном T, включающим все сигнатуры. T объявлен как часть типа Filter (а не часть конкретной сигнатуры типа), и TypeScript привяжет T, когда вы объявите функцию типа Filter.

```
type Filter<T> = {
    (array: T[], f: (item: T) => boolean): T[]
}
let filter: Filter<number> = // ...
```

Сокращенные сигнатуры вызова.

```
type Filter = <T>(array: T[], f: (item: T) => boolean) => T[] ❸
let filter: Filter = // ...

type Filter<T> = (array: T[], f: (item: T) => boolean) => T[] ❹
let filter: Filter<string> = //
```

Именованная сигнатура вызова функции с диапазоном T, ограниченным сигнатурой. TypeScript привяжет конкретный тип к T при вызове filter, и каждый вызов filter получит свою собственную привязку для T.

```
function filter<T>(array: T[], f: (item: T) => boolean): T[] {
// ...
}
```

### Про Promise

Поскольку TypeScript выводит конкретные типы для обобщенных на основе аргументов, передаваемых в обобщенную функцию, иногда может возникнуть такая ситуация

```
let promise = new Promise(resolve =>
    resolve(45)
)

promise.then(result => // Выведен как {}
    result * 4 // Ошибка TS2362: левая часть арифметической
) // операции должна иметь тип 'any', 'number',
// 'bigint' или enum.

```

Что происходит? Почему TypeScript вывел result как {}? Потому что мы не предоставили ему достаточно информации. TypeScript для вывода обобщенного типа использует только типы аргументов обобщенной функции, поэтому он по умолчанию вывел T как {}.

Чтобы это исправить, нужно явно аннотировать параметр обобщенного типа промисов:

```
let promise = new Promise<number>(resolve =>
    resolve(45)
)

promise.then(result => // number
    result * 4
)
```