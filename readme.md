<pre>
   __                           _             
  / /  __ _ _ __ __ ___   _____| |            
 / /  / _` | '__/ _` \ \ / / _ \ |            
/ /__| (_| | | | (_| |\ V /  __/ |            
\____/\__,_|_|  \__,_| \_/ \___|_|            
                                              
   ___                          _             
  / _ \___ _ __   ___ _ __ __ _| |_ ___  _ __ 
 / /_\/ _ \ '_ \ / _ \ '__/ _` | __/ _ \| '__|
/ /_\\  __/ | | |  __/ | | (_| | || (_) | |   
\____/\___|_| |_|\___|_|  \__,_|\__\___/|_|   
</pre>

**[View latest updates video](http://nettuts.s3.amazonaws.com/2089_laravel-generator/updates-to-laravel-generator-script.mov).**

On its own, when running migrations, Laravel simply creates the specified file, and adds a bit of boilerplate code. It's then up to you to fill in the Schema and such. ...Well that's a pain.

This generator task will fill in the gaps. It can generate several things:

- [Controllers](#controllers)
- [Models](#models)
- [Views](#views)
- [Migrations and Schema](#migrations)
- [Resources](#resources)
- [Assets](#assets)
- [Tests](#tests)

## Installation

Simply download the `generate.php` file from this repo, and save it to your `application/tasks/` directory. Now, we can execute the various methods from the command line, via Artisan.

### Controllers

To generate a controller for your project, run:

```bash
  php artisan generate:controller Admin
```

This will create a file, `application/controllers/admin.php`, with the following contents:

```php
<?php 

class Admin_Controller extends Base_Controller 
{

}
```

However, we can also create methods/actions as well.

```bash
  php artisan generate:controller Admin index show edit
```

The arguments that we specify after the controller name (Admin) represent the methods that we desire.

```php
<?php 

class Admin_Controller extends Base_Controller 
{

	public function action_index()
	{

	}

	public function action_show()
	{

	}

	public function action_edit()
	{

	}

}
```

But, what if we want to use restful methods? Well just add `restful` as any argument, like this:

```bash
php artisan generate:controller Admin restful index index:post
```

That will produce.

```php
<?php 

class Admin_Controller extends Base_Controller 
{

	public $restful = true;

	public function get_index()
	{

	}

	public function post_index()
	{

	}

}
```

Sweet! It's-a-nice. When using restful methods, you can specify the verb after a `:` - `index:post` or maybe `update:put`.

Lastly, if you need to create nested controllers, follow the period convention:

```bash
php artisan generate:controller admin.panel
```

This will produce "controllers/admin/panel.php" with the following contents:

```php
<?php

class Admin_Panel_Controller extends Base_Controller {

}
```

So that's the controller generator.


### Models

For now, models generation is very simplistic.

```bash
php artisan generate:model user
```

Will produce:

```php
<?php

class User extends Eloquent 
{

}
```


### Migrations

We can also generate migrations and schema. Let's say that we need to create a users table in the DB. Type:

```bash
php artisan generate:migration create_users_table
```

Notice that we separate each word with an underscore, and use a keyword, like `create`, `update`, `delete`, or `add`.

This will create a new file in `application/migrations/` that contains:

```php
<?php 
    
class Create_Users_Table
{

    public function up()
    {
        Schema::create('users', function($table)
        {
			$table->timestamps();
        });
    }

    public function down()
    {
        Schema::drop('users');
	}

}
```

Not bad, not bad. But what about the schema? Let's specify the fields for the table.

```bash
php artisan generate:migration create_users_table id:integer name:string email_address:string
```

That will give us:

```php
<?php 
    
class Create_Users_Table
{

    public function up()
    {
        Schema::create('users', function($table)
        {
			$table->increments('id');
			$table->string('name');
			$table->string('email_address');
			$table->timestamps();
        });
    }

    public function down()
    {
        Schema::drop('users');
	}

}
```

Ahh hell yeah. Notice that it automatically applies the `timestamps()`. A bit opinionated maybe, but it only takes a second to delete, if you don't want them.

You can also specify other options, such as making your `email` field unique, or the `age` field optional - like this:

```bash
php artisan generate:migration create_users_table name:string email_address:string:unique age:integer:nullable
```

Now we get:

```php
<?php 
    
class Create_Users_Table
{

    public function up()
    {
        Schema::create('users', function($table)
        {
			$table->increments('id');
			$table->string('name');
			$table->string('email_address')->unique();
			$table->integer('age')->nullable();
			$table->timestamps();
        });
    }

    public function down()
    {
        Schema::drop('users');
	}

}
```
Sweet!

#### Pivot Table Migrations

Pivot tables are used by Laravel to maintain many-to-many relationships between models. Eloquent prefers that the tables be created with a particular naming convention: "[singular model name]_[singular model name]", with the model names in alphabetical order joined by an underscore. The table name relating a User model to a Role model would be `role_user` (no caps). 

The structure of the tables also has a specific requirement that it reference the 'id' field of each of the linked models using the singular name of the model, so the table would contain a `user_id` field and a `role_id` field.

That's all of the actual requirements, and the Generator will make an attempt to build the file correctly even if you give it slightly incorrect parameters. All you have to do is make the first option after your migration name `pivot`. After that you can go ahead and sepcify additional columns just as you would a normal table. The following will all create the same table in the proper format even though you might not have gotten the request quite right:
```bash

    php artisan generate:migration create_users_roles_table pivot 
    php artisan generate:migration create_roles_users_table pivot
    php artisan generate:migration create_User_role_table pivot
    php artisan generate:migration create_users_roles_table pivot status:boolean 
```

Just make sure that the first option after migration instruction is `pivot`. You'll notice that the last command adds a boolean `status` field to the table. Fields can be added to the table just as you would for a non-pivot table as long as they follow the `pivot` option on the command line.

As usual the Generator is a bit opinionated about a few things, so we're not through… 

We add an auto-incrementing `id` field even though it's not strictly necessary; timestamps are expected in eloquent pivot tables by default, so we add those too; efficiency dictates that the linked `id` fields be indexed as foreign keys and that foreign key constraints be added to them, so we add those as well and set the `on_delete` and `on_update` constraints to `"cascade"` (note that if your database doesn't support foreign key constraints the indexes will be created but the constraints will be ignored). 

So now we get (from that last command):

```php
<?php 
    
class Create_Users_Organizations_Table {

	public function up()
    {
		/** @var \Laravel\Database\Schema\Table $table **/
		Schema::create('organization_user', function($table) {
			$table->increments('id');
			$table->boolean('status');
			$table->integer('organization_id')->unsigned();
			$table->integer('user_id')->unsigned();
			$table->timestamps();
			$table->foreign('organization_id')->references('id')->on('organizations')
			      ->on_delete('cascade')->on_update('cascade');
			$table->foreign('user_id')->references('id')->on('users')
			      ->on_delete('cascade')->on_update('cascade');
	});

    }

	public function down()
    {
		/** @var \Laravel\Database\Schema\Table $table **/
		Schema::table('organization_user', function($table) {
			$table->drop_foreign('organization_user_organization_id_foreign');
			$table->drop_foreign('organization_user_user_id_foreign');
	});

		/** @var \Laravel\Database\Schema\Table $table **/
		Schema::drop('organization_user');
    }
}
```

So the table has been properly named in the singular, even if the plural has been used to define it (thank you Laravel `Str` class) with the models in the proper alphabetical order, the related id keys are created as unsigned integers as required, and indexes with constraints are created. Everything is nicely dropped, including the indexes, when rolling back the migration.

Cool!

Of course, anything that you don't want can be easily deleted before you run your migration.


#### Keywords

It's important to remember that the script tries to figure out what you're trying to accomplish. So it's important that you use keywords, like "add" or "delete". Here's some examples:

```bash
php artisan generate:migration add_user_id_to_posts_table user_id:integer
php artisan generate:migration delete_user_id_from_posts_table user_id:integer
php artisan generate:migration create_posts_table title:string body:text
```

There's three best practices worth noting here:

1. Our class/file names are very readable. That's important.
2. The script uses the word that comes before "_table" as the table name. So, for `create_users_table`, it'll set the table name to `users`.
3. I've used the CRUD keywords to describe what I want to do.

If we run the second example:

```bash
php artisan generate:migration delete_user_id_from_posts_table user_id:integer
```

...we get:

```php
<?php 
    
class Delete_User_Id_From_Posts_Table
{

    public function up()
    {
        Schema::table('posts', function($table)
        {
			$table->drop_column('user_id');
        });
    }

    public function down()
    {
        Schema::table('posts', function($table)
        {
			$table->integer('user_id');
		});
	}

}
```

### Views

Views are a cinch to generate.

```bash
php artisan generate:view index show
```

This will create two files within the `views` folder:

1. index.blade.php
2. show.blade.php

You can also specify subdirectories, via the period.

```bash
php artisan generate:view home.index home.edit
```

Which will create:

1. home/index.blade.php
2. home/edit.blade.php

### Resources

But what if you want to save some time, and add a few things at once? Generating a resource will produce a 
controller, model, and view files. For example:

```bash
    php artisan generate:resource post index show
```

This will create:

1. A `post.php` model
2. A `posts.php` controller with index + show actions
3. A post views folder with `index.blade.php` and `show.blade.php` views.

### Putting It All Together

So let's rapidly generate a resource for blog posts.

```bash
php artisan generate:resource post index show
php artisan generate:migration create_posts_table id:integer title:string body:text
php artisan migrate 
```

Of course, if you haven't already installed the migrations table, you'd do that first: `php artisan migrate:install`.

With those three lines of code, you now have:

1. A controller with two actions
2. A post model
3. a post views folder with two views: index and show
4. A new `posts` table in your db with id, title, and body fields.

Nifty, ay?


### Assets

Just as a convenience, you can also generate stylesheets and scripts through the command line.

```bash
php artisan generate:assets style.css style2.css admin/script.js
```

This will create three files:

```bash
public/css/style.css
public/css/style2.css
public/js/admin/script.js
```

Note: that may seem like a lot to write for a simple task, but you can always create aliases for this stuff.

```bash
alias la="php artisan generate:assets";
```
Now, it's as easy as doing:

```bash
la style.css script.js
```

The default directory paths are:

- CSS => `public/css/`
- JavaScript => `public/js/`
- CoffeeScript => `public/js/coffee/`
- Sass (sass or scss extension) => `public/css/sass`


If the generator detects that you're attempting to create a script or stylesheet that has the same name as one of the popular libraries/frameworks, it will automatically download the latest version of the asset for you. For instance:

```bash
php artisan generate:assets main.js underscore.js
```

Will generate two files: `main.js` and `underscore.js`. However, it'll detect that you likely want the Underscore.js library and will download it for you.

A handful of external assets are provided by default, but you're free to add more. Just hunt down the `$external_assets` array within the generator script. 

```php
public static $external_assets = array(
    // JavaScripts
    'jquery.js' => 'http://code.jquery.com/jquery.js',
    'backbone.js' => 'http://backbonejs.org/backbone.js',
    'underscore.js' => 'http://underscorejs.org/underscore.js',
    'handlebars.js' => 'http://cloud.github.com/downloads/wycats/handlebars.js/handlebars-1.0.rc.1.js',

    // CSS
    'normalize.css' => 'https://raw.github.com/necolas/normalize.css/master/normalize.css'
);
```

### Tests

Laravel Generator can also generate PHPUnit test classes for you, like so:

```bash
php artisan generate:test user
```

If you only specify a class name, as we've done above, you'll get:

```php
<?php
// application/tests/user.test.php
class User_Test extends PHPUnit_Framework_TestCase 
{

}
```

However, you can also add test cases as well.

```bash
php artisan generate:test user can_reset_user_password can_disable_user
```

This command will now produce:

```php
<?php

class User_Test extends PHPUnit_Framework_TestCase {

    public function test_can_reset_user_password()
    {

    }

    public function test_can_disable_user()
    {

    }

}
```

## Tips and Tricks

#### Shorthand
In addition to using the full generator names, there are also some convenience versions.

- `generate:controller`    => `generate:c`
- `generate:model`         => `generate:m`
- `generate:migration`     => `generate:mig`
- `generate:view`          => `generate:v`
- `generate:resource`      => `generate:r`
- `generate:assets`        => `generate:a`

You could also just create Bash aliases, like this:

```bash
alias g:c="php artisan generate:controller";
alias g:m="php artisan generate:model"
```

Now, to create a controller, you can simply type:

```bash
g:c config index
````

[Refer here](https://github.com/JeffreyWay/Laravel-Generator/blob/master/aliases.md) for more alias recommendations.


#### Backbone

Let's say that you're working with Backbone, but haven't yet downloaded any of the scripts. Ideally, you'd be using a tool, like Yeoman or Bower, but, if not, do:

```bash
php artisan generate:assets jquery.js underscore.js backbone.js main.js
```

This will produce four files:

- js/jquery.js
- js/underscore.js
- js/backbone.js
- js/main.js

The first three files will be populated the latest version of their associated libraries/frameworks.