# Ограниченный полиморфизм

Иногда недостаточно просто сказать: «Этот элемент имеет некий обобщенный тип T, и другой элемент тоже должен его иметь». В некоторых случаях вы хотите сказать: «Тип U должен быть как минимум T». Мы зовем это установкой верхней границы U.

Зачем? Представим реализацию двоичного дерева с тремя типами узлов:

1. Обычные TreeNodes.
2. LeafNodes, являющиеся TreeNodes, не имеющими дочерних узлов.
3. InnerNodes, являющиеся TreeNodes, имеющими дочерние узлы.

Начнем с объявления типов для узлов:

```
type TreeNode = {
    value: string
}
type LeafNode = TreeNode & {
    isLeaf: true
}
type InnerNode = TreeNode & {
    children: [TreeNode] | [TreeNode, TreeNode]
}
```

Здесь мы говорим, что TreeNode является объектом с одним свойством — value. Тип LeafNode имеет те же свойства, что и TreeNode, плюс свойство isLeaf, которое всегда true. InnerNode также имеет все свойства TreeNode плюс свойство children, указывающее на один или два дочерних узла.

Теперь напишем функцию mapNode, которая получает TreeNode, делает отображение на его значение и возвращает новый TreeNode. Нам нужно получить функцию mapNode, которую можно будет использовать так:

```
let a: TreeNode = {value: 'a'}
let b: LeafNode = {value: 'b', isLeaf: true}
let c: InnerNode = {value: 'c', children: [b]}

let a1 = mapNode(a, _ => _.toUpperCase()) // TreeNode
let b1 = mapNode(b, _ => _.toUpperCase()) // LeafNode
let c1 = mapNode(c, _ => _.toUpperCase()) // InnerNode
```

Стоп, а как можно написать функцию mapNode, получающую подтип TreeNode и возвращающую тот же самый подтип? При передаче в нее LeafNode должен возвращаться LeafNode, при передаче InnerNode — InnerNode, а при передаче TreeNode — TreeNode.

Вот ответ:

```
function mapNode<T extends TreeNode>( // 1
    node: T, // 2
    f: (value: string) => string
): T { // 3
return {
    ...node,
    value: f(node.value)
    }
}
```

1. mapNode — это функция, определяющая один параметр обобщенного типа — T. T имеет верхнюю границу в виде TreeNode. Это значит, что T — это либо TreeNode, либо подтип TreeNode.
2. mapNode получает два параметра. Первый — это node типа T, потому что в ❶ мы указали, что node extends TreeNode. Если мы передаем нечто не являющееся TreeNode — например, пустой объект {}, null или массив из нескольких TreeNode, — это сразу вызовет красное подчеркивание. node должен быть либо TreeNode, либо подтипом TreeNode
3. mapNode возвращает значение типа T. Вспомните, что T может быть либо TreeNode, либо подтипом TreeNode.
