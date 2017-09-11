That’s funny how times change. Sometimes you start doing something that you always refused to do and thought that noting could make you do that. And now you even enjoy doing that.

For me personally, it was unclear for a long time what the frontend actually is, and integrating it with the backend seemed like a magic. When Node appeared, it became a trend to write the backend using JS. When Angular appeared, developers started to use it with everything and anything. And then there was React, Flux/Redux, but this entire frontend hype still went past me. And here’s why. Every time I took a shot to grasp this new uprising world, I was suffering with the options, instruments, new trendy procedures, arrangement of files and whatnot. The time the new framework appeared, it was already out-of-date or the whole concept was wrong. Nothing consistent here! I just couldn’t spare my time on something I very unlikely would use.

Why many of us love Rails so much? Because of the Rails way! There are many ways of doing the same task, but from all of them you will be offered the tried-and-true and the developer-approved one. No one makes you do that in this particular way, but if so, it’ll work out of the box. In JS world, unfortunately, it wasn’t like that. At least as yet.

For my job I had to work using both Rails and Angular, where, thanks to the nice initial architecture, the project support and development were all right. But the project was based on the Rails Asset Pipeline and such a decision was questioned by many new developers. 

On the last Railsclub conference, after [Zach Briggs’ speech] (https://www.youtube.com/watch?v=3cL1IKjbOus), we were talking for a solid hour on how they solve some frontend problems and that it’s a pain for everybody, and that new times call for new measures. The speech was about Vue.js and encouraged to “give JS one more chance”. Well, Zach talked me over and I did decide to give JS one more chance.

### What is Vue? 

Vue.js is a framework, which acquired its popularity as it works in Laravel (a php-clone of Rails) right out of the box. JQuery was integrated in Rails at some point, and then for Laravel they stared using Vue. Maybe that’s why it’s not really trendy nowadays, even though it seems that it’s gaining popularity with every passing day.

One of the advantages is that while the engine for the pages rendering/rerendering was being done, the author of the React engine helped the developers with this work. Thus Vue doesn’t only meet the awesome React, but also exceeds it in speed and performance.

But above all was the fact (and that was also the main reason why I actually gave JS a chance) that it offers an iterative integration. The iterative integration allows you to change your frontend little by little. If you want to add a tiny bit of interactivity to the page, you can use just one Vue app in one particular spot. Should you need to use some components, deal, a little bit here and there and there’s no need to use SPA in all your projects. Do you need lots of frontend in different spots? Just do separate micro-vue apps, one for each controller, as in any way your backend and your controller use the same resources that you allocate. And if you want the SPA, help yourself, here’s the Vue-resource which allows communicating with the SPA and here’s the Vuex for the Flux-architecture. Rock it!

<p class="notice is-new">
  <span class="left-part">
    Ivan Shamatov will teach you everything from this article 
  </span>
  <span class="right-part">
    <a onClick="ga('send', 'event', 'article button', 'learn more');" href="https://mkdev.me/en/mentors/IvanShamatov"
 class="button">Sing up</a>
  </span>
</p>

### rails/webpacker

I don’t know if you’re looking forward to the Rails 5.1 release, but I am, at least because we are promised to get the nicest instrument for the frontend work. The gem Webpacker solves lots of questions on how to integrate the frontend into the Rails app. All those files arrangements, default configurations, batch managers and everything that you usually do manually.

The gem sure enough needs some polishing, but it’s a significant step that was fiercely waited. And besides, you can already test it. So enough talk, let’s roll!

### It's coding time!

My aspirations are to write a series of articles about Vue+Rails. And hey, don’t let them fade away! As an example I’m going to use an app for cinema tickets booking. An empty app will be more than enough to close down today’s topic on how to make a basic setup for the frontend. So let’s start.

```bash
$ rails new cinematronix
```


#### Setup
In the first place let’s add all the necessary gems. You need a Webpack to do all the frontend tricks and a Foreman for initiating several processes at a time (there’ll be more about it later).

```ruby
# Gemfile
gem 'webpacker'
gem 'foreman'
```

```bash
$ bundle install
```
After installing the gems, there are more commands available in Rails for us.

```bash
$ bin/rails webpacker:install
$ bin/rails webpacker:install:vue
$ bin/yarn install
```

The first command creates a frontend setup. And you know what? I don’t even want to explain what’s going on here, as it’s of no importance for starting. Some warm memories evoke in my head, from the times when I just began working with Rails and did projects with no comprehension on how everything works.

The second one generates the template, settings and actually installs Vue.js. And all of this in just one line.

And the third one will install all the necessary npm-packages, which are defined in package.json in the root folder.

#### Vue app
When the setup is done, there will be a javascript folder in the app directory. Yep, the frontend now is not some kind of asset or whatever, but the essence of higher order. I changed the default code a little, but it’s pretty close to the one that you see here. As you can see, you natively have an almost empty application.js. The code similar to the one below is in hello_vue.js.

The thing is that the Webpacker allows us to create some packs. And I’m certain that it’s very convenient when you have several frontend-apps in your project. But for today’s goals it’s more than enough just to copy this code to application.js and delete all the ‘Hello’ mentions.

```js
// app/javascript/packs/application.js

import Vue from 'vue'
import App from '../components/app.vue'

document.addEventListener('DOMContentLoaded', () => {
  document.body.appendChild(document.createElement('app'))
  const app = new Vue({
    el: 'app',
    template: '<App/>',
    components: { App }
  })

  console.log(app)
})
```

I’ll tell you what this part does: it waits for the DOM-tree to load and then starts to initialize a vue-app. It works just as jQuery,ready (), but without jQuery.

What else did I change? The path to app.vue. Vue is a component framework, so in the manual it’s recommended to put its components into the subfolder with the same name (and I totally agree).

As it happens, I couldn’t avoid an App.vue component. But here I simply added some indents inside of each parts of the component. That’s for the purposes of convenience alone, so you can fold each tag in your favorite Sublime so they won’t disturb you.

```js
// app/javascript/components/app.vue

<template>
  <div id='app'>
    <p>{{ message }}</p>
  </div>
</template>

<script>
  export default {
    data: function () {
      return {
        message: "Welcome to Cinematronix!"
      }
    }
  }
</script>

<style scoped>
  p {
    font-size: 2em;
    text-align: center;
  }
</style>
```

That’s what the basic Vue-component looks like. It consists of the template, some logic and styles attached to this particular template. There’s actually a perfect manual for Vue.js where everything is explained in simple terms. So you can always do some self-education without my help.

#### Backend

All right! Now we need to deliver the very same app to the user. So let’s add `javascript_pack_tag` to the layout. This is a new helper from the Webpacker which takes the indicated file from the `app/javascript/packs` folder and then creates an app using the paths inside of the entry-point.

```ruby
# app/views/layouts/application.html.erb
<!DOCTYPE html>
<html>
  <head>
    <title>Cinematronix</title>
    <%= csrf_meta_tags %>

    <%= stylesheet_link_tag 'application', media: 'all' %>
    <%= javascript_pack_tag 'application' %>
  </head>

  <body>
    <%= yield %>
  </body>
</html>
```

But we don’t have even a default controller to bring back the aforementioned layout. So let’s run a couple of familiar commands.

```bash
$ bin/rails g controller Landing index
```

```ruby
# config/routes.rb
root to: 'landing#index'
```

And the last thing to do is to delete everything from our Vue in app/views/landing/index.html.erb. Clear it up!

#### 3.. 2.. 1.. Here we go!

It won’t be long now. I’ve already mentioned that we use Foreman for initiating several processes in one terminal. Of course, we could initiate the Rails-server in one tab and the frontend assembler in the other, but it’s so inconvenient! By the way, the Webpacker comes fitted with the special webpack-dev-server which compiles the app on-the-fly and loads it right into (your ear) your browser.

```
# Procfile
backend: bin/rails s -p 3000
frontend: bin/webpack-dev-server
```
But here’s the tricky bit. The assets are being downloaded from the other host, it’s localhost:8080 by default. That means that we need to uncomment the special setting for the development environment.

```ruby
# config/environments/development.rb
Rails.application.configure do
  # Make javascript_pack_tag load assets from webpack-dev-server.
  config.x.webpacker[:dev_server_host] = 'http://localhost:8080'
  ...
end
```

And as a finishing touch let’s finally run it!

```bash
$ foreman start
```

Here’s the result you should have in your browser:

![Welcome](https://habrastorage.org/files/f2d/840/513/f2d8405137574184be645a551b90f7cf.png)

### Afterword

So what is the net result? The ‘Hello, world’ vue app, attached to the Rails just in some simple steps. No headache, no npm installing, no trendy Yarn, no manual writing of package.json, no transpilers and their right versions adding, no grasping of ES5/ES6. 

In fact, you shouldn’t know any of that when you just start. But it doesn’t mean that I discourage you from becoming competent. I am just totally for the idea that an entry level should be lower. And if it was difficult for you to give it a shot, just try it.

### Sources

* [Diff on Github](https://github.com/IvanShamatov/cinematronix/pull/1/files) — what’s been added in comparison with the app made by default using Rails new
* [rails/webpacker](https://github.com/rails/webpacker)
* [vue.js guide](https://vuejs.org/v2/guide/)
* [Zach Briggs, Youtube](https://www.youtube.com/watch?v=3cL1IKjbOus)
