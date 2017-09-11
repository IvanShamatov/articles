Шок! Не вся правда о декораторах, то, что от нас скрывали! Без регистрации и СМС.

> Гем `draper` не декоратор! И `cells` — тоже.

— Всё, народ, расходимся. Всем спасибо, все свободны. Ведь это и так все знают. Правда?

Первая статья [про интеракторы](https://mkdev.me/posts/paru-slov-pro-interaktory-v-rails) вызвала живой интерес у публики. Так что, держите продолжение, теперь уже про декораторы!

### К делу

Сразу договоримся, я придерживаюсь GOF определения паттерна "Декоратор". Именно такое определение нам дает [википедия](https://ru.wikipedia.org/wiki/%D0%94%D0%B5%D0%BA%D0%BE%D1%80%D0%B0%D1%82%D0%BE%D1%80_(%D1%88%D0%B0%D0%B1%D0%BB%D0%BE%D0%BD_%D0%BF%D1%80%D0%BE%D0%B5%D0%BA%D1%82%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D1%8F)).

> Декоратор (англ. Decorator) — структурный шаблон проектирования, предназначенный для динамического подключения дополнительного поведения к объекту. Шаблон Декоратор предоставляет гибкую альтернативу практике создания подклассов с целью расширения функциональности.
>
> Добавляемая функциональность реализуется в небольших объектах. Преимущество состоит в возможности динамически добавлять эту функциональность до или после основной функциональности объекта

#### Дано:

Как всегда, гораздо понятнее становится на примере. Продолжим разбираться с формированием заказа, а именно сделаем расчет стоимости доставки.

Как бы это странным не казалось, но стоимость доставки зависит от объема, веса, дальности доставки, от того будет ли это доставка до дверей или только до городского терминала, нужно ли обернуть заказ в пупырки или обрешетку, ну и конечно от того, какой службой доставки пользоваться. Короче, параметров тьма! Просто взгляни на сайт любой транспортной компании и увидишь калькулятор на 100500 параметров.

Изначально у нас будет следующий набор.

У нас есть класс `Order` с аттрибутами: вес, ширина, высота, длина и город, куда надо доставить этот заказ.

```ruby
class Order
  attr_accessor :weight, :width, :height, :length, :city
 
  def initialize(weight, width, height, length, city)
       @weight = weight
       @width = width
       @height = height
       @length = length
       @city = city
  end
 
  def volume # не громкость
       width*height*length
  end
end
```

Так же у нас есть внешний сервис для рассчета стоимости доставки до города (всегда возвращает 100).

```ruby
class DlhApi # служба доставки с желто-красным логотипом
  def self.calculate(city)
    100
  end
end
```

Ну и сам класс доставки.

```ruby
class Delivery
  WEIGHT_COEFFICIENT = 1
  VOLUME_COEFFICIENT = 1

  def initialize(order)
    @order = order
  end

  def cost
    @order.weight * WEIGHT_COEFFICIENT + @order.volume * VOLUME_COEFFICIENT
  end

  def city
    @order.city
  end
end

order = Order.new(10, 5, 1, 1, "Moscow")
Delivery.new(order).cost #=> 15
```

Тааак, с этим разобрались! Есть заказ. Есть доставка, в которую передаем заказ. И есть метод для рассчета стоимости доставки! Вроде всё пока просто.

#### Решение:

1. Решение в лоб. Стоит заметить, что не всегда следует стремиться получить красивые абстракции. Иногда решение в лоб — это то, что нужно.

  А решение состоит в том, что мы добавим все наши "модификаторы стоимости" прямо в класс `Delivery` в качестве опций. И там уже в методе `cost` перебором по опциям будем изменять конечную стоимость. В целом, для двух-трех модификаторов, почему бы и нет? Но у нас их очень много. Одних служб доставки может быть 10.

2. Решение наследованием. Мы можем создать подкласс `OneDayDelivery`, `DlhDelivery`, `UpToDoorDelivery`, которые и будем создавать и не будет никаких if-ов. Но следуя такой логике, у нас обязан появиться подкласс `OneDayDlhUpToDoorBubbleWrappedDelivery`. И именно эту проблему и призван решить паттер декоратор.

3. Решение декораторами. И вот как это будет.

У декораторов есть такая традиция — каждый новый год они собираются в бане и оборачивают друг друга (if you know what I mean). Если серьезно, то декоратор — это же структурный паттерн, подвид паттерна composite. Мы можем обернуть один декоратор другим, а потом третьим, и тем самым, каждый новый уровень будет добавлять новую функциональность. Вот простейший пример:

```ruby
class DeliveryDecorator
  def initialize(obj)
    @obj = obj
  end

  def cost
    @obj.cost + 10
  end
end

DeliveryDecorator.new( Delivery.new( order )).cost #=> 25
DeliveryDecorator.new( DeliveryDecorator.new( Delivery.new( order ))).cost #=> 35
DeliveryDecorator.new( DeliveryDecorator.new( DeliveryDecorator.new( Delivery.new( order )))).cost #=> 45
```

При инициализации мы сохраняем заказ, который декорируем, а потом обращаемся к его методу `cost`, внутри нашей реализации метода `cost`.
Из этого можно сделать такой вывод: все декораторы одного объекта должны обладать одним интерфейсом (в нашем случае это метод `cost`).


#### Оффтопик

> **Вопрос читателя**: Самый банальный вопрос - организация кода.... Где и что должно
> находиться....

> **Ответ экспертной комиссии**: Эта должена находится сдеся вот!
>
> А в качестве аргумента к ответу экспертной комиссии, прикладываю нотариально заверенный скриншот.

![](https://habrastorage.org/files/cb3/926/a13/cb3926a13988449c95ebd789bfb92edb.png)

Для меня такая структура очевидна и понятна.

* Во-первых, декораторы лежат в папке декораторов.
* Во-вторых, такая структура позволяет мне утилизировать рельсовые namespace'ы. Если я положил в папочку `delivery`, то и неймспейс будет `Delivery` — круто же!

<p class="notice clearfix">
<a href="https://mkdev.me/mentors/IvanShamatov" class="button pull-right" onClick="ga('send', 'event', 'article button', 'learn more');">Записаться</a>
Автор этой статьи может стать <br />твоим персональным наставником
</p>

#### Поехали дальше

Я уверен, ты уже подсмотрел реализацию пупырчатого декоратора на картинке. Поэтому приведу примеры реализации двух других декораторов.

```ruby
class Delivery::Dlh
  def initialize(obj)
    @obj = obj
  end

  def cost
    @obj.cost + SomeDhlApi.calculate(@obj.city)
  end
end

class Delivery::UpToDoor
  STANDARD_TIP = 10
 
  def initialize(obj)
    @obj = obj
  end

  def cost
    @obj.cost + STANDARD_TIP
  end
end

order = Order.new(10, 5, "Moscow")
# Просто доставка
Delivery.new( order ).cost #=> 15

# Доставка курьерской службой
Delivery::Dlh.new( Delivery.new(order)).cost #=> 115

# Доставка курьерской службой, посылка обернута в пупырку
Delivery::BubbleWrapped.new(
  Delivery::Dlh.new(
    Delivery.new( order )
  )
).cost #=> 125

# Доставка курьерской службой до двери, посылка обернута в пупырку
Delivery::UpToDoor.new(
  Delivery::BubbleWrapped.new(
    Delivery::Dlh.new(
      Delivery.new( order )
    )
  )
).cost #=> 140

```

Разве это не прекрасно? Мы взяли и всю эту *сложную* логику разнесли на логические единицы, положили в разные файлы, делаем такие красивые вызовы. Можно ли желать лучшего?

#### Можно!

ДЗ — разобраться как же это так работает, вот вам пример.

```ruby
module Delivery::BubbleWrapped
  def cost
    super + 10
  end
end

module Delivery::UpToDoor
  def cost
    super + 15
  end
end

module Delivery::Dhl
  def cost
    super + SomeDhlApi.calculate(city)
  end
end


order = Order.new(10, 5, 1, 1, "Moscow")
delivery = Delivery.new(order)
delivery.cost #=> 15

delivery.extend(Delivery::Dhl)
delivery.cost #=> 115

delivery.extend(Delivery::BubbleWrapped)
delivery.cost #=> 125

delivery.extend(Delivery::UpToDoor)
delivery.cost #=> 140
```

#### Источники

— Не для того чтобы сослаться, откуда я брал материал. А для самостоятельного расширенного изучения вопроса. Поверьте, это того стоит.

1. [Design Patterns in Ruby. Глава про декораторы](http://www.informit.com/store/design-patterns-in-ruby-9780321490452)
2. [Tidy Views and Beyond with Decorators](https://robots.thoughtbot.com/tidy-views-and-beyond-with-decorators)
3. [Evaluating Alternative Decorator Implementations In Ruby](https://robots.thoughtbot.com/evaluating-alternative-decorator-implementations-in)
4. [Decorators, Presenters, Delegators and Rails](https://robertomurray.co.uk/blog/2014/decorators-presenters-delegators-rails/)
5. [Stackoverflow. Decorators vs presenters](http://stackoverflow.com/questions/7860301/ruby-on-rails-patterns-decorator-vs-presenter)

#### P.S. Так, а что там с draper'ом не так?

Да всё с ним так. Гем `draper` предоставляет возможность создать полноценный декоратор.
Для этого следует добавить внутрь `Draper::Decorator` вызов `delagate_all`, чтобы наш вызов передался по цепочке, если в последнем слое не реализован нужный нам метод.

А, и еще одно ограничение на данный момент, драпер не умеет оборачивать декоратор в еще один такой же декоратор (см последние 2 строчки примера), та же проблема что и с модульным подходом.

```ruby
require 'draper'
class DeliveryDecorator < Draper::Decorator
  delegate_all
  def cost
    object.cost + 10
  end
end

DeliveryDecorator.new(Delivery.new(order)).cost #=> 25
DeliveryDecorator.new(DeliveryDecorator.new(Delivery.new(order))).cost #=> 25
...
```

Причина, по которой я осмелился на столь громкие заявления, что `draper` не декоратор так это как его используют. А используют его как мостик между моделью и отображением. Они так и позиционируются — Decorators/View-Models for Rails Applications.

И на самом то деле за это отвечает паттерн `Presenter`. Но о нём в следующий раз. Stay tuned.
