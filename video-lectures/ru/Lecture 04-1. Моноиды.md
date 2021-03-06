Моноиды
===

### Мотивирующий пример

Любые два объекта во Вселенной воздействуют друг на друга.
Мы можем смоделировать систему взаимодействующих тел, используя список объектов.
Вычислить ускорение каждого объекта мы можем, определив сумму сил, с которыми действуют на него остальные объекты, и поделив на массу исходного объекта. Помимо моделирования самих объектов, нас также интересует силовое поле, которое они порождают.
Как мы можем выделить это поле в программе? И как его отобразить?

### Сложение

Чтобы найти элегантное решение для задачи с системой тел, сделаем небольшое отступление.
Допустим, нам нужно посчитать сумму списка чисел. Мы, конечно, можем использовать свёртку списка.
При правой свёртке все скобки будут собираться справа, а при левой — слева.
Но это не единственная возможная конфигурация скобок, которая может привести нас к верному результату.
Действительно, операция сложения ассоциативна, а значит, мы можем расставить скобки, как хотим.
Например, мы можем разбить весь список чисел на куски, рассчитать каждый кусок при помощи функции `sum`, а затем сложить результаты для кусков.
Зачем нам разбивать список на куски?
Например, чтобы провести вычисления параллельно!
Заметим, что произвольно расставлять скобки мы можем далеко не для всех операций.
Для этого нам нужно, чтобы операция принимала и возвращала значения одного типа.
Кроме того, эта операция должна быть ассоциативной.
Какие ещё операции удовлетворяют этим условиям?
Аналогично сложению, умножение чисел также ассоциативно.
Логические операции `ИЛИ` и `И` ассоциативны.
Конкатенация строк (да и на самом деле любых списков) также ассоциативна.
Операция композиции функций тоже ассоциативна.
Однако, типы функций могут быть разными.
Мы можем привести типы всех функций к одному, используя один и тот же тип для аргумента и результата.
Вспомним, что список значений, который мы сворачиваем, может быть пуст.
Что мы делаем в этом случае?
При вычислении суммы мы возвращаем ноль. При вычислении произведения — единицу. При вычислении логического ИЛИ — `False`.
Если нам нужно склеить пустой список строк — мы возвращаем пустую строку.
В случае композиции пустого списка функций мы возвращаем функцию, которая ничего не делает — тождественную функцию `id`.
Заметим, что в каждом случае выбранное значение по умолчанию не меняет результат, если мы прибавим его используя нашу ассоциативную операцию.
Действительно, ноль плюс любое число — это то же число.
Умножение на единицу также не меняет число. Логическое ИЛИ с `False` не меняет булево значение.
Склейка с пустой строкой не меняет строку.
А композиция с тождественной функцией не меняет функцию.
Причём всё это верно, даже если мы поменяем операнды местами.

### Класс типов `Monoid`

Итак, чтобы производить вычисление с произвольным порядком скобок, нам необходима операция шага свёртки, которая принимает аргументы одного типа, ассоциативна и имеет некоторый нейтральный элемент.
Математическая структура, обладающая этими свойствами, называется *моноид*.
В Haskell моноид представлен классом типов `Monoid`. У этого класса 2 метода:
`mappend` — это ассоциативная операция, она принимает на вход два значения типа `a` и возвращает значение того же типа;
`mempty` — это нейтральный элемент относительно операции `mappend`.
Операцию `mappend` часто удобно использовать в её инфиксном варианте, который определён в модуле `Data.Monoid`.
Для наглядности и краткости, мы будем использовать этот вариант.
Для всех реализаций этого класса должны выполняться следующие законы:
Комбинация `mempty` с любым значением при помощи `mappend` оставляет исходное значение.
В любом порядке.
Операция `mappend` ассоциативна.
Мы можем определить реализацию экземпляра класса типов `Monoid` для списков.
`mempty` — это пустой список. А `mappend` — это обычная конкатенация списков.
Законы для этой реализации принимают следующий вид.
Очевидно, все законы выполняются.
Мы можем также определить реализацию экземпляра класса типов `Monoid` для логического ИЛИ.
Тип, для которого мы определяем экземпляр — `Bool`.
Нейтральный элемент — это `False`.
А `mappend` — это операция логического ИЛИ. Законы для этой реализации принимают такой вид и легко видеть, что они выполняются.

### Обёртки `newtype`

