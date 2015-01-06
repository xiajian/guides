Getting Started with Engines
============================

In this guide you will learn about engines and how they can be used to provide
additional functionality to their host applications through a clean and very
easy-to-use interface.

本指南将介绍engines，了解engines如何通过干净易用的接口为其宿主程序添加功能。


读完本文，你将学到：

* 什么是engine
* 怎样生成engine
* 为engine创建特性
* 将engine嵌入到应用中
* 在应用中覆盖Engine的功能

--------------------------------------------------------------------------------

What are engines?
-----------------

Engines可以看作为宿主应用提供功能的最小化应用程序。Rails应用实际上是“增强版”的
engine，其中，`Rails::Application`类从`Rails::Engine`类中继承了很多方法。

所以，engines和应用可以看作是仅有轻微区别的同一事物。正如你所见，Engines与应用
共享通用的结构。

Engines are also closely related to plugins. The two share a common `lib`
directory structure, and are both generated using the `rails plugin new`
generator. The difference is that an engine is considered a "full plugin" by
Rails (as indicated by the `--full` option that's passed to the generator
command). We'll actually be using the `--mountable` option here, which includes
all the features of `--full`, and then some. This guide will refer to these 
"full plugins" simply as "engines" throughout. An engine **can** be a plugin,
and a plugin **can** be an engine.

Engines与插件也很相似。两者共享相同的`lib`目录结构，并且都可通过`rails plugin new`
创建。唯一的区别在于，engine被Rails看作"full plugin"(即在使用`rails plugin new`命令时，传递
`--full`选项)。也可使用`--mountable`选项，其包含了`--full`的所有特性，以及一些其他特性。本指南
将"full plugins"简化为engines。engine可以是插件，插件也可称为engine。

本指南中创建名为"blorgh"的engine，其提供为宿主应用程序提供博客的功能，并允许创建
新的文章和注释。在指南的开始，仅关于engine本身。但在随后的章节，将看到如何将其挂载到
应用程序中。

Engine可以从宿主应用程序中隔离。这意味着，应用程序的中`articles_path`路由辅助方法和engine中的
路由辅助方法并不冲突。除此之外，控制器，模型以及表名都会被命名空间隔离。随后，你就会看到如何
处理这个。

It's important to keep in mind at all times that the application should
**always** take precedence over its engines. An application is the object that
has final say in what goes on in its environment. The engine should
only be enhancing it, rather than changing it drastically.

注意: **应用程序总是优先于其engine**。在自己的运行环境中，应用程序说了算，engine仅仅起到辅助
增强的作用，而不是彻底的改变它。

可以通过查看engine的代码，从而加强对其的认识: 

