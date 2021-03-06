# Введение

## Система типов

**Система типов** - набор правил, по котором программа понимает, как эти типы должны присвоены переменной.

Главным образом системы типов делятся на два вида: в одних вы должны сообщать компилятору тип каждого элемента посредством явного синтаксиса, другие выводят типы автоматически. Оба вида имеют как плюсы, так и минусы.

TypeScript создан на стыке этих двух видов: вы можете явно аннотировать типы либо позволить TypeScript делать их вывод за вас.

Для явного объявления типов используются аннотации**:**

```
let a: number = 1 // a является number
let b: string = 'hello' // b является string
let c: boolean[] = [true, false] // c является массивом booleans
```

Если вы хотите, чтобы TypeScript вывел за вас типы, то просто не прописывайте их:

```
let a = 1 // a является number
let b = 'hello' // b является string
let c = [true, false] // c является массивом booleans
```

{% hint style="info" %}
В большинстве случаев лучше позволять TypeScript выводить типы по мере его возможностей и снижать тем самым объем явно аннотированного кода до минимума.
{% endhint %}

### Конвертируются ли типы автоматически

JavaScript является слабо типизированным языком, поэтому если вы произведете недопустимое сложение, например числа и массива.

```
3 + [1]; // вычисляется как "31"
```

В то время как JavaScript старается произвести умные преобразования в стремлении вам помочь, TypeScript начинает указывать на ошибки, как только вы делаете что-либо неверно.

```
3 + [1]; // Ошибка TS2365: оператор '+'
// не может быть применен к типам '3'
// и 'number[]'.
```

Как только вы делаете нечто, что выглядит неправильным, TypeScript на это указывает. Неявное преобразование, которое производит JavaScript, может серьезно усложнить обнаружение источника ошибок, особенно при масштабировании проекта крупной командой разработчиков.

### Когда проверяются типы

TypeScript проверяет типы в процессе компиляции, поэтому вам не нужно запускать код, чтобы увидеть ошибку. TypeScript статически анализирует код на наличие подобных ошибок и показывает их еще **до запуска**.

![TS показывает ошибки до запуска](<.gitbook/assets/image (1) (1).png>)

К слову, есть множество возможных ошибок, которые TypeScript не может обнаружить при компиляции. К ним относятся переполнения стека, разрывы сетевых соединений и некорректный ввод данных пользователем.
