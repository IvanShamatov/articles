Shock! Not the full truth about decorators, everything that they have been hiding from us! Without registration and SMS.

> Gem 'draper' is not a decorator! And 'cells' too.

- Okay, people, move along!  It's a wrap. Everyone knows it already. Right?

The first article [about interactors](https://mkdev.me/en/posts/a-couple-of-words-about-interactors-in-rails) quickened interest in the audience. So here is the continuance, this time about decorators!

### To the point

Let's agree from the beginning: I adhere to the GOF definition of a 'Decorator' pattern. That is exactly what [Wikipedia says](https://en.wikipedia.org/wiki/Decorator_pattern).

> Decorator is a design pattern that allows behavior to be added to an individual object dynamically. Decorator pattern provides the flexible alternative to the practice of creating subclasses in order to extend functionality.
>
> The added functionality is realized in a small objects. The advantage is that one can dynamically add the functionality before or after the main object functionality.

#### Given:

As always, the example makes it easier to understand. We will continue to discuss the order placement, more specifically, we will implement the delivery cost calculation.

However strange it may sound, but the delivery cost depends on the delivery volume, weight, destination, whether it will be up-to-door shipping or the delivery to the city access point, whether you need the order to be wrapped in a bubble-pack or in a bandel, and, of course, which delivery service you are going to use. In short, there is a load of parameters! Just look at the site of any shipping company and you will see the calculator with thousands of parameters.

In the beginning we will have the following set.

We have the class `Order` with attributes: weight, width, height, length and city where the order has to be delivered to.

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
 
  def volume # not the volume of sound
       width*height*length
  end
end
```

Also, we have an external service for calculating the delivery cost depending on the city (always returns 100).

```ruby
class DlhApi # the delivery service with a red-yellow logo
  def self.calculate(city)
    100
  end
end
```

And the delivery class itself.

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

order = Order.new(10, 5, 1, 1, "London")
Delivery.new(order).cost #=> 15
```

Okay, this is done. There is an order. There is a delivery our order is passed onto. And there is a method for calculating the delivery cost. Seems pretty easy so far.

#### The solution:

1. Brute-force solution. It is worth noting that one shouldn't always go after beautiful abstractions. Sometimes the brute force solution is the right thing.

  And the solution is that we add all our "cost modifiers" into the class `Delivery` itself as options. And there in the method `cost` we will change the final cost just directly searching the options. All in all, why not, if there are only 2-3 modifiers? But we have a lot of them. Just the delivery services alone can count 10.

2. The inheritance solution. We can create the subclass `OneDayDelivery`, `DlhDelivery`,`UpToDoorDelivery`, which we will create without any if-s. But following the same logic, we will have to create the subclass `OneDayDlhUpToDoorBubbleWrappedDelivery`. And that is the problem that the Decorator pattern is destined to solve.

3. Solution with decorators. And that's how it will look like.

Decorators have a sort of tradition - every New Year they gather together in a bath-house and wrap each other ([if you know what I mean](https://en.wikipedia.org/wiki/The_Irony_of_Fate)). But speaking seriously, decorator is a design pattern, the subtype of a composite pattern. We can wrap one decorator with another, then another, and this way every new layer will add the new functionality. Here's a simple example:

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

We save the order that we are decorating upon initialization, and then call its method `cost`, inside our implementation of the method `cost`.
From this we can conclude: all the decorators of one object should have the common interface (in our case this is the method `cost`)


#### Offtop

> **Reader's question**: The most obvious question - the code organization.... What and where
> should be placed....

> **Expert committee answer**: These shoulda be placed ova hea!
>
> And as an argument to the expert committee answer I'll attach the notarized screenshot.

![](https://habrastorage.org/files/cb3/926/a13/cb3926a13988449c95ebd789bfb92edb.png)

Such structure is evident and easily understandable for me.

* First of all, decorators are placed in the decorators' folder.
* Secondly, such structure allows me to utilize rails' namespaces. If I place something in the folder 'delivery' and after that I will have the namespace 'Delivery', too - that is cool!

<p class="notice clearfix">
<a href="https://mkdev.me/en/mentors/IvanShamatov" class="button pull-right" onClick="ga('send', 'event', 'article button', 'learn more');">Enroll</a>
The author of this article can become <br />your personal mentor.
</p>

#### Let's keep going

I'm sure that you've already had a sneaky peek at the bubble decorator implementation on the picture. That's why I'll give an example of the two other decorators' implementation.

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

order = Order.new(10, 5, "London")
# Just delivery
Delivery.new( order ).cost #=> 15

# Expedited shipping
Delivery::Dlh.new( Delivery.new(order)).cost #=> 115

# Expedited shipping, the parcel is wrapped in a bubble-pack
Delivery::BubbleWrapped.new(
  Delivery::Dlh.new(
    Delivery.new( order )
  )
).cost #=> 125

# Up to door shipping, the parcel is wrapped in a bubble-pack
Delivery::UpToDoor.new(
  Delivery::BubbleWrapped.new(
    Delivery::Dlh.new(
      Delivery.new( order )
    )
  )
).cost #=> 140

```

Isn't that wonderful? We upped and divided all this *complicated* logic to the logical units, put it into different files, we make such beautiful calls. Can it be any better?

#### It can!

Homework - figure out how all this works, here's an example.

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


order = Order.new(10, 5, 1, 1, "London")
delivery = Delivery.new(order)
delivery.cost #=> 15

delivery.extend(Delivery::Dhl)
delivery.cost #=> 115

delivery.extend(Delivery::BubbleWrapped)
delivery.cost #=> 125

delivery.extend(Delivery::UpToDoor)
delivery.cost #=> 140
```

#### Sources

This is not simply referring to where I got the material from. This is for more extensive self-study. Trust me, this is worth it.

1. [Design Patterns in Ruby. The chapter about decorators] (http://www.informit.com/store/design-patterns-in-ruby-9780321490452)
2. [Tidy Views and Beyond with Decorators](https://robots.thoughtbot.com/tidy-views-and-beyond-with-decorators)
3. [Evaluating Alternative Decorator Implementations In Ruby](https://robots.thoughtbot.com/evaluating-alternative-decorator-implementations-in)
4. [Decorators, Presenters, Delegators and Rails](https://robertomurray.co.uk/blog/2014/decorators-presenters-delegators-rails/)
5. [Stackoverflow. Decorators vs presenters](http://stackoverflow.com/questions/7860301/ruby-on-rails-patterns-decorator-vs-presenter)

#### P.S. And now, what is wrong with the draper?

Well, everything's fine. Gem `draper` allows us to create a full decorator.
In order to get that done, one should add into the `Draper::Decorator` the call `delegate_all`, so that our call would be passed in chain order unless the last layer implements the method we need.

Ah, and there's one more restriction at the moment, draper isn't able to wrap decorator and another similar decorator (ref. the last 2 lines of the example), the same problem as with the modular approach.

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

The reason why I dared to make such a loud statement is that `draper` is not a decorator in the way that it's used. And it is used as a bridge between a model and a view. And that's how they position themselves -  Decorators/View-Models for Rails Applications.

But in fact the `Presenter` pattern is responsible for that. But we'll talk about it later on. Stay tuned.
