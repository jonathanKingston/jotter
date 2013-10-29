---
layout: post
title: "Trialling laravel PHP"
date: 2013-10-25 01:29
comments: true
categories: [PHP, framework, Laravel, First impressions]
---


Laravel was suggested to me last week and I have only just got time to take a look at it properly.

The framework has a similar structure to Rails so it seems pretty familiar to me so far. In making this article I created a [test REST laravel application](https://github.com/jonathanKingston/laravel-test).

Initial install
---------------

Firstly I needed to install everything as my machine has recently decided to need rebuilding, I needed to install PHP and composer etc.

    sudo apt-get install php5
    curl -sS https://getcomposer.org/installer | php
    sudo mv composer.phar /usr/local/bin/composer

Setting up a new instance is then quick and simple with:

    composer create-project laravel/laravel laravel-test --prefer-dist

###Setup a new site in Apache
I then setup an area to play around with in Apache:

    sudo touch /etc/apache2/sites-available/laravel-test

I then edited the file to have the following config:
{% codeblock laravel-test %}
Listen 8080

<VirtualHost *:8080>
        ServerAdmin webmaster@localhost

        DocumentRoot /home/jonathan/workspace/laravel-test/public
        <Directory /home/jonathan/workspace/laravel-test/public>
                RewriteEngine On
                Options Indexes FollowSymLinks MultiViews
                AllowOverride All
                Order allow,deny
                allow from all
        </Directory>


        ErrorLog ${APACHE_LOG_DIR}/error.log

        # Possible values include: debug, info, notice, warn, error, crit,
        # alert, emerg.
        LogLevel warn

        CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
{% endcodeblock %}

I then ran the following to enable the site:
    sudo a2ensite laravel-test;
    sudo a2enmod rewrite;
    sudo service apache2 reload;

I then ran the following to set the required permissions:
    sudo chmod -R 777 app/storage;


Quick start guide
-----------------
The laravel quick start guide I think could flow better.
The guide doesn't mention the benefits of separating out code into controllers.
New developers are not suggested that [REST resourceful routing](http://laravel.com/docs/controllers#resource-controllers) would be better later, I actually found this feature hard to find even though I was fairly sure the framework supported it.
There is not a detailed enough explanation to how to setup a web server to securely use the framework.

As a guide it doesn't seem like it would promote good development practice for new developers.

Templates
---------

The [Blade](http://laravel.com/docs/templates) templates remind me of [Erb](http://www.ruby-doc.org/stdlib-2.0.0/libdoc/erb/rdoc/ERB.html) and [Twig](http://twig.sensiolabs.org/) templates, so far they seem a little weird but certainly no [Template Toolkit](http://www.template-toolkit.org).

I am however a little perplexed as to why Laravel has created another template engine when Twig appears to have the same goals and instead of using this:
    @section('main')
      this is a test.
    @endsection

Twig would use:
    {% raw %}
    {% block main %}
       this is a test.
    {% endblock %}
    {% endraw %}

I personally find this easier to read, luckily it looks like you can use Twig in Laravel with [TwigBridge](https://github.com/rcrowe/TwigBridge).

Config changes
--------------

I changed my database to PostgreSQL rather than mySQL opening `app/config/database.php`:

            'default' => 'pgsql',

Also modifying the config further down for pgsql.

###New DB setup
Making sure we have the database installed:

    sudo apt-get install postgresql-9.1;
    sudo apt-get install php5-pgsql;

Changing the PostgreSQL default password if you have not already:
    sudo -u postgres psql postgres;

    \password postgres
    create database database;

Forms
-----

Forms look like they could do with a lot of work, as this is a modern framework I was expecting something more powerful than [Rails forms](http://guides.rubyonrails.org/form_helpers.html).
Having used [FormHandler](http://search.cpan.org/~gshank/HTML-FormHandler-0.40053/lib/HTML/FormHandler.pm) in Perl on a regular basis the forms in Laravel appear to be far weaker than that.
Anyway, we are left with something that is a PHP clone of Rails here, it does the job.

Migrations
----------
For a corporate environment of any size, having the ability to roll back and forward database structures is very powerful.

Laravel provides migrate in the artisan command:
    php artisan migrate:make create_test_table

Here is a sample database I created:
{% raw %}
    Schema::create('test', function($table)
    {
        $table->increments('id');
        $table->string('name');
        $table->text('description');
        $table->timestamps();
    });
{% endraw %}

Now lets run the migrations:
    php artisan migrate

Controller
----------

I then setup a sample controller:

    php artisan controller:make testController

Then setup the route to link to this controller making it a [resource controller](http://laravel.com/docs/controllers#resource-controllers):

    Route::resource('test', 'testController');

I then populated the controller code:

{% codeblock %}
{% raw %}
        public function index()
        {
                $tests = Test::all();
                return View::make('test/index', array('tests' => $tests));
        }
{% endraw %}
{% endcodeblock %}

Building the templates
----------------------
For the controller I then needed to build the following template s:

-    app/views/test/index.blade.php
-    app/views/test/edit.blade.php
-    app/views/test/show.blade.php
-    app/views/test/create.blade.php
-    app/views/test/destroy.blade.php

I also made the templates extend from a template 'layout':
{% codeblock %}
{% raw %}
<!DOCTYPE html>
<html>
  <head>
    <title>Test application</title>
  </head>
  <body>
    @yield('content')
  </body>
</html>
{% endraw %}
{% endcodeblock %}

Here is the edit action that I used:
{% codeblock %}
{% raw %}
@extends('layout')

@section('content')
  <h1>Edit test {{ $test->id }}</h1>

  {{ Form::model($test, array('action' => array('testController@update', $test->id), 'method' => 'put')) }}
    <div>{{ Form::label('name', 'Test name') }} {{ Form::text('name') }}</div>
    <div>{{ Form::label('description', 'Description') }} {{ Form::textarea('description') }}</div>
    {{ Form::submit() }}
  {{ Form::close() }}

  {{ link_to_action('testController@index', 'Show all') }}
@endsection
{% endraw %}
{% endcodeblock %}

I was happy to see several [Helpers](http://laravel.com/docs/helpers#urls) for producing links and URL's this is an absolute must for templates to keep links fresh and easy to maintain on a larger site.

[See all the rest of the template code I used.](https://github.com/jonathanKingston/laravel-test/tree/master/app/views/test)

ORM wrapper
-----------

The ORM wrapper seems easy enough to follow for beginners, following the same direction as most other frameworks.

However the Laravel docs shows how to use validations but not where they should be placed or what best practices are. I have decided to hook in with model so only one place defines the database validation.

I added the following code to my model;

-  This adds in a boot method which allows the model to define hooks.
-  I then use this for validating the model saving (Before updating or storing).
-  The validator is saved to the model so the controller can acess it.

{% codeblock %}
{% raw %}
  public $validator;

  public $rules = array(
    'name' => 'Required|Min:3|Alpha'
  );

  public static function boot() {
    parent::boot();

    //Add in saving hook to prevent database being saved
    Test::saving(function ($model) {
      $model->validator = Validator::make($model->toArray(), $model->rules);

      return $model->validator->passes();
    });
  }
{% endraw %}
{% endcodeblock %}

In the controller I then use:

Check out the [Validator API](http://laravel.com/docs/validation#error-messages-and-views) for more info.

{% codeblock %}
{% raw %}
$test = Test::find($id);
$test->update(array('name' => Input::get('name'), 'description' => Input::get('description')));

if ($test->validator->fails()) {
  return Redirect::action('testController@edit', array($test->id))->withInput()->withErrors($test->validator);
}
return Redirect::action('testController@show', array($test->id));
{% endraw %}
{% endcodeblock %}

I then modified the template to show the errors:
{% codeblock %}
{% raw %}
@if ( $errors->count() > 0 )
  <p>The following errors have occurred:</p>

  <ul>
  @foreach( $errors->all() as $message )
    <li>{{ $message }}</li>
  @endforeach
  </ul>
@endif
{% endraw %}
{% endcodeblock %}

The $errors will be populated from the redirect in the controller from `withErrors` and the forms will be populated with the erronous form input from the `withInput` method call too.

Improvements
------------

-   Documentation, key parts of the framework are less documented than far less important parts.
-   Add in the ability to do FormHandler style forms which can render forms, validate and connect to the model.
-   Stronger content-type response handling like [respond_with in Rails](http://apidock.com/rails/ActionController/MimeResponds/respond_with)
-   Why do the Laravel code samples use spaces but the code in the framework use tabs? (Pick one - please)
-   Scaffolding - yes it will make things look even more like Rails however it is a brilliant beginner tool and with the ability to customise the generated code for bigger projects it makes creating new wireframes is very fast.

Conclusion
----------

I am happy to see that the famework is fairly extendable, I didn't look into what the extension mechanisms are like and what popular extension code quality was.
I think overall it seems a far leaner framework than CakePHP and with most of the ability of Rails. I think being a youngish framework still, more could be done to improve the setup, documentation and tools.
