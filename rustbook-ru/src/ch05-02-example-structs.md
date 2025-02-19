## Пример использования структур

Чтобы понять, когда нам может понадобиться использование структур, давайте напишем программу, которая вычисляет площадь прямоугольника. Мы начнём с использования одиночных переменных, а затем будем улучшать программу до использования структур.

Давайте создадим новый проект программы при помощи Cargo и назовём его *rectangles*. Наша программа будет получать на вход длину и ширину прямоугольника в пикселях и затем рассчитывать площадь прямоугольника. Листинг 5-8 показывает один из коротких вариантов кода который позволит нам сделать именно то, что надо, код в файле проекта *src/main.rs*.

<span class="filename">Имя файла: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:all}}
```

<span class="caption">Листинг 5-8: Вычисление площади прямоугольника, заданного отдельными переменными ширины и высоты</span>

Теперь, проверим её работу `cargo run`:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/output.txt}}
```

Этот код успешно вычисляет площадь прямоугольника, вызывая функцию `area` с каждым измерением, но мы можем сделать больше для повышения ясности и читабельности этого кода.

Проблема данного метода очевидна из сигнатуры `area`:

```rust,ignore
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:here}}
```

Функция `area` должна вычислять площадь одного прямоугольника, но функция, которую мы написали, имеет два параметра, и нигде в нашей программе не ясно, что эти параметры взаимосвязаны. Было бы более читабельным и управляемым сгруппировать ширину и высоту вместе. Мы уже обсуждали один из способов сделать это в разделе ["Тип кортеж"] главы 3: использование кортежей.

### Рефакторинг при помощи кортежей

Листинг 5-9 это другая версия программы, использующая кортежи.

<span class="filename">Имя файла: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-09/src/main.rs}}
```

<span class="caption">Листинг 5-9: Определение ширины и высоты прямоугольника с помощью кортежа</span>

С одной стороны, эта программа лучше. Кортежи позволяют добавить немного структуры, и теперь мы передаём только один аргумент. Но с другой стороны, эта версия менее понятна: кортежи не называют свои элементы, поэтому нам приходится индексировать части кортежа, что делает наше вычисление менее очевидным.

Если мы перепутаем местами ширину с высотой при расчёте площади, то это не имеет значения. Но если мы хотим нарисовать нарисовать прямоугольник на экране, то это уже будет иметь значение! Мы должны помнить, что ширина  `width` находится в кортеже с индексом `0`, а высота `height` с индексом `1`. Если кто-то другой поработал бы с кодом, ему бы пришлось разобраться в этом и также помнить про порядок. Легко забыть и перепутать эти значения и это вызовет ошибки, потому что данный код не передаёт наши намерения.

### Рефакторинг при помощи структур: добавим больше смысла

Мы используем структуры чтобы добавить смысл данным при помощи назначения им осмысленных имён . Мы можем переделать используемый кортеж в структуру с единым именем для сущности и частными названиями её частей, как показано в листинге 5-10.

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-10/src/main.rs}}
```

<span class="caption">Листинг 5-10: Определение структуры <code>Rectangle</code></span>

Здесь мы определили структуру и дали ей имя `Rectangle`. Внутри фигурных скобок определили поля как `width` и `height`, оба - типа `u32`. Затем в `main` создали конкретный экземпляр `Rectangle` с шириной в 30 и высотой в 50 единиц.

Наша функция `area` теперь определена с одним параметром названным `rectangle`, чей тип является неизменяемым заимствованием структуры `Rectangle`. Как упоминалось в Главе 4, необходимо заимствовать структуру, а не передавать её во владение. Таким образом функция `main` сохраняет `rect1` в собственности и может её использовать дальше, по этой причине мы и используем `&` в сигнатуре и в месте вызова функции.

Функция `area` имеет доступ к полям `width` и `height` экземпляра `Rectangle`. Сигнатура нашей функции для `area` теперь точно говорит, что мы имели в виду: посчитать площадь `Rectangle` используя поля `width` и `height`. Такой подход сообщает, что ширина и высота связаны по смыслу друг с другом. А названия значений структуры теперь носят понятные описательные имена, вместо ранее используемых значений индексов кортежа `0` и `1`. Это плюс к ясности.

### Добавление полезной функциональности при помощи Выводимых Типажей

Было бы полезно иметь возможность печатать экземпляр `Rectangle` во время отладки программы и видеть значения всех полей. Листинг 5-11 использует макрос [`println!` macro]<!--  -->, который мы уже использовали в предыдущих главах. Тем не менее, это не работает.

<span class="filename">Файл: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/src/main.rs}}
```

<span class="caption">Листинг 5-11: Попытка распечатать экземпляр <code>Rectangle</code></span>

При компиляции этого кода мы получаем ошибку с сообщением:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:3}}
```

Макрос `println!` умеет выполнять множество видов форматирования, по умолчанию фигурные скобки в `println!` говорят использовать форматирование известное как типаж `Display`: вывод в таком варианте форматирования предназначен для непосредственного использования конечным пользователем. Примитивные типы изученные ранее, по умолчанию реализуют типаж `Display`, потому что есть только один способ отобразить число `1` или любой другой примитивный тип пользователю. Но для структур у которых `println!` должен форматировать способ вывода данных, это является менее очевидным, потому что есть гораздо больше возможностей для отображения: Вы хотите запятые или нет? Вы хотите печатать фигурные скобки? Должны ли отображаться все поля? Из-за этой неоднозначности Rust не пытается  угадать, что нам нужно, а структуры не имеют встроенной реализации `Display` для использования в `println!` с заполнителем `{}`.

