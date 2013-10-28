---
layout: post
title: "Trialing laravel PHP"
date: 2013-10-25 01:29
comments: true
categories: [PHP, framework, Laravel]
published: false
---


Laravel was suggested to me last week and I have only just got time to take a look at it properly.

The framework is set out in a similar direction to Rails so it seems pretty familiar.

Initial install
---------------

Firstly I needed to install everything as my machine has recently decided to need rebuilding, I needed to install PHP and composer etc.

    sudo apt-get install php5
    curl -sS https://getcomposer.org/installer | php
    sudo mv composer.phar /usr/local/bin/composer

Setting up a new instance is then quick and simple with:

    composer create-project laravel/laravel laravel-test --prefer-dist

###Setup a new site in apache
I then setup an area to play around with in apache:

    sudo touch /etc/apache2/sites-available/laravel-test

I edited the file to have the following config:
{% codeblock laverel-test %}
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
Leaves a lot to be desired, not promoting new developers REST or at least explaining using REST would be more desired later.
Controllers are not even mentioned either.
There is not a detailed enough explanation to how to setup a web server to securely use the framework.

Templates
---------

The [Blade](http://laravel.com/docs/templates) templates remind me of [Erb](http://www.ruby-doc.org/stdlib-2.0.0/libdoc/erb/rdoc/ERB.html) and [Twig](http://twig.sensiolabs.org/) templates, so far they seem a little weird but certainly no [Template toolkit](http://www.template-toolkit.org).

I am however perplexed a little as to why Laravel has created another template engine when Twig appears to have the same goals and instead of using this:
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

I changed my database to PostgreSQL rather than mySQL opening app/config/database.php:

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

Forms look like they could do with a lot of work, they appear to just be a form output tool.
Having used formHandler on a regular basis recently the forms in Laravel appear to be far weaker than that.

Migrations
----------
For a corporate environment of any size, having the ability to roll back and forward database structures is very powerful.

Laravel provides migrate in the artisan command:
    php artisan migrate:make create_test_table

Here is a sample database I created:

    Schema::create('test', function($table)
    {
        $table->increments('id');
        $table->string('name');
        $table->text('description');
        $table->timestamps();
    });


Now lets run the migrations:
    php artisan migrate

Controller
----------

I then setup a sample controller:

    php artisan controller:make testController

Then setup the route to link to this controller making it a [resource controller](http://laravel.com/docs/controllers#resource-controllers):

    Route::resource('test', 'testController');

I then populated the controller code:

{% codeblock lang:php testController.php https://github.com/jonathanKingston/laravel-test/blob/master/app/controllers/testController.php link start:10 %}
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
For our controller we now need some templates:


{{ Form::open(array('url' => 'test')) }}
    {{ Form::label('email', 'E-Mail Address') }}
{{ Form::close() }}

ORM wrapper
-----------



Improvements
------------

Documentation, key parts of the framework are less documented that far less important parts.

Open questions
--------------
So far I have the current questions that I need to answer:

- Is there a way to uniformly handle different content types in the conroller.
- Are there built in validations for forms?
- Why do the Laravel code samples use spaces but the code in the framework use tabs?