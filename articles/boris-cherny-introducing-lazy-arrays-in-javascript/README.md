# Ленивые массивы в JavaScript

*Перевод статьи [Boris Cherny](https://github.com/bcherny): [Introducing: Lazy Arrays in JavaScript](https://performancejs.com/post/ewffd34/Introducing:-Lazy-arrays-in-JavaScript). Опубликовано с разрешения автора.*

![](./lazy.gif)

Сегодня я представляю [lazy-arr](https://github.com/bcherny/lazy-arr), привносящую ленивые массивы в JavaScript.

## Что такое ленивый массив и почему он полезен?

Давайте вспомним свое первое собеседование по разработке программного обеспечения: напишем функцию Фибоначчи. Мы определяем базовые случаи `0` и `1`, а затем рекурсивно генерируем остальные:

```js
let fib = n => {
  switch (n) {
    case 0: return 0
    case 1: return 1
    default: return fib(n - 1) + fib(n - 2)
  }
}
```

Проще некуда.

В чем проблема этого решения? Ну, это действительно неэффективно. Чтобы вычислить `n`-е число Фибоначчи, мы вызовем `fib` `2` в степени `n-1` раз! Ясно, что это отстой, и мы должны делать лучше.

Один из способов сделать это - запомнить вывод `fib`. То есть, поскольку `fib` является чистой и идемпотентной функцией, мы можем кэшировать ее вывод:

```js
let memoize = fn => {
  let cache = new Map
  return _ => {
    if (!cache.has(_)) {
      cache.set(_, fn(_))
    }
    return cache.get(_)
  }
}

let fib = memoize(n => {
  switch (n) {
    case 0: return 0
    case 1: return 1
    default: return fib(n - 1) + fib(n - 2)
  }
})
```

Ах, уже лучше! Теперь мы вызываем `fib` только `n - 1` раз для ввода размера `n`. Как еще мы можем сделать это?

## Ленивые последовательности

В Scala вы можете сделать это так (авторство [Philipp Gabler](https://github.com/phipsgabler)):

```
def fib(n: Int): Stream[Int] = {
  lazy val stream: Stream[Int] = 0 #:: stream.scan(1)(_ + _)
  stream.take(n)
}
```

Здесь определяется ленивый поток чисел (два начальных числа и функция, генерирующая больше чисел), а когда вы вызываете функцию `fib(n)`, она возвращает `n`-е число или генерирует и возвращает его, если оно все еще не было вычислено. Еще один способ посмотреть на это, как на генератор и кеш, отслеживающий ранее созданные значения, которые вы можете получить с помощью индекса значения.

Ленивые потоки - это действительно крутая абстракция для работы с последовательностями, которые либо дорого рассчитать, либо невозможно вычислить для всех индексов (бесконечные последовательности). Они популярны в функциональных языках, особенно в языках с ленивым вычислением по умолчанию. Например, вот так это можно сделать в Haskell:

```
fibs :: [Integer]
fibs = 0 : 1 : zipWith (+) fibs (tail fibs)
```

И та же идея, но в Clojure:

```
(defn fib []
  ((fn rfib [a b]
       (cons a (lazy-seq (rfib b (+ a b)))))
    0 1))
```

Ну, вы поняли.

Итак, как вы это можете сделать в JavaScript?

## Ленивые последовательности в JavaScript

С [lazy-arr](https://github.com/bcherny/lazy-arr) вы можете это сделать так:

```js
let fibs = lazy([0, 1])(_ => fibs[_ - 1] + fibs[_ - 2])
```

И это всё! Затем вы можете обращаться к элементам в массиве по мере необходимости, и они будут вычисляться по требованию:

```js
fibs[10] // 55
fibs[7]  // 13
```

В первой строке вычисляется десятый номер Фибоначчи, и поскольку мы определили, что вычисление рекурсивно (в терминах предыдущих чисел Фибоначчи), нам нужно вычислить первые девять чисел Фибоначчи, чтобы вычислить десятую. Поэтому, когда мы вычисляем седьмой номер Фибоначчи во второй строке, результат возвращается мгновенно, потому что мы уже вычислили его!

Хорошая новость для обеспокоенного пользователя. Массив `fibs` не делает ничего лениво или с состоянием, или рекурсивно, или что угодно в этом роде. Это всего лишь массив. Всё зашито `lazy-arr`, а генератор - одна строка.

Аккуратно, да?

---

*Слушайте наш подкаст в [iTunes](https://itunes.apple.com/ru/podcast/девшахта/id1226773343) и [SoundCloud](https://soundcloud.com/devschacht), читайте нас на [Medium](https://medium.com/devschacht), контрибьютьте на [GitHub](https://github.com/devSchacht), общайтесь в [группе Telegram](https://t.me/devSchacht), следите в [Twitter](https://twitter.com/DevSchacht) и [канале Telegram](https://t.me/devSchachtChannel), рекомендуйте в [VK](https://vk.com/devschacht) и [Facebook](https://www.facebook.com/devSchacht).*

[Статья на Medium](https://medium.com/devschacht/boris-cherny-introducing-lazy-arrays-in-javascript-d7f9da531820)