Продолжив чтение текста ошибки, мы найдём полезное замечание:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:9:10}}
```

Давайте попробуем! Вызов макроса `println!` теперь будет выглядеть так `println!("rect1 is {:?}", rect1);`. Ввод спецификатора `:?` внутри фигурных скобок говорит макросу `println!`, что мы хотим использовать другой формат вывода известный как `Debug`. Типаж `Debug` позволяет печатать структуру способом, удобным для разработчиков, чтобы видеть значение во время отладки кода.

Скомпилируем код с этими изменениями. Упс! Мы всё ещё получаем ошибку:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:3}}
```

Снова компилятор даёт нам полезное замечание:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:9:10}}
```

Rust *&nbsp;реализует* функциональность для печати отладочной информации, но <em>не включает (не выводит) её по умолчанию</em> , мы должны явно включить эту функциональность для нашей структуры. Чтобы это сделать, добавляем внешний атрибут <code>#[derive(Debug)]</code> сразу перед определением структуры как показано в листинге 5-12.

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-12/src/main.rs}}
```

<span class="caption">Листинг 5-12: Добавление атрибута для вывода типажа  <code>Debug</code> и печати экземпляра <code>Rectangle</code> с отладочным форматированием</span>

Теперь при запуске программы мы не получим ошибок и увидим следующий вывод:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-12/output.txt}}
```

Отлично! Это не самый красивый вывод, но он показывает значения всех полей экземпляра, которые определённо помогут при отладке. Когда у нас более крупные структуры, то полезно иметь более простой для чтения вывод; в таких случаях можно использовать код `{:#?}` вместо `{:?}` в строке макроса `println!`. В этом примере использование стиля `{:#?}` приведёт к такому выводу:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-02-pretty-debug/output.txt}}
```

Другой способ вывести значение в формате `Debug` — это использовать макрос  [`dbg!` macro]<!--  --> , который забирает во владение выражение, печатает номер файла и строки, где происходит `dbg!` вызов макроса в коде вместе с результирующим значением этого выражения и возвращает владение значения.

> Примечание: вызов `dbg!` макрос печатает в стандартный поток консоли для ошибок ( `stderr` ), а не в `println!` который печатает в стандартный поток консоли вывода ( `stdout` ). Подробнее о `stderr` и `stdout` мы поговорим в разделе [“Запись ошибок в поток вывод для ошибок вместо стандартного потока вывода” Главы 12]<!--  -->.

Вот пример, когда нас интересует значение, которое присваивается полю `width`, а также значение всей структуры в `rect1` :

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/src/main.rs}}
```

Можем написать макрос `dbg!` вокруг выражения `30 * scale`, потому что `dbg!` возвращает владение значения выражения, поле `width` получит то же значение, как если бы у нас не было вызова `dbg!`. Мы не хотим чтобы макрос `dbg!` становился владельцем `rect1` , поэтому мы используем ссылку на `rect1` в следующем вызове. Вот как выглядит вывод этого примера:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/output.txt}}
```

Мы можно увидеть, первый бит вывода поступил из строки 10 *src/main.rs*, там где мы отлаживаем выражение `30 * scale` и его результирующее значение равно 60 ( `Debug` форматирование, реализованное для целых чисел, заключается в печати только их значения). Вызов `dbg!` в строке 14 *src/main.rs* выводит значение `&rect1` , которое является структурой `Rectangle` . В этом выводе используется красивое форматирование `Debug` типа `Rectangle`. Макрос `dbg!` может быть очень полезен, когда вы пытаетесь понять, что делает ваш код!

В дополнение к `Debug` , Rust предоставил нам ряд `derive` , которые мы можем использовать с атрибутом производных, которые могут добавить полезное поведение к нашим пользовательским типам. Эти черты и их поведение перечислены в [Приложении C.] игнорировать . Мы расскажем, как реализовать эти трейты с пользовательским поведением, а также как создать свои собственные трейты в главе 10. Кроме того, есть много других `derive` ; для получения дополнительной информации смотри раздел [“Атрибуты” справочника Rust].

Функция `area` является довольно специфичной: она считает только площадь прямоугольников. Было бы полезно привязать данное поведение как можно ближе к структуре `Rectangle`, потому что наш специфичный код не будет работать с любым другим типом. Давайте рассмотрим, как можно улучшить наш кода превращая функцию `area` в <em>метод</em> <code>area</code>, определённый для типа `Rectangle`.


["Тип кортеж"]: ch03-02-data-types.html#the-tuple-type
[Приложении C.]: appendix-03-derivable-traits.md
[`println!` macro]: ../std/macro.println.html
[`dbg!` macro]: ../std/macro.dbg.html
[“Запись ошибок в поток вывод для ошибок вместо стандартного потока вывода” Главы 12]: ch12-06-writing-to-stderr-instead-of-stdout.html
[“Атрибуты” справочника Rust]: ../reference/attributes.html