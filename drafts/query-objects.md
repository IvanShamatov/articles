# Query-object
Уже больше года прошло с момента публикации моих статей про базовые сервисные объекты. Надо бы продолжить! 

В последнем номере [Ruby weekly](http://rubyweekly.com/issues/365) я наткнулся на классную статью про [паттерн команда](https://drivy.engineering/code_simplicity_command_pattern). В самом начале там написано: 
> Паттерн-команда иногда называют сервисным объектом, operation'ом, action'ом и наверняка еще каким-либо именем, о котором я не знаю. Как ни назови, суть подобного паттерна достаточно проста: взять некое бизнес-действие и обернуть в объект с простейшим интерфейсом.

От себя добавлю, что статья про [интеракторы](https://mkdev.me/posts/paru-slov-pro-interaktory-v-rails) — про то же самое. В Trailblazer-e их называют operation'ами, а в dry-экосистеме — ближе всего к такому определению dry-transaction.

В телеграм-чате я как-то упомянул про "сервисные объекты" и один товарищ зацепился за это имя и стал расспрашивать, что же такое эти самые "сервисные объекты". Закончилось тем, что мы сошлись на названии PORO (Plain Old Ruby Object). И я с ним полностью согласен. Это всего-лишь обычный объект, никакой в этом магии нет. А сервисными я их называю потому, что они ("serve") обслуживают и выполняют какую-то небольшую задачу и прячут внутри себя логику по её свершению.

Помимо интеракторов есть и другие сервисы. Их полно и это лишь зависит от твоей фантазии. Например:

* интеракторы (interactors) ([мы уже их знаем](https://mkdev.me/posts/paru-slov-pro-interaktory-v-rails))
* декораторы (decorators) ([и про них говорили](https://mkdev.me/posts/ne-vsya-pravda-o-dekoratorah))
* презентеры (presenters)
* сериалайзеры (serializers)
* политики (policies)
* query-объекты (queries)
* и т.д.

Помимо этого я в своих проектах выделяю и другие сервисы, например: отчеты (reports), компоненты (cells от trailblazer), метрики (metrics).

Ну и конечно, есть уже настолько привычные нашему взгляду сервисы, которыми мы постоянно пользуемся, но просто не думали о них, как о севрисных-объектах в rails:

* uploaders (carrierwave)
* validators (active-model)
* jobs/workers (activejob/sidekiq)
* subscribers (wisper)
* enums (classy-enum)

И так далее. Понятно, что список можно продолжать, но на сегодня есть конкретная тема — это Query-объекты.

## Query-object
Что же такое query-object? Да это просто объект, который позволяет создать большой и сложный sql-запрос (query) с помощью используемого ORM. И говоря про rails, мы подразумеваем ActiveRecord.


### query-object
Разберём, как мы можем воспользоваться таким классом на примере интернет-магазина и каталог товаров в нём. Сначала, как это могло бы выглядеть в контроллере, а потом с использованием сервисного объекта. С каждым шагом

```ruby
class CatalogController < ApplicationController
  def index
    @products = Product.all
  end
end
``` 

Вот так будет выглядеть выборка всего списка товаров из базы. Но нам, конечно, нужно не все, а лишь те товары, что для этой страницы (вспоминаем про пагинацию).

```ruby
def index
  page_number = params[:page] || 0
  @products = Product.page(page_number)
end
```

Вспоминаем, что нам пользователь может сортировать товары по цене.

```ruby
def index
  price_sort_direction = params[:price_sort_direction].to_sym || :desc
  page_number = params[:page] || 0
  @products = Product.order(price: price_sort_direction).page(page_number)
end
```

Но кстати, ведь сортировать можно и не только по цене.

```ruby
def index
  sort_direction = params[:sort_direction].to_sym || :desc
  sort_type = params[:sort_type].to_sym || :price
  page_number = params[:page] || 0
  @products = Product.order(sort_type => sort_direction).page(page_number)
end
```

Конечно же, нужно выбирать товары определенной категории.

```ruby
def index
  @products = Product.all
   
  category_id = params[:category_id]
  @products = @products.where(category_id: category_id) if category_id
  
  # ... Здесь я спрятал весь предыдущий код, который, конечно, никуда не делся ...
end
```

Добавим же теперь парочку критериев для фильтрации товаров

```ruby
def index
  @products = Product.all
  
  property_ids = params[:properties]
  if properties
    @products = @products.joins(:product_properties)
                         .where(property_id: property_ids) 
  end 

  # ... Здесь я спрятал весь предыдущий код, который, конечно, никуда не делся ...
end
```

А может добавим еще ценовой диапазон?

```ruby
def index
  @products = Product.all
  
  from_price = params[:from_price]
  @products = @products.where('price > ?', from_price) if from_price
  
  to_price = params[:to_price]
  @products = @products.where('price < ?', to_price) if to_price
  
  # ... Здесь я спрятал весь предыдущий код, который, конечно, никуда не делся ...
end
```

Да и куда же без самого обычного поиска? Возможность вбить пару букв и найти похожее слово в названии товара.

```ruby
def index
  @products = Product.all
  
  search = params[:search]
  @products = @products.where("title ILIKE '%?%'", search) if search
  
  # ... Здесь я спрятал весь предыдущий код, который, конечно, никуда не делся ...
end
```

Вот так. Всего пару иттераций и наш простейший фильтрованный каталог готов. Посмотрим, что же вышло в итоге:

```ruby
def index
  @products = Product.all
  
  search = params[:search]
  @products = @products.where("title ILIKE '%?%'", search) if search
  
  from_price = params[:from_price]
  @products = @products.where('price > ?', from_price) if from_price
  
  to_price = params[:to_price]
  @products = @products.where('price < ?', to_price) if to_price

  property_ids = params[:properties]
  if properties
    @products = @products.joins(:product_properties)
                         .where(property_id: property_ids) 
  end 
  
  category_id = params[:category_id]
  @products = @products.where(category_id: category_id) if category_id
    
  sort_direction = params[:sort_direction].to_sym || :desc
  sort_type = params[:sort_type].to_sym || :price
  page_number = params[:page] || 0
  @products = @products.order(sort_type => sort_direction).page(page_number)
end
```

Достаточно аккуратно, если учесть, что тут вовсе не все сценарии описаны, а самые базовые. Да и к тому же, у меня есть опыт написания подобных штук, а чаще встречаются куда более громоздкие и странные варианты.

Если же взглянуть на весь экшен, то по сути, происходит просто формирование большого запроса к базе данных.

### Query-object
Ну что же, давай поместим всё вот это в отдельный класс, который будем вызывать. Тем самым спрячем логику за "ёмким" названием `FindProducts`. Вот так будет выглядеть наш контроллер:

```ruby
class CatalogController < ApplicationController
  def index
    @products = FindProducts.new(Product.all).call(permitted_params)
  end
  
  def permitted_params
    params.permit(:search, :from_price, :to_price,
                  :properties, :category_id,
                  :sort_direction, :sort_type, :page)
  end
end
```

А вот как может выглядеть в таком случае наш объект:

```ruby
# app/queries/find_products.rb
class FindProducts
  attr_accessor :initial_scope

  def initialize(initial_scope)
    @initial_scope = initial_scope
  end
  
  def call(params)
    scoped = search(initial_scope, params[:search])
    scoped = filter_by_price(scoped, params[:from_price], params[:to_price])
    scoped = filter_by_properties(scoped, parmas[:properties])
    scoped = filter_by_category(scoped, params[:category_id])
    scoped = sort(scoped, params[:sort_type], params[:sort_direction]
    scoped = paginate(scoped, params[:page]
    scoped
  end
  
  private def search(scoped, query = nil)
    query ? scoped.where("title ILIKE '%?%'", query) : scoped
  end
  
  private def filter_by_price(scoped, from = nil, to = nil)
    from_price ? scoped.where('price > ?', from_price) : scoped
    to_price ? scoped.where('price < ?', to_price) : scoped
  end

  private def filter_by_properties(scoped, properties = nil)
    if properties
      scoped.joins(:product_properties).where(property_id: properties)
    else
      scoped
    end 
  end 
  
  private def filter_by_category(scoped, category_id = nil)
    category_id ? scoped.where(category_id: category_id) : scoped
  end
  
  private def sort(scoped, sort_type = :desc, sort_direction = :price) 
    scoped.order(sort_type => sort_direction)
  end

  private def paginate(scoped, page_number = 0)
    scoped.page(page_number)
  end
end
```
Вот такой здоровый класс получился. Зато один класс — одна задача, да и читать его гораздо проще. Вся логика объяснена в методе call:

1. сначала делаем поиск в дефолтной выборке;
2. после чего фильтруем по цене;
3. фильтруем по параметрам;
4. фильтруем по категории;
5. сортируем;
6. и наконец получаем нужную нам страницу в этой выборке.

Стоит так же наверно пояснить про default_scope. Вполне можно было бы сделать Product.all прямо в этом классе, но как мне кажется, таким образом мы сделали наш сервис гибче. Например, мы можем передавать сразу только те товары, что есть в наличии или предварительно отобранный скоуп товаров по данной геолокации. Разумеется и то и другое можно так же внести в этот сервисный объект.

## Query-object specs
 Особенно любимы сервисные объекты тем, что их достаточно просто тестировать (в отличии от спеков на контроллеры или им подобные штуки). Однако я слышал критику в адрес того, как я тестирую query-объекты. А делаю я следующим образом: я проверяю что в результирующем запросе в базу есть та или иная строка.

Например:

```ruby
RSpec.describe FindProducts do
  let(:initial_scope) { Product.all }
  
  let(:params) { {} }
  
  subject { described_class.new(initial_scope).call(params) }
  
  context 'with empty params' do
    it 'sorts' do
      expect(subject.to_sql).to include('ORDER BY "products"."price" DESC')
    end
    
    it 'paginates' do
      expect(subject.to_sql).to include('LIMIT')
      expect(subject.to_sql).to include('OFFSET')
    end
  end
end
```
Т.е. я делаю SQL из ActiveRecord::Relation объекта, и проверяю, что при нужных мне параметрах появляется та или иная строка в результирующем SQL. Критика заключается в том, что я на самом деле не тестирую результат, ведь я мог ошибиться в самом SQL, и тогда спека ничего не проверит.

Противоположностью к такому решению является создание реальных объектов в базе, когда будет производиться настоящий запрос в неё и получение объектов. После чего просто нужно сравнить те ли это объекты, что ожидаешь, или нет. 