Что если теперь мы хотим определить моноид для логического И?
Мы уже реализовали экземпляр для `Bool` и не можем определить второй экземпляр.
Проблема в том, что математический моноид определяется не только типом, но и операцией `mappend`.
Для одного типа может быть несколько операций, образующих моноид.
При этом класс типов может зависеть только от выбора типа.
Чтобы решить эту проблему, в Haskell обычно используются обёртки над типами.
Например, для моноидов логического И и логического ИЛИ                мы можем объявить два новых типа-обёртки для типа `Bool`.
Тип `Any` будет использоваться для логического ИЛИ, а тип `All` для логического И.
Теперь, когда у нас есть два различных типа, мы можем определить для них два различных экземпляра.
Для типа `Any` мы используем обёрнутый `False` в качестве нейтрального элемента и операцию логического ИЛИ.
Для типа `All` мы используем обёрнутый `True` и операцию логического И.
Для создания типов-обёрток стоит использовать ключевое слово `newtype` вместо `data`.
В отличие от `data`, `newtype` работает только с обёртками. То есть с типами, у которых один конструктор с одним полем.
Преимущество `newtype` в том, что новый тип будет иметь ровно такое же представление, как и оборачиваемый тип.
Это значит, что использование `newtype` не влияет на производительность программы. При этом, поскольку обёртка — это не синоним, использование `newtype` может предотвратить больше ошибок на этапе компиляции.
Вернёмся к моноидам.
Чтобы определить моноид для суммы и произведения чисел, создадим обёртки `Sum` и `Product`.
Заметим, что эти обёртки полиморфны и могут обернуть любой тип.
Для `Sum a` нейтральный элемент будет ноль, а `mappend` — операция сложения.
Для `Product a` — единица и операция умножения.
Чтобы определить моноид для функций из `a` в `a`, используем новую обёртку.
Функции, у которых совпадают типы аргумента и результата, называются эндоморфизмы.
Назовём нашу обёртку `Endo`.
Нейтральным элементом для `Endo` будет тождественная функция.
А `mappend` будет представлен композицией функций.
Законы моноида для `Endo` принимают следующий вид и, очевидно, выполняются.
На практике моноиды встречаются везде:
логи и прочая отладочная информация образуют моноид; конфигурации и настройки образуют моноид;
изображения образуют моноид с операцией наложения;
объекты векторной графики (например, конверты) часто являются моноидами;
моноиды используются в реализациях эффективных структур.


В качестве упражнения, реализуйте экземпляр класса типов `Monoid` для ассоциативных списков.
Операция сложения для ассоциативных списков объединяет их в один.
Для совпадающих ключей соответствующие значения складываются при помощи операции `mappend`.

### Моноидная свёртка списка

Окей, моноиды — это потрясающе, но зачем они нам нужны в прграммировании?
В конце концов, мы можем просто использовать операции сложения и умножения!
Моноиды позволяют нам абстрагироваться от расстановки скобок.
И это важно по нескольким причинам.
Во-первых, мы можем реализовывать алгоритмы по принципу «Разделяй и властвуй».
Во-вторых, мы можем вычислять каждый кусок данных параллельно.
И, наконец, мы можем решить, как расставить скобки, потом — результат от этого не изменится!
Для свёртки списка моноидных значений существует функция `mconcat`.
Эта функция использует `mappend` в качестве шага свёртки и `mempty` в качестве начального значения.
Мы можем реализовать многие известные нам функции, используя `mconcat`.
Чтобы конкатенировать список списков, мы можем использовать тот факт, что список — моноид.
Чтобы вычислить сумму списка мы можем обернуть каждое значение в списке в конструктор `Sum`, использовать `mconcat` и развернуть получившийся результат.
Аналогичным образом мы можем реализовать `product`, `and` и `or`.

### Возвращение к мотивирующему примеру

Итак, какое отношение моноиды имеют к силовому полю?
Поле образует моноид!
Мы можем представить поле функцией, принимающей на вход точку в пространстве, и возвращающую вектор — силу поля в этой точке.
Давайте объявим тип-обёртку `Field` для такой функции.
Мы можем складывать поля, складывая значения сил в каждой точке.
Такая операция будет, как и сложение векторов, ассоциативна.
Нейтральным значением относительно этой операции будет поле с нулевой силой во всех точках пространства.
Определим реализацию экземпляра класса типов `Monoid` для поля.
Для нейтрального элемента используем функцию, всегда возвращающую нулевой вектор.
Для операции сложения используем лямбда-функцию, принимающую координаты точки и вычисляющую сумму сил двух полей.
Теперь, когда у нас в программе есть сущность `Field`, мы можем использовать его для определения силы в любой точке.
Для этого нам понадобится определять поле для одного тела.
Напряжённость поля определяется произведением гравитационной постоянной на массу тела,
поделённого на квадрат расстояния до заданной точки.
Вектор будет направлен к заданной точке.
Поле всей системы мы можем получить, вычислив поле каждого тела, и применив функцию `mconcat`.
Теперь мы можем отобразить поле, рассчитав значения в интересующих нас точках.

Чтобы не терять объекты моделирования из виду, давайте определять масштабирование автоматически.
А именно, давайте реализуем функцию `bounds`, вычисляющую границы наименьшего прямоугольника, в котором находятся все объекты.
Определите экземпляр класса `Monoid` для типа `Bounds` и используйте его при реализации функции.

Ссылка на проект с визуализацией силового поля находится в описании под видео.
Успехов!

