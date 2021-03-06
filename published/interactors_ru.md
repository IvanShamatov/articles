Про что? — Про интеракторы. Ой да не суть. Скажите лучше, вы к какому клану относитесь: остроконечных или тупоголовых? Перефразирую: вы за тонкие модели или за тонкие контроллеры? Как же это, пожалуй, подло с моей стороны задавать заведомо неверный вопрос.

Обитая в мире Rails, вам наверняка приходилось слышать подобные высказывания.

> — Вся логика должна размещаться в модели, контроллер предоставляет только простой http-уровень. Контроллер принял данные, а дальше задача модели их правильно обработать и сохранить в базу данных

Им же в ответ сообщают:

> — Контроллер на то так и называется, чтобы контролировать процесс. Модель должна знать лишь о том, что как правильно замапить данные на таблицу в базе.

И тут приезжает [@dhh](http://twitter.com/dhh) на своем спорткаре, финишировав в очередных гонках и говорит:

> — Вы конечно как хотите, но я тут в каждую директорию вашего проекта добавил папочку `concerns`. Вы можете наделать себе модулей, в них запихнуть все методы и вынести логику в цепочку методов.

> — Ну нет, Дэвид, это не круто, Rails уже не торт. Rails умирает, решений нет.

На самом же деле решение есть, и даже не одно, и стары они как мир. Как дядюшка Фаулер завещал. Называется Интерактор (Interactor), а еще Command Pattern, или Operation — это всё названия одного и того же подхода свойственного для DDD (Domain driven design). Да-да, куча иностранных слов, призванных ввести читателя в ступор. Обо всём по порядку.

#### Domain Driven Design

Что же такое DDD? Это не очередная аббревиатура о подходе к тестированию (TDD/BDD), нет.

"DDD — это набор принципов и схем, помогающих разработчикам создавать изящные системы объектов. При правильном применении оно приводит к созданию программных абстракций, которые называются моделями предметных областей. В эти модели входит сложная бизнес-логика, устраняющая промежуток между реальными условиями области применения продукта и кодом" гласит википедия и дает ссылку на [msdn](https://msdn.microsoft.com/ru-ru/magazine/dd419654.aspx).

Итак, господа, мы — создатели ***изящных*** систем объектов, создаем абстракции  :) Именно в отсутствии достаточных уровней абстракций чаще всего и упрекают Rails. Хотя казалось бы? Возьми да сделай! В конце-то концов, ты программист или настройщик фреймворка?

#### Interactor

А это что такое? Вот пару словосочетаний, которые описывают назначение интеракторов: business case object, use case object.

Интерактор — сервисный объект, который создает абстракцию над небольшой областью знаний (модель предметной области) и инкапсулирет бизнес-логику приложения.

#### TL;DR;

Давайте наконец перейдем от слов к делу и рассмотрим использование интеракторов на примере.

> **Задача**: Реализовать размещение заказа на сайте онлайн-магазине.

> **Размышления**: Создание заказа — это довольно широкое понятие, оно может включать в себя массу логики по формированию записи в базе данных, проведению платежа, смене статуса, уведомлении клиента, формировании задач на обратный звонок и уточнение заказа.

#### Шаг 1. Инкапсулируем логику

Да это все так, но для того чтобы нам начать работать, нам вовсе не обязательно это всё знать — мы инкапсулируем эту область знаний одним интерактором.

В своем примере я буду использовать gem interactor, но в реальности вы можете написать свой сервисный объект. Для установки гема добавим его в Gemfile `gem 'interactor'` и выполним `$ bundle install`.

Предположим, что код нашего контроллера, где мы создаем заказ будет выглядеть так:

```ruby
#  /app/controllers/orders_controller.rb
1  class OrdersController < ApplicationController
2    def create
3      result = PlaceOrder.call(
4        params: order_params,
5        user: current_user
6      )
7      if result.success?
8        redirect_to result.order, notice: "Order created"
9      else
10       redirect_to cart_path, status: :internal_server_error
11     end
12   end
13
14   private def order_params
15     params.require(:order).permit(...)
16   end
17 end
```

Как мы видим, у нас приходит некий запрос на сервер, который обрабатывается в `OrdersController` экшеном `create`. В строке 3 мы создаем сервисный объект PlaceOrder и вызываем его метод call с двумя параметрами. Кстати говоря, этих параметров может быть сколь угодно много. Все они формируют **контекст** — то, с чем будет работать интерактор. Например, в данном случае для того чтобы создать заказ, нам необходима информация о выбранных элементах и о заказчике. ВСЁ!

Далее проиходит какая-то магия внутри, и если всё прошло успешно, то контроллер возвращает успех/редиректит на страницу заказа. Если нет, ну что ж, отправим пользователя на страницу корзины.

<p class="notice clearfix">
<a href="https://mkdev.me/mentors/IvanShamatov" class="button pull-right" onClick="ga('send', 'event', 'article button', 'learn more');">Записаться</a>
Ты можешь сделать автора статьи<br />своим персональным наставником
</p>

#### Шаг 2. Разобьем по уровням ответственности

Зачастую, прямо здесь работа и заканчивается. Предположим, что размещение заказа — это в действительности просто создание заказа в базе данных.

```ruby
#  app/interactors/place_order.rb
1  class PlaceOrder
2    include Interactor
3
4    def call
5      order = Order.new(context.params)
6      order.user = context.user
7      if order.save
8        context.order = order
9      else
10       context.fail!
11     end
12   end
13 end

```
И если мы решили проблему с помощью интерактора — это уже успех. Мы как минимум выделили некую логику, которая не должна находиться в контроллере, потому что не имеет отношения к http-уровню. Она не должна быть и в модели, потому что не является оберткой для создания правильного запроса к базе данных (а как мы помним, это и есть истинная задача ActiveRecord паттерна). Эта логика является описанием бизнес-кейса. В нашем случае single purpose business case.

> **Внимание**: Практически всегда бизнес-кейс можно описать действием — "зарегать пользователя", "разместить заказ", "запустить ракету", "сделать хорошо". Это отличный способ именовать свой интерактор. Сразу понятно, что в нём происходит.

В строке 5 мы видим обращение к `context`. Именно в контекст передаются те параметры, что мы указали при вызове PlaceOrder.call. А так же в строке 8 мы сохраняем значение в контекст для того, чтобы получить это значение потом в контроллере.

В строке 10 обратим внимание на вызов `.fail!` — таким образом мы сигнализируем, что интерактор не выполнился и дальнейшее выполнение будет прекращено. Нужно ли говорить, что как раз таки здесь, именно в интеракторе и нужно имплементировать валидацию данных?

Понимаю, это так сложно по началу принять, что встроенные в rails валидации — это не самое удачное решение. Но приходилось ли вам писать валидации, которые должны срабатывать в одном сценарии и не срабатывать в другом. Или бывало-ли, когда в разных сценариях на один и тот же колбек нужно запустить разные методы? Да им просто там не место. Эти методы — часть бизнес-кейса, а не модели. И они должны размещаться в интеракторе.

#### Шаг 2. Часть 2

Вернёмся к нашим баранам: ну хорошо. А если всё же размещение заказа — это гораздо более широкое понятие и нужно сделать 1-3-7-100 шагов?

Другими словами, что если мы разобьем сценарий "Разместить заказ" на:
 
1. Создать заказ
2. Произвести платеж
3. Отправить уведомление
4. Открыть бутылку шампанского

Да не проблема. Мы делим логику на интеракторы, которые будем вызывать внутри других интеракторов.

![](http://jonfriesen.ca/content/images/2013/Oct/deeper.jpg)

В данной конкретной имплементации `gem interactor` умеет делать для многошаговых интеракторов органайзеры. На примере сразу станет всё ясно.

```ruby
# app/interactors/place_order.rb
1  class PlaceOrder
2    include Interactor::Organzier
3   
4    organize CreateOrder, MakePayment, SendNotification, OpenBottleOfChampagne
5  end
```

Самая вкуснота представлена на 4й строке. Это просто список интеракторов, которые будут запущены один за другим.

При этом у них будет один общий контекст (`context`), и если вам в MakePayment нужна информация о заказе, просто сохраните эту информацию в CreateOrder в контекст. А информация о пользователе будет доступна в SendNotification интеракторе.

Если же в ходе выполнения цепочки интеракторов где-то происходит `context.fail!`, то выполнение прекращается и следующие шаги выполнены не будут. Делать выдуманную имплементацию этих внутренних шагов считаю уже излишним. Вы и сами прекрасно можете представить, что в них может происходить. Замечательно!


#### Тестирование интеракторов

Вот мы и добрались до самого интересного. Тестирование интеракторов — это божественно приятно. Я вообще понял смысл тестирования, только когда познал эту концепцию.

* Во-первых, нам не нужно тестировать контроллер (интеграционные тесты идут в пекло!).
* Во-вторых, нам не нужно тестировать модель и её какие-то магические колбеки и обработки (вы же не будете тестировать метапрограмминг рельсы, правда?).
* В-третьих, тестирование интерактора подобно тестированию чистого руби объекта. Ты задал данные на входе и проверил успех и контекст на выходе. Изумительно до дрожи в пальчиках рук.

```ruby
require 'rails_helper'
describe PlaceOrder do
  before do
    @user = FactoryGirl.create(:user)
    @params = {order_items_attributes: {item_id: 1, quantity: 2},...}
  end
 
  it ".call should create order and make payment" do
    interactor = PlaceOrder.call(user: @user, params: @params)
    expect(interactor).to be_a_success
    expect(interactor.order).to eq(Order.last)
    ...
  end
end
```

Вот и всё, ребята, что я хотел рассказать сегодня. Используйте сервисные объекты, интеракторы и не стесняйтесь юзать не родные Rails абстракции. И да пребудет с вами сила!

#### Источники

1. [Where's your business logic?](http://collectiveidea.com/blog/archives/2012/06/28/wheres-your-business-logic/)
2. [Trailblazer. A new architecture for Rails](https://leanpub.com/trailblazer)
3. [Wiki: Command Pattern](https://en.wikipedia.org/wiki/Command_pattern)
4. [Wiki: Domain Driven Design](https://en.wikipedia.org/wiki/Domain-driven_design)
