# View Layer in Ruby frameworks

OKEEEY,
we are going to talk about view layer in ruby web frameworks, especially in Rails. Have you ever thought that Rails is MTC framework? Model-Template-Controller. And we all understand that by saying "view" in the context of Rails we are talking about template. 

We all used to see code like that:

```ruby
Resource.new.update
```

We sometimes can see code like that:

```ruby
ResourcesController.new.index
```

But I'm not sure you ever used something like that:

```ruby
ResourceView.new(params).render
```

And actually quite understandable.

As you may know "MVC-paradigm" first appeared in Steve Burbeck paper ["How to use Model-View-Controller (MVC)"](http://www.math.sfedu.ru/smalltalk/gui/mvc.pdf) as it was implemented in Smalltalk-80v2. And those days, back in 80th you had not browser to interpret your HTML output. View itself was a running loop, which asks for changes from controller and renders itself. And because of the fact **browser** executes our 'view'-code we have not "real" view in web frameworks. What we expect user to see and interact with goes back and forth through HTTProtocol (not only http, but most of the time). Need to mention, I'm not talking about frontend frameworks now.

But what's the problem? We don't have 'real' view, so what? No example — no problem.
I first started to think about that, when found myself struggling with some logic in view. You can mess up really easy if you just put everything in .erb templates.

* HAML/Slim. You can make it cleaner using haml/slim, as it will be simple to navigate through markup;
* Partials. Move repeating and shared part of view in separate file;
* Helpers to the rescue. Move some 


Does anybody know any dissadvantages of using helpers in rails?

 * shared namespace (you can easily redefine any method and you will know it only after going to these actual page)
 * not really fun to make some html helpers, using html concatination or even using rails helpers to create helpers. 
 * testing... You have to check some html going to controller action and forming some external params. It's really look like testing implementation.

OK, what are the alternatives?
* partials
* We can fully rewrite our template to some ruby dsl, like ActiveAdmin. Would it help? Let's see how it might look
* Or we can use some presenters.

## So presenters
Presenters are just PORO with a mixture of delegation.

```ruby
# app/presenters/post_presenter.rb
class PostPresenter
  attr_reader :object
  
  def initialize(object)
    @object = object
  end
  
  def author_link
    link_to object.author_name, object.author
  end
end
```



## Cells