* [Devise](https://github.com/plataformatec/devise) - 提供授权的engine
* [Forem](https://github.com/radar/forem) - 提供论坛功能的engine
* [Spree](https://github.com/spree/spree) - 提供电子商务平台的engine
* [RefineryCMS](https://github.com/refinery/refinerycms) - CMS engine

最后，没有James Adam、Piotr Sarnacki、Rails核心团队以及其他的所有人， engine将不会诞生。如果你
见到他们，别忘了道谢。

Generating an engine
--------------------

To generate an engine, you will need to run the plugin generator and pass it
options as appropriate to the need. For the "blorgh" example, you will need to
create a "mountable" engine, running this command in a terminal:

为了生成engine, 需要运行plugin生成器，并传递所需的最佳选项。以"blorgh"为例，可能需要传递
"mountable"选项，命令如下:

```bash
$ bin/rails plugin new blorgh --mountable
```

插件生成器可选的全部列表如下(注意，不要在rails项目下运行该命令):

```bash
$ bin/rails plugin --help
```

The `--mountable` option tells the generator that you want to create a
"mountable" and namespace-isolated engine. This generator will provide the same
skeleton structure as would the `--full` option. The `--full` option tells the
generator that you want to create an engine, including a skeleton structure
that provides the following:

`--mountable`选项告诉生成器，创建一个"mountable"且命名隔离的engine。生成器将提供与`--full`选项
类似的框架结构。`--full`选项创建包含如下结构的engine :

  * `app`目录树directory tree
  * 包含如下内容的`config/routes.rb`文件:

    ```ruby
    Rails.application.routes.draw do
    end
    ```

  * `lib/blorgh/engine.rb`，其功能类似Rails项目的`config/application.rb`:

    ```ruby
    module Blorgh
      class Engine < ::Rails::Engine
      end
    end
    ```

`--mountable`选项比`--full`多加的内容:

  * 资源清单文件(`application.js` and `application.css`)
  * `ApplicationController`的命名空间的桩
  * `ApplicationHelper`命名空间的桩
  * engine的视图布局模板
  * `config/routes.rb`中的命名隔离 :

    ```ruby
    Blorgh::Engine.routes.draw do
    end
    ```

  * `lib/blorgh/engine.rb`中的命名空间隔离:

    ```ruby
    module Blorgh
      class Engine < ::Rails::Engine
        isolate_namespace Blorgh
      end
    end
    ```

Additionally, the `--mountable` option tells the generator to mount the engine
inside the dummy testing application located at `test/dummy` by adding the
following to the dummy application's routes file at
`test/dummy/config/routes.rb`:

此外，`--mountable`选项通知生成器，可以将engine挂载到某个应用程序中，并在应用的路由文件(`config/routes.rb`)
添加如下的内容: 

```ruby
mount Blorgh::Engine, at: "blorgh"
```

### Inside an Engine

#### Critical Files

在每个engine目录下，都存在`blorgh.gemspec`文件。当将engine包含到应用程序中时，需要在
Rails应用程序的`Gemfile`，添加如下的行: 

```ruby
gem 'blorgh', path: "vendor/engines/blorgh"
```

Don't forget to run `bundle install` as usual. By specifying it as a gem within
the `Gemfile`, Bundler will load it as such, parsing this `blorgh.gemspec` file
and requiring a file within the `lib` directory called `lib/blorgh.rb`. This
file requires the `blorgh/engine.rb` file (located at `lib/blorgh/engine.rb`)
and defines a base module called `Blorgh`.

不要忘记运行`bundle install`。通过在`Gemfile`中指定gem，Bundler将会加载，并解析`blorgh.gemspec`文件，加载`lib/blorgh.rb`
文件。`lib/blorgh.rb`需要require `blorgh/engine.rb`文件(位于`lib/blorgh/engine.rb`)，并定义名为`Blorgh`的基础模块。

```ruby
require "blorgh/engine"

module Blorgh
end
```


**提示**: 某些engines使用该文件(`lib/blorgh.rb`)设置全局配置选项。这是个相当不错的注意，如果想要提供
配置选项，就优先考虑将配置放置在这个定义了`module`的文件中。

`lib/blorgh/engine.rb`定义了engine的基类:

```ruby
module Blorgh
  class Engine < Rails::Engine
    isolate_namespace Blorgh
  end
end
```

通过继承`Rails::Engine`类，从而告知Rails在特定路径中存在一个engine，并将engine正确
的挂载到应用程序中，执行诸如将 engine 的 `app`目录下的models、mailers、controllers以及
views分别添加到加载路径中。

这里需要特别注意的是`isolate_namespace`方法。该方法负责将控制器，模型，路由和其他东西放到独立
的命名空间中，隔离应用中类似的组件。没有该方法，engine的组件可能“泄漏”到应用程序，造成不必要的中断，
或者engine组件被应用程序中类似的东西覆盖。其中，冲突的一个例子是： helper方法的冲突，不调用`isolate_namespace`
engine中的helper方法将包含到应用的控制器中。

NOTE: It is **highly** recommended that the `isolate_namespace` line be left
within the `Engine` class definition. Without it, classes generated in an engine
**may** conflict with an application.

注意: 高度推荐将`isolate_namespace`放置在`Engine`类定义的左边。若不如此，在engine中生成的类
将会在应用中引起冲突。

命名空间的隔离意味着，通过调用`bin/rails g model`生成的模型，比如`bin/rails g model article`，生成的类不是
`Article`，而是`Blorgh::Article`。此外，模型的表也被命名空间化，为`blorgh_articles`，而不是简单的`articles`。
类似模型，控制器为`Blorgh::ArticlesController`，视图为`app/views/blorgh/articles`。Mailers也是如此。

Finally, routes will also be isolated within the engine. This is one of the most
important parts about namespacing, and is discussed later in the
[Routes](#routes) section of this guide.

最后，路由也会在engine中被隔离，这是命名空间中最重要的一部分，并将在指南的[Routes]()章节中被讨论。

#### `app` Directory

Inside the `app` directory are the standard `assets`, `controllers`, `helpers`,
`mailers`, `models` and `views` directories that you should be familiar with
from an application. The `helpers`, `mailers` and `models` directories are
empty, so they aren't described in this section. We'll look more into models in
a future section, when we're writing the engine.

在`app`目录中，是标准的`assets`, `controllers`, `helpers`, `mailers`, `models` , `views`
应用程序目录。`helpers`, `mailers`和`models`目录是空的，所以，在此不作讨论，而`models`将
会在以后实现`engine`的章节进一步讨论。

`app/assets`目录也和应用程序中相似。唯一不同的是，文件都被放在以engine名分隔的子目录下。由于
engine被命名隔离，其资源文件也一样。

Within the `app/controllers` directory there is a `blorgh` directory that
contains a file called `application_controller.rb`. This file will provide any
common functionality for the controllers of the engine. The `blorgh` directory
is where the other controllers for the engine will go. By placing them within
this namespaced directory, you prevent them from possibly clashing with
identically-named controllers within other engines or even within the
application.

在`app/controllers`目录下，名为`blorgh`的目录下，包含着`application_controller.rb`。
`application_controller.rb`中包含了engine中控制器的通用功能。engine的控制器放在`blorgh`
目录下，从而，避免与应用中已存在的控制器引起冲突。

**注意**: engine中的`ApplicationController`和Rails应用程序中相同，从而方便将应用程序转换为
engine。

最后，`app/views`目录下包含`layouts`目录，其中，包含的文件为`blorgh/application.html.erb`。
该文件为engine指定布局。如果engine仅仅作为单独的engine，可以在`blorgh/application.html.erb`
为engine添加定制化的布局，而不是使用应用的`app/views/layouts/application.html.erb`文件。

如果不想强迫用户使用特定的布局，可以将该文件删除，然后在engine的控制器用引用其他的布局。

#### `bin` Directory

This directory contains one file, `bin/rails`, which enables you to use the
`rails` sub-commands and generators just like you would within an application.
This means that you will be able to generate new controllers and models for this
engine very easily by running commands like this:

`bin`目录包含名为`rails`的文件。该文件确保可以使用`rails`子命令以及相关的生成器，如同
在Rails应用中那样。这意味着，如下的命令均可使用: 

```bash
$ bin/rails g model
```

Keep in mind, of course, that anything generated with these commands inside of
an engine that has `isolate_namespace` in the `Engine` class will be namespaced.

谨记: 由于`Engine`中的`isolate_namespace`方法，命令生成的任何东西都是命名空间隔离。

#### `test` Directory

The `test` directory is where tests for the engine will go. To test the engine,
there is a cut-down version of a Rails application embedded within it at
`test/dummy`. This application will mount the engine in the
`test/dummy/config/routes.rb` file:

```ruby
Rails.application.routes.draw do
  mount Blorgh::Engine => "/blorgh"
end
```

上述代码行将engine挂载到`/blorgh`路径下，应用程序总是可以通过该路径访问engine。

Inside the test directory there is the `test/integration` directory, where
integration tests for the engine should be placed. Other directories can be
created in the `test` directory as well. For example, you may wish to create a
`test/models` directory for your model tests.

测试文件中，存在集成测试和各种模块测试(model, controller, view)。

Providing engine functionality
------------------------------

The engine that this guide covers provides submitting articles and commenting
functionality and follows a similar thread to the [Getting Started Guide](getting_started.html), 
with some new twists.

本指南所描述的engine，提供提交文章和注释的功能，其类似[Getting Started Guide](getting_started.html)
中描述的程序，但稍有区别。

### Generating an Article Resource

The first thing to generate for a blog engine is the `Article` model and related
controller. To quickly generate this, you can use the Rails scaffold generator.

首先为`blog`生成`Article`模型，以及相关的控制器。为了快速生成资源，需要使用scaffold生成器。

```bash
$ bin/rails generate scaffold article title:string text:text
```

This command will output this information:

```
invoke  active_record
create    db/migrate/[timestamp]_create_blorgh_articles.rb
create    app/models/blorgh/article.rb
invoke    test_unit
create      test/models/blorgh/article_test.rb
create      test/fixtures/blorgh/articles.yml
invoke  resource_route
 route    resources :articles
invoke  scaffold_controller
create    app/controllers/blorgh/articles_controller.rb
invoke    erb
create      app/views/blorgh/articles
create      app/views/blorgh/articles/index.html.erb
create      app/views/blorgh/articles/edit.html.erb
create      app/views/blorgh/articles/show.html.erb
create      app/views/blorgh/articles/new.html.erb
create      app/views/blorgh/articles/_form.html.erb
invoke    test_unit
create      test/controllers/blorgh/articles_controller_test.rb
invoke    helper
create      app/helpers/blorgh/articles_helper.rb
invoke      test_unit
create        test/helpers/blorgh/articles_helper_test.rb
invoke  assets
invoke    js
create      app/assets/javascripts/blorgh/articles.js
invoke    css
create      app/assets/stylesheets/blorgh/articles.css
invoke  css
create    app/assets/stylesheets/scaffold.css
```

The first thing that the scaffold generator does is invoke the `active_record`
generator, which generates a migration and a model for the resource. Note here,
however, that the migration is called `create_blorgh_articles` rather than the
usual `create_articles`. This is due to the `isolate_namespace` method called in
the `Blorgh::Engine` class's definition. The model here is also namespaced,
being placed at `app/models/blorgh/article.rb` rather than `app/models/article.rb` due
to the `isolate_namespace` call within the `Engine` class.

scaffold生成器所做的第一件事是，调用`active_record`生成器，生成迁移和资源模型。注意，
这里的迁移名为`create_blorgh_articles`，而不是`create_articles`。模型文件也是如此。

> 注: 每个invoke都代表调用一个生成器。

Next, the `test_unit` generator is invoked for this model, generating a model
test at `test/models/blorgh/article_test.rb` (rather than
`test/models/article_test.rb`) and a fixture at `test/fixtures/blorgh/articles.yml`
(rather than `test/fixtures/articles.yml`).

接下来，调用`test_unit`生成器，为模型生成测试。

After that, a line for the resource is inserted into the `config/routes.rb` file
for the engine. This line is simply `resources :articles`, turning the
`config/routes.rb` file for the engine into this:

随后，在`config/routes.rb`中，插入一行资源文件，即`resources :articles`。

```ruby
Blorgh::Engine.routes.draw do
  resources :articles
end
```

Note here that the routes are drawn upon the `Blorgh::Engine` object rather than
the `YourApp::Application` class. This is so that the engine routes are confined
to the engine itself and can be mounted at a specific point as shown in the
[test directory](#test-directory) section. It also causes the engine's routes to
be isolated from those routes that are within the application. The
[Routes](#routes) section of this guide describes it in detail.

Next, the `scaffold_controller` generator is invoked, generating a controller
called `Blorgh::ArticlesController` (at
`app/controllers/blorgh/articles_controller.rb`) and its related views at
`app/views/blorgh/articles`. This generator also generates a test for the
controller (`test/controllers/blorgh/articles_controller_test.rb`) and a helper
(`app/helpers/blorgh/articles_controller.rb`).

随后，调用`scaffold_controller`生成器，生成名为`Blorgh::ArticlesController`
(位于`app/controllers/blorgh/articles_controller.rb`)的控制器，以及对控制器
相关的测试(`test/controllers/blorgh/articles_controller_test.rb`)以及辅助方法
(`app/helpers/blorgh/articles_controller.rb`)。

Everything this generator has created is neatly namespaced. The controller's
class is defined within the `Blorgh` module:

```ruby
module Blorgh
  class ArticlesController < ApplicationController
    ...
  end
end
```

NOTE: The `ApplicationController` class being inherited from here is the
`Blorgh::ApplicationController`, not an application's `ApplicationController`.

注意: 这里的`ApplicationController`类是从`Blorgh::ApplicationController`继承而来的
而不是`ApplicationController`。

The helper inside `app/helpers/blorgh/articles_helper.rb` is also namespaced:

```ruby
module Blorgh
  module ArticlesHelper
    ...
  end
end
```

This helps prevent conflicts with any other engine or application that may have
an article resource as well.

Finally, the assets for this resource are generated in two files:
`app/assets/javascripts/blorgh/articles.js` and
`app/assets/stylesheets/blorgh/articles.css`. You'll see how to use these a little
later.

最终，生成资源文件，即`app/assets/javascripts/blorgh/articles.js` 和 `app/assets/stylesheets/blorgh/articles.css`。

By default, the scaffold styling is not applied to the engine because the
engine's layout file, `app/views/layouts/blorgh/application.html.erb`, doesn't
load it. To make the scaffold styling apply, insert this line into the `<head>`
tag of this layout:

由于engine的布局文件的原因，scaffold的样式并不因用到engine中。为了应用scaffold的样式，
在布局的`<head>`中，插入如下的行: 

```erb
<%= stylesheet_link_tag "scaffold" %>
```

You can see what the engine has so far by running `rake db:migrate` at the root
of our engine to run the migration generated by the scaffold generator, and then
running `rails server` in `test/dummy`. When you open
`http://localhost:3000/blorgh/articles` you will see the default scaffold that has
been generated. Click around! You've just generated your first engine's first
functions.

在engine的根目录下，运行`rake db:migrate`，就可以看到项目，然后，在嵌入的测试项目中，运行
`rails server`，打开`http://localhost:3000/blorgh/articles`，看到默认生成的scaffold。

If you'd rather play around in the console, `rails console` will also work just
like a Rails application. Remember: the `Article` model is namespaced, so to
reference it you must call it as `Blorgh::Article`.

如果，你更加喜欢控制台，`rails console`的效果和Rails应用程序中类似，只不过，`Article`模型被命名空间隔离，
引用的时候需要调用`Blorgh::Article`。

```ruby
>> Blorgh::Article.find(1)
=> #<Blorgh::Article id: 1 ...>
```

One final thing is that the `articles` resource for this engine should be the root
of the engine. Whenever someone goes to the root path where the engine is
mounted, they should be shown a list of articles. This can be made to happen if
this line is inserted into the `config/routes.rb` file inside the engine:

最后，engine中的`articles`资源应该是engine的根。无论何时何处，engine的根目录都应该
列出文章列表。这需要在engine的`config/routes.rb`写入如下的行: 

```ruby
root to: "articles#index"
```

Now people will only need to go to the root of the engine to see all the articles,
rather than visiting `/articles`. This means that instead of
`http://localhost:3000/blorgh/articles`, you only need to go to
`http://localhost:3000/blorgh` now.

### Generating a Comments Resource

Now that the engine can create new articles, it only makes sense to add
commenting functionality as well. To do this, you'll need to generate a comment
model, a comment controller and then modify the articles scaffold to display
comments and allow people to create new ones.

From the application root, run the model generator. Tell it to generate a
`Comment` model, with the related table having two columns: a `article_id` integer
and `text` text column.

```bash
$ bin/rails generate model Comment article_id:integer text:text
```

This will output the following:

```
invoke  active_record
create    db/migrate/[timestamp]_create_blorgh_comments.rb
create    app/models/blorgh/comment.rb
invoke    test_unit
create      test/models/blorgh/comment_test.rb
create      test/fixtures/blorgh/comments.yml
```

This generator call will generate just the necessary model files it needs,
namespacing the files under a `blorgh` directory and creating a model class
called `Blorgh::Comment`. Now run the migration to create our blorgh_comments
table:

```bash
$ rake db:migrate
```

To show the comments on an article, edit `app/views/blorgh/articles/show.html.erb` and
add this line before the "Edit" link:

```erb
<h3>Comments</h3>
<%= render @article.comments %>
```

This line will require there to be a `has_many` association for comments defined
on the `Blorgh::Article` model, which there isn't right now. To define one, open
`app/models/blorgh/article.rb` and add this line into the model:

```ruby
has_many :comments
```

Turning the model into this:

```ruby
module Blorgh
  class Article < ActiveRecord::Base
    has_many :comments
  end
end
```

NOTE: Because the `has_many` is defined inside a class that is inside the
`Blorgh` module, Rails will know that you want to use the `Blorgh::Comment`
model for these objects, so there's no need to specify that using the
`:class_name` option here.

Next, there needs to be a form so that comments can be created on an article. To
add this, put this line underneath the call to `render @article.comments` in
`app/views/blorgh/articles/show.html.erb`:

```erb
<%= render "blorgh/comments/form" %>
```

Next, the partial that this line will render needs to exist. Create a new
directory at `app/views/blorgh/comments` and in it a new file called
`_form.html.erb` which has this content to create the required partial:

```erb
<h3>New comment</h3>
<%= form_for [@article, @article.comments.build] do |f| %>
  <p>
    <%= f.label :text %><br>
    <%= f.text_area :text %>
  </p>
  <%= f.submit %>
<% end %>
```

When this form is submitted, it is going to attempt to perform a `POST` request
to a route of `/articles/:article_id/comments` within the engine. This route doesn't
exist at the moment, but can be created by changing the `resources :articles` line
inside `config/routes.rb` into these lines:

```ruby
resources :articles do
  resources :comments
end
```

This creates a nested route for the comments, which is what the form requires.

The route now exists, but the controller that this route goes to does not. To
create it, run this command from the application root:

```bash
$ bin/rails g controller comments
```

This will generate the following things:

```
create  app/controllers/blorgh/comments_controller.rb
invoke  erb
 exist    app/views/blorgh/comments
invoke  test_unit
create    test/controllers/blorgh/comments_controller_test.rb
invoke  helper
create    app/helpers/blorgh/comments_helper.rb
invoke    test_unit
create      test/helpers/blorgh/comments_helper_test.rb
invoke  assets
invoke    js
create      app/assets/javascripts/blorgh/comments.js
invoke    css
create      app/assets/stylesheets/blorgh/comments.css
```

The form will be making a `POST` request to `/articles/:article_id/comments`, which
will correspond with the `create` action in `Blorgh::CommentsController`. This
action needs to be created, which can be done by putting the following lines
inside the class definition in `app/controllers/blorgh/comments_controller.rb`:

```ruby
def create
  @article = Article.find(params[:article_id])
  @comment = @article.comments.create(comment_params)
  flash[:notice] = "Comment has been created!"
  redirect_to articles_path
end

private
  def comment_params
    params.require(:comment).permit(:text)
  end
```

This is the final step required to get the new comment form working. Displaying
the comments, however, is not quite right yet. If you were to create a comment
right now, you would see this error:

```
Missing partial blorgh/comments/comment with {:handlers=>[:erb, :builder],
:formats=>[:html], :locale=>[:en, :en]}. Searched in:   *
"/Users/ryan/Sites/side_projects/blorgh/test/dummy/app/views"   *
"/Users/ryan/Sites/side_projects/blorgh/app/views"
```

The engine is unable to find the partial required for rendering the comments.
Rails looks first in the application's (`test/dummy`) `app/views` directory and
then in the engine's `app/views` directory. When it can't find it, it will throw
this error. The engine knows to look for `blorgh/comments/comment` because the
model object it is receiving is from the `Blorgh::Comment` class.

This partial will be responsible for rendering just the comment text, for now.
Create a new file at `app/views/blorgh/comments/_comment.html.erb` and put this
line inside it:

```erb
<%= comment_counter + 1 %>. <%= comment.text %>
```

The `comment_counter` local variable is given to us by the `<%= render
@article.comments %>` call, which will define it automatically and increment the
counter as it iterates through each comment. It's used in this example to
display a small number next to each comment when it's created.

That completes the comment function of the blogging engine. Now it's time to use
it within an application.

Hooking Into an Application
---------------------------

Using an engine within an application is very easy. This section covers how to
mount the engine into an application and the initial setup required, as well as
linking the engine to a `User` class provided by the application to provide
ownership for articles and comments within the engine.

在应用程序中使用engine是非常的容易的。本节覆盖如何将engine挂载到应用程序中，初始化设置，以及
将engine链接到应用程序提供的`User`类中，从而提供文章和注释的关联用户的关系。

### Mounting the Engine

First, the engine needs to be specified inside the application's `Gemfile`. If
there isn't an application handy to test this out in, generate one using the
`rails new` command outside of the engine directory like this:

首先，需要在应用程序的`Gemfile`中，指定engine。如果没有现成的测试应用程序，就使用`rails new`
生成一个新的。

```bash
$ rails new unicorn
```

Usually, specifying the engine inside the Gemfile would be done by specifying it
as a normal, everyday gem.

```ruby
gem 'devise'
```

However, because you are developing the `blorgh` engine on your local machine,
you will need to specify the `:path` option in your `Gemfile`:

如果，在本地机器开发`blorgh`，可以在`Gemfile`中，使用`:path`选项来指定。

```ruby
gem 'blorgh', path: "/path/to/blorgh"
```

运行`bundle`来安装gen。

As described earlier, by placing the gem in the `Gemfile` it will be loaded when
Rails is loaded. It will first require `lib/blorgh.rb` from the engine, then
`lib/blorgh/engine.rb`, which is the file that defines the major pieces of
functionality for the engine.

如上所述，`Gemfile`中列出的gem包，将会在Rails加载时加载。首先，加载`lib/blorgh.rb`，
然后，加载`lib/blorgh/engine.rb`(定义了engine的主要的功能)。

为了让应用程序可以访问engine的功能，需要将engine挂载到应用程序的`config/routes.rb`文件中: 

```ruby
mount Blorgh::Engine, at: "/blog"
```

This line will mount the engine at `/blog` in the application. Making it
accessible at `http://localhost:3000/blog` when the application runs with `rails
server`.

上述代码，将engine挂载到应用程序的`/blog`目录下。然后，运行`rails server`，并通过
`http://localhost:3000/blog`访问。

NOTE: Other engines, such as Devise, handle this a little differently by making
you specify custom helpers (such as `devise_for`) in the routes. These helpers
do exactly the same thing, mounting pieces of the engines's functionality at a
pre-defined path which may be customizable.

注意: 有一些engines，比如，Devise，挂载的方式稍微有些不同，在路由中使用了像`devise_for`这样的
自定义辅助方法)。这些辅助方法做的事情是类似的，将engine的部分功能挂载在一个可预定义的路仅中。

### Engine setup

The engine contains migrations for the `blorgh_articles` and `blorgh_comments`
table which need to be created in the application's database so that the
engine's models can query them correctly. To copy these migrations into the
application use this command:

engine中包含了`blorgh_articles`和`blorgh_comments`这样的表的迁移，这些迁移将在
在应用程序中运行，从而创建响应的表，使得engine的模型可以被正确的查询。将迁移
复制到应用程序中，可以使用如下的命令。

```bash
$ rake blorgh:install:migrations
```

If you have multiple engines that need migrations copied over, use
`railties:install:migrations` instead:

如果存在多个engine需要复制迁移，请使用`railties:install:migrations`: 

```bash
$ rake railties:install:migrations
```

This command, when run for the first time, will copy over all the migrations
from the engine. When run the next time, it will only copy over migrations that
haven't been copied over already. The first run for this command will output
something such as this:

该命令第一次运行时，将engine中所有迁移拷贝到应用程序中。第二次运行时，仅仅拷贝那些
没有拷贝过的。命令的第一次运行输出如下: 

```bash
Copied migration [timestamp_1]_create_blorgh_articles.rb from blorgh
Copied migration [timestamp_2]_create_blorgh_comments.rb from blorgh
```

The first timestamp (`[timestamp_1]`) will be the current time, and the second
timestamp (`[timestamp_2]`) will be the current time plus a second. The reason
for this is so that the migrations for the engine are run after any existing
migrations in the application.

第一个时间戳(`[timestamp_1]`)是当前时间，第二个时间戳(`[timestamp_2]`)是当前时间+1秒。
这样设计的原因是为了将engine的迁移在应用程序的迁移之后运行。

To run these migrations within the context of the application, simply run `rake
db:migrate`. When accessing the engine through `http://localhost:3000/blog`, the
articles will be empty. This is because the table created inside the application is
different from the one created within the engine. Go ahead, play around with the
newly mounted engine. You'll find that it's the same as when it was only an
engine.

在应用程序环境下运行这些迁移，可简单的运行`rake db:migrate`。然后，通过`http://localhost:3000/blog`
访问engine时，文章为空，这是因为，在应用程序中创建表和在engine中创建表是不同。

If you would like to run migrations only from one engine, you can do it by
specifying `SCOPE`:

如果，只想从一个engine中运行迁移，可通过`SCOPE`来指定环境。

```bash
rake db:migrate SCOPE=blorgh
```

This may be useful if you want to revert engine's migrations before removing it.
To revert all migrations from blorgh engine you can run code such as:

在移除engine之前，回滚engine的迁移时，非常的有用。为了移除blorgh engine的所有迁移，
可以输入如下的命令:

```bash
rake db:migrate SCOPE=blorgh VERSION=0
```

### Using a Class Provided by the Application

#### Using a Model Provided by the Application

When an engine is created, it may want to use specific classes from an
application to provide links between the pieces of the engine and the pieces of
the application. In the case of the `blorgh` engine, making articles and comments
have authors would make a lot of sense.

当engine创建时，可能需要使用应用程序中的特定类，从而提供engine和应用程序之间的链接。
例如，在`blorgh`的engine中，如何让文章和注释拥有作者。

A typical application might have a `User` class that would be used to represent
authors for an article or a comment. But there could be a case where the
application calls this class something different, such as `Person`. For this
reason, the engine should not hardcode associations specifically for a `User`
class.

典型的应用程序可能使用`User`来，用来表示文章或注释的作者。但有的应用中，可能使用`Person`。
因此，就不能在engine中使用硬编码的关联。

To keep it simple in this case, the application will have a class called `User`
that represents the users of the application (we'll get into making this
configurable further on). It can be generated using this command inside the
application:

```bash
rails g model user name:string
```

The `rake db:migrate` command needs to be run here to ensure that our
application has the `users` table for future use.

Also, to keep it simple, the articles form will have a new text field called
`author_name`, where users can elect to put their name. The engine will then
take this name and either create a new `User` object from it, or find one that
already has that name. The engine will then associate the article with the found or
created `User` object.

First, the `author_name` text field needs to be added to the
`app/views/blorgh/articles/_form.html.erb` partial inside the engine. This can be
added above the `title` field with this code:

```erb
<div class="field">
  <%= f.label :author_name %><br>
  <%= f.text_field :author_name %>
</div>
```

Next, we need to update our `Blorgh::ArticleController#article_params` method to
permit(许可) the new form parameter:

```ruby
def article_params
  # 如下代码的含义，:article的hash中必须带有:title, :text, :author_name三个参数
  params.require(:article).permit(:title, :text, :author_name)
end
```

The `Blorgh::Article` model should then have some code to convert the `author_name`
field into an actual `User` object and associate it as that article's `author`
before the article is saved. It will also need to have an `attr_accessor` set up
for this field, so that the setter and getter methods are defined for it.

`Blorgh::Article`模型将`author_name`域转换为实际的`User`对象，在文章保存前，关联文章的`author`。
这需要使用`attr_accessor`设置该域。

To do all this, you'll need to add the `attr_accessor` for `author_name`, the
association for the author and the `before_save` call into
`app/models/blorgh/article.rb`. The `author` association will be hard-coded to the
`User` class for the time being.

```ruby
attr_accessor :author_name
belongs_to :author, class_name: "User"

before_save :set_author

private
  def set_author
    self.author = User.find_or_create_by(name: author_name)
  end
```

By representing the `author` association's object with the `User` class, a link
is established between the engine and the application. There needs to be a way
of associating the records in the `blorgh_articles` table with the records in the
`users` table. Because the association is called `author`, there should be an
`author_id` column added to the `blorgh_articles` table.

通过使用带有`User`类的`author`关联对象，从而建立了engine与应用程序的关联。然后，
需要在`blorgh_articles`表和`users`表之间，建立关联，由于关联名为`author`，必须
在`blorgh_articles`表中，添加一个`author_id`列。

To generate this new column, run this command within the engine:

为了生成一个新的列，需要在engine中运行如下的命令: 

```bash
$ bin/rails g migration add_author_id_to_blorgh_articles author_id:integer
```

NOTE: Due to the migration's name and the column specification after it, Rails
will automatically know that you want to add a column to a specific table and
write that into the migration for you. You don't need to tell it any more than
this.

注意: Rails可从迁移名中推断出你的目的，为特定的表添加新列。所以，取个有意义的迁移名
很重要。

This migration will need to be run on the application. To do that, it must first
be copied using this command:

```bash
$ rake blorgh:install:migrations
```

Notice that only _one_ migration was copied over here. This is because the first
two migrations were copied over the first time this command was run.

```
NOTE Migration [timestamp]_create_blorgh_articles.rb from blorgh has been
skipped. Migration with the same name already exists. NOTE Migration
[timestamp]_create_blorgh_comments.rb from blorgh has been skipped. Migration
with the same name already exists. Copied migration
[timestamp]_add_author_id_to_blorgh_articles.rb from blorgh
```

Run the migration using:

```bash
$ rake db:migrate
```

Now with all the pieces in place, an action will take place that will associate
an author - represented by a record in the `users` table - with an article,
represented by the `blorgh_articles` table from the engine.

Finally, the author's name should be displayed on the article's page. Add this code
above the "Title" output inside `app/views/blorgh/articles/show.html.erb`:

```erb
<p>
  <b>Author:</b>
  <%= @article.author %>
</p>
```

By outputting `@article.author` using the `<%=` tag, the `to_s` method will be
called on the object. By default, this will look quite ugly:

使用`<%=`标签输出`@article.author`，会自动调用对象的`to_s`方法，这看起来非常的丑陋。

```
#<User:0x00000100ccb3b0>
```

This is undesirable. It would be much better to have the user's name there. To
do this, add a `to_s` method to the `User` class within the application:

```ruby
def to_s
  name
end
```

Now instead of the ugly Ruby object output, the author's name will be displayed.

#### Using a Controller Provided by the Application

Because Rails controllers generally share code for things like authentication
and accessing session variables, they inherit from `ApplicationController` by
default. Rails engines, however are scoped to run independently from the main
application, so each engine gets a scoped `ApplicationController`. This
namespace prevents code collisions, but often engine controllers need to access
methods in the main application's `ApplicationController`. An easy way to
provide this access is to change the engine's scoped `ApplicationController` to
inherit from the main application's `ApplicationController`. For our Blorgh
engine this would be done by changing
`app/controllers/blorgh/application_controller.rb` to look like:

Rails的控制器通常用来共享权限代码、以及访问session变量，这些默认从`ApplicationController`
中继承。而，Rails的engine使用了命名隔离的控制器，命名空间阻止了代码冲突，engine的控制器
经常需要访问应用程序的主控制器(`ApplicationController`)。简单的方法是，将继承的控制器修改
为`ApplicationController`。

```ruby
class Blorgh::ApplicationController < ApplicationController
end
```

By default, the engine's controllers inherit from
`Blorgh::ApplicationController`. So, after making this change they will have
access to the main application's `ApplicationController`, as though they were
part of the main application.

This change does require that the engine is run from a Rails application that
has an `ApplicationController`.

### Configuring an Engine

This section covers how to make the `User` class configurable, followed by
general configuration tips for the engine.

#### Setting Configuration Settings in the Application

The next step is to make the class that represents a `User` in the application
customizable for the engine. This is because that class may not always be
`User`, as previously explained. To make this setting customizable, the engine
will have a configuration setting called `author_class` that will be used to
specify which class represents users inside the application.

To define this configuration setting, you should use a `mattr_accessor` inside
the `Blorgh` module for the engine. Add this line to `lib/blorgh.rb` inside the
engine:

定义可配置的设置，需要使用`mattr_accessor`，在engine的`lib/blorgh.rb`中设置。

```ruby
mattr_accessor :author_class
```

This method works like its brothers, `attr_accessor` and `cattr_accessor`, but
provides a setter and getter method on the module with the specified name. To
use it, it must be referenced using `Blorgh.author_class`.

The next step is to switch the `Blorgh::Article` model over to this new setting.
Change the `belongs_to` association inside this model
(`app/models/blorgh/article.rb`) to this:

```ruby
belongs_to :author, class_name: Blorgh.author_class
```

The `set_author` method in the `Blorgh::Article` model should also use this class:

```ruby
self.author = Blorgh.author_class.constantize.find_or_create_by(name: author_name)
```

To save having to call `constantize` on the `author_class` result all the time,
you could instead just override the `author_class` getter method inside the
`Blorgh` module in the `lib/blorgh.rb` file to always call `constantize` on the
saved value before returning the result:

为了避免每次调用constantize，可使用如下的方法覆盖`author_class`的getter方法。

```ruby
def self.author_class
  @@author_class.constantize
end
```

This would then turn the above code for `set_author` into this:

```ruby
self.author = Blorgh.author_class.find_or_create_by(name: author_name)
```

Resulting in something a little shorter, and more implicit in its behavior. The
`author_class` method should always return a `Class` object.

Since we changed the `author_class` method to return a `Class` instead of a
`String`, we must also modify our `belongs_to` definition in the `Blorgh::Article`
model:

改变了`author_class`方法，返回为`Class`而不是`String`，所以，需要修改`Blorgh::Article`模型中
`belongs_to`的定义。

```ruby
belongs_to :author, class_name: Blorgh.author_class.to_s
```

To set this configuration setting within the application, an initializer should
be used. By using an initializer, the configuration will be set up before the
application starts and calls the engine's models, which may depend on this
configuration setting existing.

为了给应用程序设置配置选项，需要使用初始化器。使用初始化器是，配置将会在应用启动和
调用engine模型之前设置，启动器一般与engine同名。

Create a new initializer at `config/initializers/blorgh.rb` inside the
application where the `blorgh` engine is installed and put this content in it:

在安装了`blorgh`的应用程序中，创建初始化器文件`config/initializers/blorgh.rb`，并
在其中添加如下行:

```ruby
Blorgh.author_class = "User"
```

WARNING: It's very important here to use the `String` version of the class,
rather than the class itself. If you were to use the class, Rails would attempt
to load that class and then reference the related table. This could lead to
problems if the table wasn't already existing. Therefore, a `String` should be
used and then converted to a class using `constantize` in the engine later on.

警告: 使用类的`String`版本，而不是class本身。如果使用class，Rails会尝试加载类，
然后引用相关的表。如果表没有事前存在，就会会引起一个问题。所以，使用`String`，然后再
用`constantize`将其转换成类。

Go ahead and try to create a new article. You will see that it works exactly in the
same way as before, except this time the engine is using the configuration
setting in `config/initializers/blorgh.rb` to learn what the class is.

There are now no strict dependencies on what the class is, only what the API for
the class must be. The engine simply requires this class to define a
`find_or_create_by` method which returns an object of that class, to be
associated with an article when it's created. This object, of course, should have
some sort of identifier by which it can be referenced.

#### General Engine Configuration

Within an engine, there may come a time where you wish to use things such as
initializers, internationalization or other configuration options. The great
news is that these things are entirely possible, because a Rails engine shares
much the same functionality as a Rails application. In fact, a Rails
application's functionality is actually a superset of what is provided by
engines!

Engine中可以使用初始化器，国际化或者其他的配置选项。

If you wish to use an initializer - code that should run before the engine is
loaded - the place for it is the `config/initializers` folder. This directory's
functionality is explained in the [Initializers section](configuring.html#initializers) 
of the Configuring guide, and works precisely the same way as the `config/initializers` 
directory inside an application. The same thing goes if you want to use a standard initializer.

For locales, simply place the locale files in the `config/locales` directory,
just like you would in an application.

Testing an engine
-----------------

When an engine is generated, there is a smaller dummy application created inside
it at `test/dummy`. This application is used as a mounting point for the engine,
to make testing the engine extremely simple. You may extend this application by
generating controllers, models or views from within the directory, and then use
those to test your engine.

The `test` directory should be treated like a typical Rails testing environment,
allowing for unit, functional and integration tests.

### Functional Tests

A matter worth taking into consideration when writing functional tests is that
the tests are going to be running on an application - the `test/dummy`
application - rather than your engine. This is due to the setup of the testing
environment; an engine needs an application as a host for testing its main
functionality, especially controllers. This means that if you were to make a
typical `GET` to a controller in a controller's functional test like this:

```ruby
get :index
```

It may not function correctly. This is because the application doesn't know how
to route these requests to the engine unless you explicitly tell it **how**. To
do this, you must also pass the `:use_route` option as a parameter on these
requests:

```ruby
get :index, use_route: :blorgh
```

This tells the application that you still want to perform a `GET` request to the
`index` action of this controller, but you want to use the engine's route to get
there, rather than the application's one.

Another way to do this is to assign the `@routes` instance variable to `Engine.routes` in your test setup:

```ruby
setup do
  @routes = Engine.routes
end
```

This will also ensure url helpers for the engine will work as expected in your tests.

Improving engine functionality
------------------------------

This section explains how to add and/or override engine MVC functionality in the
main Rails application.

### Overriding Models and Controllers

Engine model and controller classes can be extended by open classing them in the
main Rails application (since model and controller classes are just Ruby classes
that inherit Rails specific functionality). Open classing an Engine class
redefines it for use in the main application. This is usually implemented by
using the decorator pattern.

For simple class modifications, use `Class#class_eval`. For complex class
modifications, consider using `ActiveSupport::Concern`.

#### A note on Decorators and Loading Code

Because these decorators are not referenced by your Rails application itself,
Rails' autoloading system will not kick in and load your decorators. This means
that you need to require them yourself.

Here is some sample code to do this:

```ruby
# lib/blorgh/engine.rb
module Blorgh
  class Engine < ::Rails::Engine
    isolate_namespace Blorgh

    config.to_prepare do
      Dir.glob(Rails.root + "app/decorators/**/*_decorator*.rb").each do |c|
        require_dependency(c)
      end
    end
  end
end
```

This doesn't apply to just Decorators, but anything that you add in an engine
that isn't referenced by your main application.

#### Implementing Decorator Pattern Using Class#class_eval

**Adding** `Article#time_since_created`:

```ruby
# MyApp/app/decorators/models/blorgh/article_decorator.rb

Blorgh::Article.class_eval do
  def time_since_created
    Time.current - created_at
  end
end
```

```ruby
# Blorgh/app/models/article.rb

class Article < ActiveRecord::Base
  has_many :comments
end
```


**Overriding** `Article#summary`:

```ruby
# MyApp/app/decorators/models/blorgh/article_decorator.rb

Blorgh::Article.class_eval do
  def summary
    "#{title} - #{truncate(text)}"
  end
end
```

```ruby
# Blorgh/app/models/article.rb

class Article < ActiveRecord::Base
  has_many :comments
  def summary
    "#{title}"
  end
end
```

#### Implementing Decorator Pattern Using ActiveSupport::Concern

Using `Class#class_eval` is great for simple adjustments, but for more complex
class modifications, you might want to consider using [`ActiveSupport::Concern`]
(http://edgeapi.rubyonrails.org/classes/ActiveSupport/Concern.html).
ActiveSupport::Concern manages load order of interlinked dependent modules and
classes at run time allowing you to significantly modularize your code.

**Adding** `Article#time_since_created` and **Overriding** `Article#summary`:

```ruby
# MyApp/app/models/blorgh/article.rb

class Blorgh::Article < ActiveRecord::Base
  include Blorgh::Concerns::Models::Article

  def time_since_created
    Time.current - created_at
  end

  def summary
    "#{title} - #{truncate(text)}"
  end
end
```

```ruby
# Blorgh/app/models/article.rb

class Article < ActiveRecord::Base
  include Blorgh::Concerns::Models::Article
end
```

```ruby
# Blorgh/lib/concerns/models/article

module Blorgh::Concerns::Models::Article
  extend ActiveSupport::Concern

  # 'included do' causes the included code to be evaluated in the
  # context where it is included (article.rb), rather than being
  # executed in the module's context (blorgh/concerns/models/article).
  included do
    attr_accessor :author_name
    belongs_to :author, class_name: "User"

    before_save :set_author

    private
      def set_author
        self.author = User.find_or_create_by(name: author_name)
      end
  end

  def summary
    "#{title}"
  end

  module ClassMethods
    def some_class_method
      'some class method string'
    end
  end
end
```

### Overriding Views

When Rails looks for a view to render, it will first look in the `app/views`
directory of the application. If it cannot find the view there, it will check in
the `app/views` directories of all engines that have this directory.

When the application is asked to render the view for `Blorgh::ArticlesController`'s
index action, it will first look for the path
`app/views/blorgh/articles/index.html.erb` within the application. If it cannot
find it, it will look inside the engine.

You can override this view in the application by simply creating a new file at
`app/views/blorgh/articles/index.html.erb`. Then you can completely change what
this view would normally output.

Try this now by creating a new file at `app/views/blorgh/articles/index.html.erb`
and put this content in it:

```html+erb
<h1>Articles</h1>
<%= link_to "New Article", new_article_path %>
<% @articles.each do |article| %>
  <h2><%= article.title %></h2>
  <small>By <%= article.author %></small>
  <%= simple_format(article.text) %>
  <hr>
<% end %>
```

### Routes

Routes inside an engine are isolated from the application by default. This is
done by the `isolate_namespace` call inside the `Engine` class. This essentially
means that the application and its engines can have identically named routes and
they will not clash.

Routes inside an engine are drawn on the `Engine` class within
`config/routes.rb`, like this:

```ruby
Blorgh::Engine.routes.draw do
  resources :articles
end
```

By having isolated routes such as this, if you wish to link to an area of an
engine from within an application, you will need to use the engine's routing
proxy method. Calls to normal routing methods such as `articles_path` may end up
going to undesired locations if both the application and the engine have such a
helper defined.

For instance, the following example would go to the application's `articles_path`
if that template was rendered from the application, or the engine's `articles_path`
if it was rendered from the engine:

```erb
<%= link_to "Blog articles", articles_path %>
```

To make this route always use the engine's `articles_path` routing helper method,
we must call the method on the routing proxy method that shares the same name as
the engine.

```erb
<%= link_to "Blog articles", blorgh.articles_path %>
```

If you wish to reference the application inside the engine in a similar way, use
the `main_app` helper:

```erb
<%= link_to "Home", main_app.root_path %>
```

If you were to use this inside an engine, it would **always** go to the
application's root. If you were to leave off the `main_app` "routing proxy"
method call, it could potentially go to the engine's or application's root,
depending on where it was called from.

If a template rendered from within an engine attempts to use one of the
application's routing helper methods, it may result in an undefined method call.
If you encounter such an issue, ensure that you're not attempting to call the
application's routing methods without the `main_app` prefix from within the
engine.

### Assets

Assets within an engine work in an identical way to a full application. Because
the engine class inherits from `Rails::Engine`, the application will know to
look up assets in the engine's 'app/assets' and 'lib/assets' directories.

Like all of the other components of an engine, the assets should be namespaced.
This means that if you have an asset called `style.css`, it should be placed at
`app/assets/stylesheets/[engine name]/style.css`, rather than
`app/assets/stylesheets/style.css`. If this asset isn't namespaced, there is a
possibility that the host application could have an asset named identically, in
which case the application's asset would take precedence and the engine's one
would be ignored.

Imagine that you did have an asset located at
`app/assets/stylesheets/blorgh/style.css` To include this asset inside an
application, just use `stylesheet_link_tag` and reference the asset as if it
were inside the engine:

```erb
<%= stylesheet_link_tag "blorgh/style.css" %>
```

You can also specify these assets as dependencies of other assets using Asset
Pipeline require statements in processed files:

```
/*
 *= require blorgh/style
*/
```

INFO. Remember that in order to use languages like Sass or CoffeeScript, you
should add the relevant library to your engine's `.gemspec`.

### Separate Assets & Precompiling

There are some situations where your engine's assets are not required by the
host application. For example, say that you've created an admin functionality
that only exists for your engine. In this case, the host application doesn't
need to require `admin.css` or `admin.js`. Only the gem's admin layout needs
these assets. It doesn't make sense for the host app to include
`"blorgh/admin.css"` in its stylesheets. In this situation, you should
explicitly define these assets for precompilation.  This tells sprockets to add
your engine assets when `rake assets:precompile` is triggered.

You can define assets for precompilation in `engine.rb`:

```ruby
initializer "blorgh.assets.precompile" do |app|
  app.config.assets.precompile += %w(admin.css admin.js)
end
```

For more information, read the [Asset Pipeline guide](asset_pipeline.html).

### Other Gem Dependencies

Gem dependencies inside an engine should be specified inside the `.gemspec` file
at the root of the engine. The reason is that the engine may be installed as a
gem. If dependencies were to be specified inside the `Gemfile`, these would not
be recognized by a traditional gem install and so they would not be installed,
causing the engine to malfunction.

To specify a dependency that should be installed with the engine during a
traditional `gem install`, specify it inside the `Gem::Specification` block
inside the `.gemspec` file in the engine:

```ruby
s.add_dependency "moo"
```

To specify a dependency that should only be installed as a development
dependency of the application, specify it like this:

```ruby
s.add_development_dependency "moo"
```

Both kinds of dependencies will be installed when `bundle install` is run inside
of the application. The development dependencies for the gem will only be used
when the tests for the engine are running.

Note that if you want to immediately require dependencies when the engine is
required, you should require them before the engine's initialization. For
example:

```ruby
require 'other_engine/engine'
require 'yet_another_engine/engine'

module MyEngine
  class Engine < ::Rails::Engine
  end
end
```
