Part 3: Образцы повсюду
===================================

В предыдущих двух частях мы достаточно подробно рассмотрели, что происходит со значениями,
которые принадлежат к `case`-классам, при сопоставлении с образцом. Мы узнали как определить
экстракторы, которые позволят извлекать данные из значений любых типов всеми возможными способами.

Пришло время посмотреть в каких местах мы можем пользоваться *образцами* (pattern). 
Пока нам встретился лишь один способ. Посмотрим и на другие!

Сопотставление с образцом
----------------------------------------

Образцы могут встретиться при обычном сопоставлении с образцом. Этот способ должен
быть Вам знаком из курса на Coursera или из предыдущих примеров этой серии статей. 
У нас есть некоторое выражение `e`, за которым следует ключевое слово `match` и 
блок, содержащий набор альтернатив. Каждая альтернатива состоит из ключевого слова `case`.
После него пишется образец, возможно с охранным выражением (*guard*) с левой части от `=>`.
В правой части находится лок кода, который выполняется в том случае, если сопоставление
с образцом произошло успешно.

Приведём простой пример с двумя альтернативами и одним охранным выражением:

~~~
case class Player(name: String, score: Int)

def printMessage(player: Player) = player match {
  case Player(_, score) if score > 100000 => println("Get a job, dude!")
  case Player(name, _) => println("Hey " + name + ", nice to see you again!")
}
~~~

Метод `printMessage` возвращает значение типа `Unit`. Он определён только лишь для 
выполнения побочных эффектов, то есть печати сообщения на экран.  Важно понимать,
что нет необходимости пользоваться сопоставлением образцом точно так же как мы бы пользовались
`switch`-выражениями в Java. Сопоставление с образцом может возвращать значения, а не просто
выполнять код в правой части `case`-альтернатив. 

Воспользовавшись этим, мы можем разделить две части нашего кода, отвечающие за разные задачи.
Это облегчит тестирование. Перепишем этот пример следующим образом:

~~~
def message(player: Player) = player match {
  case Player(_, score) if score > 100000 => "Get a job, dude!"
  case Player(name, _) => "Hey " + name + ", nice to see you again!"
}

def printMessage(player: Player) = println(message(player))
~~~

Теперь у нас есть отдельный метод, возвращающий строку. По сути это чистая функция,
возвращающая результат сопоставления с образцом. Мы можем присвоить результат переменной
или сохранить его как значение.

Образцы в определениях значений
----------------------------------------------

Также мы можем воспользоваться образцами слева от знака `=` при определении значений 
(а также изменяемых знаений, но мы будем придерживаться функциональному стилю программирования, 
поэтому они будут встречатся крайне редко). Предположим у нас есть метод, что возвращает 
текущего игрока. Мы воспользуемся определением-заглушкой, которое всегда возвращает
некоторое значение в качестве игрока:

~~~
def currentPlayer(): Player = Player("Daniel", 3500)
~~~

Обычно мы определяем значение так:

~~~
val player = currentPlayer()
doSomethingWithTheName(player.name)
~~~

Возможно через Python Вы знакомы с возможностью называемой распаковкой последовательности (sequence unpacking).
Так мы можем использовать люой образец слева от определения переменной. В Scala эта возможность также доступна.
Так мы можем одновременно определить нового игрока и извлечь необходимые данные:

~~~
val Player(name, _) = currentPlayer()
doSomethingWithTheName(name)
~~~

Мы можем пользоваться такой записью с любым образцом, но было бы хорошо если бы наш образец
всегда успешно проходил сопоставление со значением. Иначе случится ошибка времени выполнения.
К примеру следующий код таит угрозу. Метод `scores` возвращает список результатов. 
В нашем примере метод просто возвращает пустой список для иллюстрации проблемы:

~~~
def scores: List[Int] = List()
val best :: rest = scores
println("The score of our champion is " + best)
~~~

Вот так так, мы получили ошибку `MatchError`. Похоже наша игра оборвалась, не дойдя до результатов.

Лучше всего пользоваться образцами при определении значений, тип которых нам известен на этапе 
компиляции. Такэе при работе с кортежами образцы могут сильно улучшить наглядность кода. 
Предположим у нас есть функция, возвращающая пару из имени игрока и его результата в игре,
без класса `Player`, которым мы пользовались до этого:

~~~
def gameResult(): (String, Int) = ("Daniel", 3500)
~~~

Доступ к полям кортежа кажется очень неуклюжим:

~~~
val result = gameResult()
println(result._1 + ": " + result._2)
~~~

Извлечение данных из кортежа при определении значения надёжно, поскольку мы знаем, что 
значение имеет тип `Tuple2`:

~~~
val (name, score) = gameResult()
println(name + ": " + score)
~~~

Так гораздо лучше, не правда ли?

Образцы в `for`-генераторах
-----------------------------------------

Также образцы играют важную роль в `for`-генераторах. Мы можем объявлять переменные
в `for`-генераторах, поэтому всё что мы только что узнали о применении образцов слева
от оператора `=` при определении переменных, также работает и в `for`-генераторах. 
Так если у нас есть набор игроков и мы хотим узнать кто же из них достоин зала славы,
а в нашей игре это просто игроки, набравшие очков больше чем некоторый заданный порог.
С помощью `for`-генераторов мы можем сделать это в очень наглядном виде:

~~~
def gameResults(): Seq[(String, Int)] =
  ("Daniel", 3500) :: ("Melissa", 13000) :: ("John", 7000) :: Nil

def hallOfFame = for {
  result <- gameResults()
  (name, score) = result
  if (score > 5000)
} yield name
~~~

В итоге мы получим список `List("Melissa", "John")`, поскольку первый игрок не проходит
условие, объявленное в охранном выражении.

Мы можем улучшить это выражение, ведь в `for`-генераторах правая часть генератора также
является образцом. Поэтому мы можем избавиться от промежуточной переменной `result`:

~~~
def hallOfFame = for {
  (name, score) <- gameResults()
  if (score > 5000)
} yield name
~~~

Сопоставление с образцом `(name, score)` всегда будет проходить успешно, поэтому
если бы не охранное выражение `if (score > 5000)`, итоговое значение было бы 
эквивалентно простому извлечению имени из всех игроков без фильтрации.  

Важно понимать, что образцы в правой части генераторов могут использоваться 
для фильтрации. Если значение не проходит сопоставление с образцом в левой 
части генератора, то оно отбрасывается. 

Предположим у нас есть список списков и мы хотим вернуть размеры
всех не пустых списков. Для этого нам необходимо отфильтровать все пустые списки
и вернуть размеры всех тех, что останутся. Вот решение:

~~~
val lists = List(1, 2, 3) :: List.empty :: List(5, 3) :: Nil

for {
  list @ head :: _ <- lists
} yield list.size
~~~

Образец в левой части генератора подходит только для не пустых списков. 
Сравнение с образцом для пустого списка не приведёт к ошибке `MatchError`, 
вместо этого пустой список будет удалён из результирующей последовательности.
Поэтому мы получим в результате список `List(3, 2)`.

Образцы очень хорошо сочетаются с `for`-генераторами. Со временем они будут
встречаться в вашем коде всё чаще и чаще. 

Анонимные функции
--------------------------------------------

Наконец, образцы могут быть использованы при определении анонимных функций. 
Вы не могли обойти стороной эту возможность, если пользовались 
`catch`-блоком для обработки исключений в Scala. Но эта тема заслуживает отдельной
статьи. Она слишком велика, но мы вернёмся к ней уже в следующей статье.

*Обновление: Исправлена ошибка в результате выражения `hallOfFame`. Спасибо Rajiv за правку.*