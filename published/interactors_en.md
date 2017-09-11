About what? - Interactors. Well anyway. You better tell me which tribe you are in: sharp-pointed or thick-headed? In other words, do you prefer skinny models or skinny controllers? That's a mean thing, indeed, to ask incorrect question on purpose. 

You must have heard such expressions while inhabiting the world of Rails.

> - All the logic should be contained in a model, a controller is just a simple http-level. The controller has received the data, it's up to the model to handle it properly and store it in the database.

Then they get the following reply:
> - Controller is called so because it should control the process. Model should only know how to map the data properly to a table in the database. 

And then [@dhh](http://twitter.com/dhh) appears in his racing car, he has just finished another race and he says:

> - Well you may do just as you like, but I have added in each directory of your project a folder named `concerns`. You can make a bunch of packages, stuff them with all of your methods and separate all the logic in a sequence of methods. 

> - No way, David, that's not cool, Rails is already not the one it used to be. It is dying, there are no solutions.

In fact the solution exists, in fact many solutions exist and they are as old as time. As uncle Fowler willed.
It is called Interactor, also a Command Pattern or Operation – these are all the names of one and the same approach inherent in DDD (Domain Driven Design). Yeah, I know, a lot of unfamiliar words intended to puzzle reader. But first things first.

#### Domain Driven Design

So, what is DDD? This is not another abbreviation standing for an approach to testing (TDD/BDD), no.

"DDD is a set of principles and procedures that help developers to create elegant object systems. When used properly, it leads to creating a lot of programming abstractions called domain models. These models include complex business logic, that eliminates the gap between the real conditions of product application field and code", - states Wikipedia and gives us a reference to [MSDN](https://msdn.microsoft.com/en-us/ren-gb/magazine/dd419654.aspx).

So, gentlemen, we are the creators of ***elegant*** object systems, we create abstractions :) It is lack of sufficient abstraction levels that Rails most often is blamed for. Though, it should seem, go and do! All in all, are you a programmer or a framework customizer?

#### Interactor

What's this one? Here are the couple of phrases describing interactors' purpose: business case object, use case object. 

Interactor is a service object that is used to create an abstraction above a little area of knowledge (domain model) and encapsulate business logic of the application.

#### TL;DR;

Let’s move from words to deeds, finally, and consider using interactors through the example.

> **Task**: Implement placing an order on a web shop site. 

> **Thinking**: Creating an order is quite broad term; it can include a lot of logic for making entry in database, effecting payment, changing status, notifying client, making task for a callback and specifying the order.

#### Step 1. Encapsulating the logic

This all is true, but if you want to start working, knowing all this is not necessary at all - we encapsulate all area of knowledge in one Interactor. 

In my example I will use gem interactor, but in reality you can write your own service object. In order to install gem we will add it to the Gemfile `gem 'interactor'` and execute `$ bundle install`. 

Let’s assume that the code of our controller, where we create an order looks like this: 

```ruby
# /app/controllers/orders_controller.rb
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
15    params.require(:order).permit(...)
16   end
17 end

```

As you see, there is a server query here, which is handled in `OrdersController` in action `create`. At line 3 we create a service object called PlaceOrder and call its method 'call' with two parameters. By the way, you can have as many parameters as you want. All they make a **context** - the thing that contains everything an interactor will work with. For example, in this case if we want to create an order we need to get information about selected items and customer. THAT'S ALL!

Then there goes some kind of magic inside, and if everything goes fine, controller returns success/redirects to the order page. If not, well, then we will redirect client to the cart page.

#### Step 2. Dividing by responsibility level. 

In most cases, that's where all the work ends. Let’s assume that placing an order is in fact just creating an order in a database. 

```ruby
# app/interactors/place_order.rb
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

And if we have solved some problem with the help of interactor - it is already a success. At least we have separated some logic that should not be placed inside a controller, because it doesn't concern http-level. It shouldn't be placed inside a model also, because it isn't a wrapper for making a proper database query (and, as we remember, that is the real goal of ActiveRecord pattern). This logic is a business case specification. In our situation 'single purpose business case' 

> **Remark**: nearly always one can describe business case like an action - "register a user", "place an order", "launch a rocket", "make a good job". This is a great way of naming an interactor. It's getting clear what is happening inside an interactor right away. 

At line 5 you can see a reference to the `context`. Parameters we pointed when calling PlaceOrder.call are passed onto this very context. Plus, at line 8 we set the value into the сontext in order to get this value later in a controller. 

Pay attention to the call `.fail!` at line 10 - that's how we warn that interactor hasn't executed and further execution will be stopped. Need I say that right here, exactly in interactor one need to implement data validation? 
I understand, at first it is really hard to admit that built-in rails validations are not the best solution. But have you ever written validations that should be activated in one case and should not be activated in another? Or did it ever happen that in different cases it is needed to run different methods on one and the same callback? They definitely don’t fit here. These methods are a part of business case, not model. And they should be placed inside an interactor. 

#### Step 2. Part 2. 

Back to our muttons: okay. But still, what if placing an order is much more broad term and you need to make 1-3-7-100 steps? 

In other words, what if we split the case "Place an order" to: 

1.  Create an order 
2.  Make payment.
3.  Send a notification. 
4.  Open a bottle of Champaign

Not a problem! We divide logic in interactors that will be called inside of other interactors. 
 
![](http://jonfriesen.ca/content/images/2013/Oct/deeper.jpg)

In this exact implementation `gem interactor` can make an organizer for multi-step interactors. An example makes it clear right away. 

```ruby
# app/interactors/place_order.rb
1 class PlaceOrder
2   include Interactor::Organzier
3 
4   organize CreateOrder, MakePayment, SendNotification, OpenBottleOfChampagne
5 end

```

The sweetest part is at line 4. That is basically just a sequence of interactors that will be run one by one. 

Furthermore, they will have common `context`, and if, say, you will need information about order in MakePayment, you can just set this information into the context in CreateOrder. Information about user will be accessible in SendNotification Interactor. 

And if during executing this sequence of interactors `context.fail!` happens somewhere, then the execution stops and further steps won't be done. I think there's no need to make a fake implementation of these inner steps. What can happen there you can perfectly imagine yourself. Alright! 

#### Testing an interactor

Okay, here we've reached the most interesting part. Testing interactors is reeeally enjoyable. As for me, I got the point of testing only when I came to know this conception: 

• Firstly, we don't need to test controller (integration tests go to hell!) 
• Secondly, we don't need to test model and all of her magic callbacks and handlers (you are not going to test metaprogramming rails, right?)
• Thirdly, testing an interactor is like testing a pure ruby object. You have set the data at the input and checked the success and context at the output. That is wickedly wonderful.

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

Okay, guys, that’s all for today. Use service objects, interactors and feel free to use non-native Rail abstractions. And may the Force be with you!

#### Additional reading

1. [Where's your business logic?](http://collectiveidea.com/blog/archives/2012/06/28/wheres-your-business-logic/)
2. [Trailblazer. A new architecture for Rails](https://leanpub.com/trailblazer)
3. [Wiki: Command Pattern](https://en.wikipedia.org/wiki/Command_pattern)
4. [Wiki: Domain Driven Design](https://en.wikipedia.org/wiki/Domain-driven_design)
