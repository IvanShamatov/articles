# Ассоциации в Rails

## Введение
Все мы любим, или по крайней мере любили Rails за её "магию". Когда мы говорим о магии, мы всегда подразумеваем именно факт чего-то, что мы еще не в силах объяснить. Если ты новичок, то чтобы получить работающее приложение не нужно прилагать много умственных усилий. И для какого-нибудь типового приложения зачастую можно найти либо уже готовое решение, либо хороший туториал. Всё работает, как говорится, "из коробки". Но если сделать шаг влево, шаг вправо, магия перестаёт быть подконтрольной и любое действие ведёт к полнейшему непониманию, что происходит и как это работает, а главное, как это работало раньше. 

Одна из таких тем, которая вызывает много вопросов — это ассоциации. Всё, что необходимо знать про ассоциации можно почерпнуть по двум ссылкам: [официальный гайд](), [официальная документашка](). Эти две статьи требуют долгого и вдумчивого прочтения, но это достаточное требование, чтобы полностью разобраться в теме ассоциаций в Rails с одним условием: вы уже должны иметь начальное представление о базах данных, о том как строятся таблицы и о внешних ключах.

Конечно же, зачастую это не так, и можно сколь угодно долго тыкать человека в эти два гайда, если он не знает предметной области, на которой базируются эти гайды — это бесполезно. А потому, хоть эта статья/серия статей про ассоциации, но разбираться мы будем в ключе построения баз данных и конечно же смотреть всё на примерах. Поехали!

## Немного об ActiveRecord
Нужно понимать, что те ассоциации, которые представлены в Rails по-умолчанию, работают для единственного гема active_record, который поставляется из коробкии и в 95% случаев покрывает потребности любого rails разработчика. Но существуют и другие гемы для работы с базами данных. 

Например, [sequel](https://github.com/jeremyevans/sequel) — отличный гем и если вам нужно сделать небольшое приложение, и не хочется пол-рельсы тащить в виде active-record'a, то использовать sequel будет отличным решением. Там ассоциации реализуются другими методами, но концептуально не отличаются. 

Или же можно взять mongoid для нереляционной базы данных MongoDB, где эти самые релейшны существуют в совершенно другом представлении, выборки осуществляются не SQL-запросами, а Map-Reduce'ом.

Раз уж мы определились, что всё нижесказанное будет касаться ActiveRecord'a, то теперь можно переходить к самому паттерну и основам баз данных. Итак, ActiveRecord — это паттерн, я уже упоминал его в одной из предыдущих статей. [TODO: вставить что-то из википедии]. 

Гем active_record является ORM'ом

Для каждой модели (именно модели — класса, отнаследованного от ActiveRecord::Base), которую мы создаем в нашем Rails приложении соответствует таблица в базе данных. По умолчанию на неё распространяются следующие правила: она должна быть названа так же как модель в множественном числе (например, модель Movie, а таблица будет movies) и в таблице должен быть столбец id. 

Когда мы создаем объект и сохраняем его в базе данных происходит следующее:

```ruby
movie = Movie.new(
  title: 'The Matrix 2: raise of machines',
  duration: 3600,
  producer: 'Vachovski brothers',
  imdb_rating: 9.1
)
movie.save
```

Метод `save`, берет все аттрибуты, которые есть у этой модели и выставляет соответствие каждого аттрибута конкретному столбцу в базе данных. 

[TODO: табличку мапинга из памяти в запись]

```sql
INSERT INTO "movies" ("title", "duration", "producer", "imdb_rating", "created_at", "updated_at") VALUES (?, ?, ?, ?, ?, ?)  [["title", "The Matrix"], ["duration", 3600], ["producer", "Vachovski brothers"], ["imdb_rating", 9.1], ["created_at", "2017-06-05 17:59:22.862206"], ["updated_at", "2017-06-05 17:59:22.862206"]]
```

А в результате после совершения этой команды мы увидим вот что-то похожее:

```ruby
 => #<Movie id: 36, title: "The Matrix", description: nil, duration: 3600, producer: "Vachovski brothers", imdb_rating: 9.1, created_at: "2017-06-05 17:59:22", updated_at: "2017-06-05 17:59:22"> 
```

На что мы обратим внимание здесь: 
`id: 36` — конечно, это может быть любой другой id

Теперь мы можем обратиться по этому самому айдишнику и получить на выходе ту же самую запись.

```sql
SELECT  "movies".* FROM "movies" WHERE "movies"."id" = ? LIMIT ?  [["id", 36], ["LIMIT", 1]]
```

[TODO: табличку мапинга из рекорда 

```ruby
Movie.find(36)
```

## belongs_to 
В рамках курса на Mkdev, есть одно классное задание про выбор текущей колоды, на котором сразу становится понятно, насколько хорошо ты разобрался в теме ассоциаций


## has_one
## has_many 
## has_and_belongs_to_many
## has_many :through
