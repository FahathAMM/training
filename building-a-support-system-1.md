# Building A Support System - Part 1

Here we try to develop a support system (help desk) in order to get familiar with the various tools provided by the Laravel framework.

Following is a high-level feature list of the application we plan to develop.

- Customers can open support tickets on any matter they need help from the support team.
- Customer can find and open the support tickets created by them using a support ticket reference to check the status and solutions or provide more details on the matter as replies to the support agent.
- Support agents can search and open support tickets to reply to them to help customers to solve their issues and close the tickets if required.

For a complete set of user stories covering the support system features refer to [Support System User Stories](./support-system-user-stories.md).

You will notice that I don't impose creating a user account for the customers in this system, which shows you that not all users should necessarily have to create accounts to use it. People bought your whole new product. They ran into an issue that needs quick help on fixing. When they come to your support line, you ask them to go through a lengthy process to get their answer. Not nice.

Let's go ahead and see how we can develop this system.

## Create a new Laravel project

First thing first, we need to create a new Laravel project. Now you know how:

```
cd ~/Sites
composer create-project laravel/laravel support
```

Open the created project in your editor and configure the database details as explained [here](../../getting-started-laravel-development.md). We will name our database `support` for this tutorial.

Once done, run bellow command to serve your application for local access:

```
php artisan serve
```

You can visit `http;//localhost:8000` using your web browser to confirm it runs.

![Default Laravel Landing Page](../../../../images/default-laravel-8-landing-page.png)


## Plan database

We will develop this application in an incremental way. That means we do not design the application at once. Instead, we will focus on one feature at a time. This will make it easy for us to focus on the framework features of Laravel related to the fraction (or feature) we develop.

### Identify user roles

First, Let's identify the users involved in this system. If you read the [User Stories](./support-system-user-stories.md) carefully, you can see there are two users mainly involved in this system.

 - Customer
 - Support Agent


 Do we need data of both of these user roles to be recorded in our system? Customers come to the support system to get some help on a particular issue they have with a product or service. Most of the time they don't like to go through the additional steps like registering and setting passwords to come to the point they can say their problem. Because of this registering their data in the system might not be required unless we need to manage a history of support queries from one customer. In this case, to keep things simple, let's not ask the customers to register. Because of this, we don't need to manage them as entities in our system at the moment. However, **Support Agents** need to register in the system we need to know which **Support Agent** is handling a particular ticket. So we have our first **Entity**.

 An **Entity** is represented by **Model** in an **MVC** application. So we need to create a model for our **Support Agent** entity. However, Laravel comes with a default model called **User** which you can find in `app/Models` folder in your newly created project. Since **Support Agent** is the only user who logs into the system at the moment let's use our **User** model to deal with the **Support Agent** entity.


### Identify other entities

Also, the most noticeable **Entity** we can identify reading the user stories is **Support Ticket**. The system should support managing **Support Tickets**. In short, let's call then **Tickets**. **Ticket** has its own properties and behaviors. Because of this, it becomes an entity.

There can be many other entities in the system. The **Ticket** entity alone is enough for us to get started. Try to identify other entities in the system using the user stories. We will be discussing a few of them in the later parts of this tutorial.

### Identify properties

**Ticket** should have some data recorded with them to identify few things.

- Name of the customer who wants support
- Email of him/her to contact back
- Phone number to contact in case support agent needs to clarify something further
- Problem description
- Reference ID to identify the ticket
- Status of the support ticket
- Time of the ticket created

In order to record these details, we need to same them in the database. For that, we need to create a **table** in our database. Since we are going to record multiple tickets in our table, let's call it **tickets** (plural).

Also, we need to decide the column names and data types for the above properties of a ticket. Try to find the data type based on the type of data you are going to store in each column of the table.

- **customer_name** : VARCHAR
- **email** : VARCHAR
- **phone** : VARCHAR
- **description** : TEXT
- **ref** : VARCHAR
- **status** : TINYINT
- **created_at** : TIMESTAMP

Keep the column names simple and short, but also descriptive. If you need to learn more about data types in MySQL, [this will help](https://www.w3schools.com/sql/sql_datatypes.asp).

### Create the database migrations

Always use migrations to make changes to your database schema. Read [Database: Migrations](https://laravel.com/docs/8.x/migrations#introduction) on official Laravel documentation to understand the migrations.

Generate a new migration to create the **tickets** table. Open your project folder in the terminal and run:

```
php artisan make:migration create_tickets_table
```

This will create a new file in `database/migrations` folder with the name that looks similar to `2021_06_15_124759_create_tickets_table.php`. Open it, and let's change the code of `up()` function to add columns to **tickets** table.

```php
/**
 * Run the migrations.
 *
 * @return void
 */
public function up()
{
    Schema::create('tickets', function (Blueprint $table) {
        $table->id();
        $table->string('customer_name');
        $table->string('email');
        $table->string('phone')->nullable();
        $table->text('description');
        $table->string('ref')->unique();
        $table->tinyInteger('status')->comment('0=new, 1=attended, 2=resolved, 3=cancelled');
        $table->timestamps();
    });
}
```

You can see the `$table->id()` was already there. This adds an `auto incremented` `primary key` to our tickets table.

Read [Available Column Types](https://laravel.com/docs/8.x/migrations#available-column-types) to understand the column types available in Laravel to be used with migrations.

Note that we have marked `phone` columns as `nullable()`. This means that the data for the `phone` filed is not required (optional). If a customer does not provide the phone number still support agents are able to communicate using email. Make sure to identify optional data mark them accordingly to avoid issues that can arise later due to the missing data.

Also, note that we are adding a comment to the `status` column to indicate the possible values for this column and their respective meanings. You may also use the `enum` data type in this case. But I'll stick to the `tiny int`.

Here, `nullable()`, `unique()` and `comment()` are column modifiers. Read [Column Modifiers](https://laravel.com/docs/8.x/migrations#column-modifiers) to understand more about them.

The last thing is `timestamps()`. This will add two columns to our table `created_at` and `updated_at` with the data type `TIMESTAMP`. We can use them to store the ticket created time and last updated time.

Once we run this database migration, it will create the **tickets** table in our database. Also if we roll back the migration it should reverse the change we made. This is very important and should not forget when you create database migrations. Make sure the `down()` function has the code required to reverse the migration.

```php
/**
 * Reverse the migrations.
 *
 * @return void
 */
public function down()
{
    Schema::dropIfExists('tickets');
}
```

Now, let's run our migration to create the table.

```
php artisan migrate
```

It will also run few additional migrations that were already there as it's the first time we run the migration on this project.

```
$ php artisan migrate
Migration table created successfully.
Migrating: 2014_10_12_000000_create_users_table
Migrated:  2014_10_12_000000_create_users_table (64.28ms)
Migrating: 2014_10_12_100000_create_password_resets_table
Migrated:  2014_10_12_100000_create_password_resets_table (49.93ms)
Migrating: 2019_08_19_000000_create_failed_jobs_table
Migrated:  2019_08_19_000000_create_failed_jobs_table (50.63ms)
Migrating: 2021_06_15_124759_create_tickets_table
Migrated:  2021_06_15_124759_create_tickets_table (23.70ms)
```

Note that Laravel has created `uesrs` table too. This would be useful in the future. We don't need to worry about the other tables at the moment.


## Models

You know that **Model** is the component of **MVC** that takes care of the storage and retrieval of the data. In other words, it is responsible for the communication with the database.

Let's create a **Model** for our `tickets` table. In Laravel a model represents a single item of its respective entity type. So the name of our model will be **Ticket**.

```
php artisan make:model Ticket
```

This creates a file with the name `Ticket.php` inside `app/Models` folder. If you open the file, you can see it contains a class called `Ticket`. By convention, this class will communicate with the `tickets` table in the database. And when I say **by convention**, notice the singular/plural connection between the model class name and the respective table name.

**Ticket** => **tickets**


## Controller

To process the tickets and their data, we need a **Controller**. Let's create a [Resource Controller](https://laravel.com/docs/8.x/controllers#resource-controllers) for managing the **Ticket** entities.

```
php artisan make:controller TicketController -r -m Ticket
```

> In case you need to see what other options available with this artisan command you may run: `php artisan help make:controller`

This creates `TicketController` inside the `app/Http/Controllers` folder. Open it and take a look. See what functions Laravel has created for us. These function are called **actions**.

- **index**: Used to show a list of items of the type of entity managed by the controller. In this case, it is **tickets**. Also, the same function would be used to search, filter, and sort the data in the list.
- **create**: Used to show a form to create a new item (Ticket).
- **store**: Used to handle form data submitted by the create form.
- **show**: Used to show a single item (Ticket).
- **edit**: Used to show a form to edit an existing item (Ticket).
- **update**: Used to handle form data submitted by the edit item form.
- **destroy**: Used to an existing item (Ticket).

## Routes

Routes define the connection between the URL we enter in the web browser to access our application and their business logic. In other words, they define which controller action is called to serve a particular request.

Let's go to the `routes/web.php` file. This file contains the routes related your web application. You will see there is one route available there:

```php
Route::get('/', function () {
    return view('welcome');
});
```

This simply loads the content of the `resources/views/welcome.blade.php` in the web browser when you enter the base URL `http://localhost:8000`. Notice you don't need a controller for this. Let's use this as the landing page (home page) of our support system.

Open `resources/views/welcome.blade.php` and add the following code.

```html
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <title>{{ config('app.name') }}</title>
        <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/css/bootstrap.min.css" integrity="sha384-Gn5384xqQ1aoWXA+058RXPxPg6fy4IWvTNh0E263XmFcJlSAwiGgFAW/dAiS6JXm" crossorigin="anonymous">
    </head>
    <body class="antialiased">
        <div class="text-center mt-5">
            <h1>Support System</h1>
        </div>
    </body>
</html>
```

Notice how we have dynamically added the page title to the blade template using the value for the `name` parameter in the `config.app.php` configuration file. You can use `config()` helper to [access configuration values](https://laravel.com/docs/8.x/helpers#method-config) from any of the files in `config` folder.

Our view has its first-ever custom content.

![Bare Landing Page](../../../../images/support-bare-landing-page.png)

We use [Bootstrap 4](https://getbootstrap.com/docs/4.0/getting-started/introduction/) to add some styling to our app.

In order to work with the resource controller we created a moment ago we need to add some routes to the `web.php` route file. Add this at the bottom of the file:

```php
Route::resource('/tickets', 'App\Http\Controllers\TicketController');
```

This adds required routes to access all actions in our resource controller. To verify this, run the following command in the terminal:

```
php artisan route:list
```

Which will give you the output:

```
+--------+-----------+-----------------------+-----------------+-----------------------------------------------+------------+
| Domain | Method    | URI                   | Name            | Action                                        | Middleware |
+--------+-----------+-----------------------+-----------------+-----------------------------------------------+------------+
|        | GET|HEAD  | /                     |                 | Closure                                       | web        |
|        | GET|HEAD  | api/user              |                 | Closure                                       | api        |
|        |           |                       |                 |                                               | auth:api   |
|        | GET|HEAD  | tickets               | tickets.index   | App\Http\Controllers\TicketController@index   | web        |
|        | POST      | tickets               | tickets.store   | App\Http\Controllers\TicketController@store   | web        |
|        | GET|HEAD  | tickets/create        | tickets.create  | App\Http\Controllers\TicketController@create  | web        |
|        | GET|HEAD  | tickets/{ticket}      | tickets.show    | App\Http\Controllers\TicketController@show    | web        |
|        | PUT|PATCH | tickets/{ticket}      | tickets.update  | App\Http\Controllers\TicketController@update  | web        |
|        | DELETE    | tickets/{ticket}      | tickets.destroy | App\Http\Controllers\TicketController@destroy | web        |
|        | GET|HEAD  | tickets/{ticket}/edit | tickets.edit    | App\Http\Controllers\TicketController@edit    | web        |
+--------+-----------+-----------------------+-----------------+-----------------------------------------------+------------+
```

Notice how it has added routes to each of `index`, `create`, `store`, `show`, `edit`, `update`, and `destroy` actions. Go ahead and try those URLs in your browser. You will see only an empty screen. That's because our **TicketController** actions are still empty.

Let's go ahead and write the required code to get what we want from each of them.

## Open a new support ticket

Once a customer comes to our support system, they should be immediately prompted for opening a new support ticket. Let's add a button to the home page to do this.

Add this code right below the `h1` tag of `resources/views/welcome.blade.php`:

```html
...
<h1>Support System</h1>
<div class="mt-5">
  <a href="{{ route('tickets.create') }}" class="btn btn-primary">Open New Ticket</a>
</div>
...
```

Here `route()` helper returns the URL when the route name is provided. Read more about [route helper](https://laravel.com/docs/8.x/helpers#method-route).

when you click on the button, it takes you to the `http://localhost:8080/tickets/create`. Now you know this is handled by the `create` action of **TicketController**. We need to make sure our `create` action shows a form for the customer to enter the data required to open a new ticket.

Open `app/Http/Controllers/TicketController.php` and add following line of code in body of `create()` function:

```php
public function create()
{
    return view('tickets.create');
}
```

As you may have already guessed, the `view()` helper function renders view file `resources/views/tickets/create.blade.php`. We don't have this file yet. Let's go create it. You will need to create the `tickets` folder first. One good way to organize the view files is to add them in a folder named with their respective entity collection. In this case `tickets`.

And we need to add the code for our form:

```html
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <title>{{ config('app.name') }}</title>
        <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/css/bootstrap.min.css" integrity="sha384-Gn5384xqQ1aoWXA+058RXPxPg6fy4IWvTNh0E263XmFcJlSAwiGgFAW/dAiS6JXm" crossorigin="anonymous">
    </head>
    <body class="antialiased">
        <div class="text-center mt-5">
            <h1>Open New Ticket</h1>
            <div class="m-5">

                <!-- FORM START -->
                <form class="" action="{{ route('tickets.store') }}" method="post">
                    <div class="row justify-content-center">
                        <div class="col-lg-6">

                            <div class="form-group row">
                                <div class="col-md-4 text-md-right">
                                    <label for="customer_name">Your Name</label>
                                </div>
                                <div class="col-md-8">
                                    <input type="text" name="customer_name" value="" class="form-control">
                                </div>
                            </div>

                            <div class="form-group row">
                                <div class="col-md-4 text-md-right">
                                    <label for="email">Email</label>
                                </div>
                                <div class="col-md-8">
                                    <input type="text" name="email" value="" class="form-control">
                                </div>
                            </div>

                            <div class="form-group row">
                                <div class="col-md-4 text-md-right">
                                    <label for="phone">Phone</label>
                                </div>
                                <div class="col-md-8">
                                    <input type="text" name="phone" value="" class="form-control">
                                </div>
                            </div>

                            <div class="form-group row">
                                <div class="col-md-4 text-md-right">
                                    <label for="description">Description</label>
                                </div>
                                <div class="col-md-8">
                                    <textarea name="description" class="form-control"></textarea>
                                </div>
                            </div>

                            <div class="row">
                                <div class="col-md-8 offset-md-4 text-md-right">
                                    <input type="submit" value="Submit" class="btn btn-success">
                                </div>
                            </div>
                        </div>
                    </div>
                </form>
                <!-- FORM END -->

            </div>
        </div>
    </body>
</html>

```

Now when you reload the browser, your form will look like this:

![Open Ticket Form](../../../../images/support-open-titcket-form.png)

We have added some **Bootstrap** classes to keep the elements organized.

If you check the code you can see that code related to our form is within the `<!-- FORM START -->` and `<!-- FORM END -->` comments. Comparing the HTML in this file to the `resources/views/welcome.blade.php` you will notice that everything outside the `<body>` tag is common to both of these files. It can be a tedious task to manage repeated code in every view file.

## Layouts

Common parts of the view files can be extracted and placed in the `Layouts` for easy management. Read [Layouts Using Template Inheritance](https://laravel.com/docs/8.x/blade#layouts-using-template-inheritance) on official Laravel documentation to know more about layouts.

Let's create a layout file at `resources/views/layouts/app.blade.php` and move the common parts there:

```html
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <title>{{ config('app.name') }}</title>
        <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/css/bootstrap.min.css" integrity="sha384-Gn5384xqQ1aoWXA+058RXPxPg6fy4IWvTNh0E263XmFcJlSAwiGgFAW/dAiS6JXm" crossorigin="anonymous">
    </head>
    <body class="antialiased">
        @yield('content')
    </body>
</html>
```

This is the main layout of our app and the `@yield('content')` is what is coming from the individual view files. In order to use this layout, we need to **extend** it from the other view files.

Now, `resources/views/welcome.blade.php` changes to:

```html
@extends('layouts.app')

@section('content')
<div class="text-center mt-5">
    <h1>Support System</h1>
    <div class="mt-5">
        <a href="{{ route('tickets.create') }}" class="btn btn-primary">Open New Ticket</a>
    </div>
</div>
@endsection
```

And, `resources/views/tickets/create.blade.php` changes to:

```html
@extends('layouts.app')

@section('content')
<div class="text-center mt-5">
    <h1>Open New Ticket</h1>
    <div class="m-5">

        <!-- FORM START -->
        <form class="" action="{{ route('tickets.store') }}" method="post">
            <div class="row justify-content-center">
                <div class="col-lg-6">

                    <div class="form-group row">
                        <div class="col-md-4 text-md-right">
                            <label for="customer_name">Your Name</label>
                        </div>
                        <div class="col-md-8">
                            <input type="text" name="customer_name" value="" class="form-control">
                        </div>
                    </div>

                    <div class="form-group row">
                        <div class="col-md-4 text-md-right">
                            <label for="email">Email</label>
                        </div>
                        <div class="col-md-8">
                            <input type="text" name="email" value="" class="form-control">
                        </div>
                    </div>

                    <div class="form-group row">
                        <div class="col-md-4 text-md-right">
                            <label for="phone">Phone</label>
                        </div>
                        <div class="col-md-8">
                            <input type="text" name="phone" value="" class="form-control">
                        </div>
                    </div>

                    <div class="form-group row">
                        <div class="col-md-4 text-md-right">
                            <label for="description">Description</label>
                        </div>
                        <div class="col-md-8">
                            <textarea name="description" class="form-control"></textarea>
                        </div>
                    </div>

                    <div class="row">
                        <div class="col-md-8 offset-md-4 text-md-right">
                            <input type="submit" value="Submit" class="btn btn-success">
                        </div>
                    </div>
                </div>
            </div>
        </form>
        <!-- FORM END -->

    </div>
</div>
@endsection
```

We will use the same layout for all other views of this application.


## Submit ticket details to the server

Coming back to our form. What happens when you fill the form and submit it? Give it a try, and you will get this:

![419 Error](../../../../images/support-419-error.png)

This error occurs because Laravel is expecting some other information with the form data to verify that it comes from a legitimate user. And that piece of information is called **CSRF** token. Read [CSRF Protection](https://laravel.com/docs/8.x/csrf) on official Laravel documentation to learn what is CSRF and why it is used.

With that knowledge in mind, let's go ahead and add a **CSRF** token to our form. Add right below the opening tag of your form:

```html
...
<!-- FORM START -->
<form class="" action="{{ route('tickets.store') }}" method="post">
    {{ csrf_field() }}

    <!-- FORM FIELDS -->
</form>
<!-- FORM END -->
```

This will add a hidden input with the value attribute set to the **CSRF** token. If you submit the form, you won't get the above error anymore.

One important thing to notice here is, we did not check for a **CSRF** token in our controller action. We are sure about this because our `store()` action is still empty. So, where did the application check for the **CSRF** token?

## Middleware

Laravel applications use a set of classes called **Middleware** to control the application flow in between the route and its respective controller action. If you check the `app/Http/Middleware` folder, you can see a number of **Middleware** classes. These are registered with the application flow in the `app/Http/Kernel.php` file. Open it and find the following code to see where it has registered the `VerifyCsrfToken` class, which is responsible for checking the **CSRF** token in requests.

```php
/**
 * The application's route middleware groups.
 *
 * @var array
 */
protected $middlewareGroups = [
    'web' => [
        \App\Http\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
        // \Illuminate\Session\Middleware\AuthenticateSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \App\Http\Middleware\VerifyCsrfToken::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],

    'api' => [
        'throttle:api',
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],
];
```

You can see many other middleware classes are also registered there for `web` and `api` routes. For now, we will focus only on the `VerifyCsrfToken`.

Read about the [Middleware](https://laravel.com/docs/8.x/middleware) on official Laravel documentation to learn more about them.


## Saving ticket details

As we mentioned earlier, inserting the new ticket details into the database would be handled by the `store()` method. Take look at the `store()` method.

```php
public function store(Request $request)
{
    //
}
```

Notice that `store()` method accepts a `$request` argument. Since Laravel internally calling this method it is passing an instance of the `Illuminate\Http\Request` class to the `store()` method. We can use this `$request` object to retrieve input data. This is called **Dependency Injection**. Read [Dependency Injection & Controllers](https://laravel.com/docs/8.x/controllers#dependency-injection-and-controllers) to understand it further.

Now when you submit your form. The data will be submitted to the server. In order to quickly see them, we can use the `dd()` helper.

```php
public function store(Request $request)
{
    dd($request->all());
}
```

And the output would be something similar to:

```
array:5 [▼
  "_token" => "jHzwGPXI2kUYiII0T28zQAGdNBJmhpXbTG9c8WaE"
  "customer_name" => "John Doe"
  "email" => "johndoe@gmail.com"
  "phone" => "+65 4563 5645"
  "description" => "My computer is not working."
]
```

Looks good! Now we need to save these to the database. You know that saving data is done through the **Model**. Let's create a new **Ticket** model to save our data.

```php
public function store(Request $request)
{
    $ticket = new Ticket();

    $ticket->customer_name = $request->input('customer_name');
    $ticket->email = $request->input('email');
    $ticket->phone = $request->input('phone');
    $ticket->description = $request->input('description');
    $ticket->ref = sha1(time());
    $ticket->status = 0;

    $ticket->save();
}
```

Save this and re-submit your form. It will add a new row to the `tickets` table in the database. Use your database management tool (Sequel Pro, phpMyAdmin) to verify it.

This does the work, but the user experience is not that good. Ideally, it should show the newly created ticket to the customer so that she can note down the reference number in case she needs to come back at a later time and check the ticket status. Also, it would be nice to show a message saying the ticket is successfully created.

Let's add it to the `store()` method.

```php
public function store(Request $request)
{
    $ticket = new Ticket();

    $ticket->customer_name = $request->input('customer_name');
    $ticket->email = $request->input('email');
    $ticket->phone = $request->input('phone');
    $ticket->description = $request->input('description');
    $ticket->ref = sha1(time());
    $ticket->status = 0;

    if ($ticket->save()) {
        return redirect(route('tickets.show', $ticket->id))
            ->with('success', 'Your ticket is created successfully. Please write down the reference number to check the ticket status later.')
    }

    return redirect()->back()->with('error', 'Oops! Could not create your ticket. Please try later.')
}
```

On success we redirect the browser to see the ticket details. Here we pass a flash message too. Read [Redirecting With Flashed Session Data](https://laravel.com/docs/8.x/responses#redirecting-with-flashed-session-data) to understand more about this.

Redirect destination should have a way to capture the flash message and show it. In our case let's add this to the `layout` so that it works for all the views in our application.

Open `resources/views/layouts/app.blade.php` and add the following code right above the `@yield('content')` line.

```html
...
<body class="antialiased">
    @if (session('success'))
        <div class="alert alert-success">
            {{ session('success') }}
        </div>
    @endif

    @if (session('error'))
        <div class="alert alert-danger">
            {{ session('error') }}
        </div>
    @endif

    @yield('content')
</body>
...
```

This will capture `success` and `error` flash messages. It should be enough for most of the cases.

## User input validation

Accepting user data as they are provided can lead to many issues. For example, an invalid email address provided by a user can render it impossible to contact him afterward to provide support. Same for phone number. Because of this, user inputs should be validated in order to reduce the errors in them. It is not possible to eliminate all issues in input data with validation, but it will help us overcome many problems that we might face due to not having any validation.

Please refer to [Validation](https://laravel.com/docs/8.x/validation) on Laravel official documentation to understand how Laravel does the user input validation.

With that knowledge, let's try to add some validations to the `store()` method of `TicketController`. Add following code at the beginning of the `store()` method:

```php
public function store(Request $request)
{
    $this->validate($request, [
        'name' => 'required|max:200',
        'email' => 'required|email',
        'description' => 'required',
    ]);

    // rest of the that was already there
}
```

The above code passes the `$request` object to the `validate()` method of the controller with a set of rules to validate the input against. If the validation fails, it redirects the user back to the form with the error messages. The code after the validation step is not executed at all.

The `name`, `email`, and `description` values are required. In addition, a maximum character limit is set for `name` attribute. Also, `email` is validated to be in valid format. Note that we do not validate the `phone` field as it is optional. But feel free to add more validation rules as you think that fits. Learn more about [built-in validation rules](https://laravel.com/docs/8.x/validation#available-validation-rules) of Laravel in official documentation.

The error messages are passed to view in the variable `$errors`, which is an instance of [Illuminate\Support\ViewErrorBag](https://laravel.com/api/8.x/Illuminate/Support/ViewErrorBag.html). We need to capture the error messages coming from the controller and show them on the form itself to help users correct them. Let's see how to sow error messages for each field on the form.

```html
<div class="form-group row">
    <div class="col-md-4 text-md-right">
        <label for="customer_name">Your Name</label>
    </div>
    <div class="col-md-8">
        <input type="text" name="customer_name" value="" class="form-control {{ $errors->has('name') ? 'is-invalid' : '' }}">
        @if($errors->has('name'))
        <div class="invalid-feedback text-left">
            {{ $errors->first('name') }}
        </div>
        @endif
    </div>
</div>
```

We use the `has()` method to check if the `name` field has any validation errors. If so, display it within a `<div>` with `invalid-feedback` css class. Also, to highlight the respective input element with a red border indicating that it has a validation error, the `is-invalid` css class is added. Those css classes are coming from Bootstrap. Learn more about  [styles for form validation](https://getbootstrap.com/docs/4.6/components/forms/#validation) in Bootstrap documentation.

**Exercise 1**: Add validation messages for the other fields.

**Exercise 2**: When there are multiple errors, display all of them at top of the form.

## View ticket details

Once the ticket is created customer should be able to see the ticket details. As we mentioned before, this is done using the `show()` action of the **TicketController**. Take a look at the `show()` method:

```php
public function show(Ticket $ticket)
{
    //
}
```

We know Laravel passes an instance of **Ticket** model as an argument to this function. But the question is which ticket does it load from the database to build the instance? In order to answer that we need to have an understanding of [Route Model Binding](https://laravel.com/docs/8.x/routing#route-model-binding).

With that knowledge, let's go ahead and add the required code to `show()`
 method to display the ticket details.

 ```php
 public function show(Ticket $ticket)
 {
    return view('tickets.show', [
        'ticket' => $ticket,
    ]);
 }
 ```

Also create the view file `resources/views/tickets/show.blade.php`.

```html
@extends('layouts.app')

@section('content')
<div class="text-center mt-5">
    <h1>Support Ticket</h1>
    <div class="m-5">

        <div class="row justify-content-center">
            <div class="col-lg-6 text-left">

                <table class="table table-bordered table-striped">
                    <tbody>
                        <tr>
                            <th>Ticket Refererence</th>
                            <td>{{ $ticket->ref }}</td>
                        </tr>
                        <tr>
                            <th>Customer Name</th>
                            <td>{{ $ticket->customer_name }}</td>
                        </tr>
                        <tr>
                            <th>Email</th>
                            <td>{{ $ticket->email }}</td>
                        </tr>
                        <tr>
                            <th>Phone</th>
                            <td>{{ $ticket->phone }}</td>
                        </tr>
                        <tr>
                            <th>Description</th>
                            <td>{{ $ticket->description }}</td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>

    </div>
</div>
@endsection
```

And, now it looks like:

![Ticket View](../../../../images/support-titket-view.png)

With that the minimum required work to open a support ticket on our online support system is over.

However, most of the support systems go a step further and send the customer an email about their support ticket. This way, even the customer forgets to note down the reference number, they have that information received in their email inbox so that they can find them easily when they want to check the status of their support request.

## Sending emails

Laravel has strong support for sending emails. Read [Laravel official documentation about mails](https://laravel.com/docs/8.x/mail#introduction) to understand how to configure and use the email features provided by Laravel.

Once you study that, now you know Laravel can send emails using an **SMTP** server. So we need one for our use. But sending emails to various different emails and checking them is not easy.

Because of this, we can use an **SMTP** email trap. Mailtrap is such a service. It gives you an **SMTP** server which you can use to send emails. Instead of sending the emails to actual email addresses, **Mailtrap** captures and shows them your own **Mailtrap** inbox. Visit `https://mailtrap.io` and create a new account. You also can use Google or Github accounts to sign up.

Once you are successfully registered, open your inbox to find the **SMTP** details for your account. Select **Laravel 7+** from the dropdown below the **Integrations** label. It will show you the configuration details you need to add to your `.env` file.

![Mailtrap SMTP Details](../../../../images/mailtrap-smtp-details.png)

Copy and paste them to your `.env` file replacing the existing email server configurations.

Ok, now you are ready for sending emails. Create a new mailable to send support ticket details to the customer.

```
php artisan make:mail TicketCreated
```

This creates a file named `TicketCreated.php` in the `app/Mail` directory. If you look at this file you would see there are 2 methods are there in the `TicketCreated` class. `__construct()` and `build()`. We use the constructor to initialize the instance variables and usual and `build()` method to build the output of the `TicketCreated` email.

To show ticket details we need a ticket. Let's add it as public property and initialize it with a **Ticket** object accepted through the constructor.

```php
namespace App\Mail;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Mail\Mailable;
use Illuminate\Queue\SerializesModels;
use App\Models\Ticket;

class TicketCreated extends Mailable
{
    use Queueable, SerializesModels;

    public $ticket;

    /**
     * Create a new message instance.
     *
     * @return void
     */
    public function __construct(Ticket $ticket)
    {
        $this->ticket = $ticket;
    }

    // rest of the code
}
```

Then create a view file to display the ticket details in `resources/views/mails/ticket-created.blade.php`.

```html
<p>
    Hi {{ $ticket->customer_name }},
</p>

<p>
    Thank you for contacting the support system.
    Your ticket is successfully created and details are shown below:
</p>

<br>

<table class="table table-bordered table-striped">
    <tbody>
        <tr>
            <td><strong>Ticket Refererence:</strong></td>
            <td>{{ $ticket->ref }}</td>
        </tr>
        <tr>
            <td><strong>Customer Name:</strong></td>
            <td>{{ $ticket->customer_name }}</td>
        </tr>
        <tr>
            <td><strong>Email:</strong></td>
            <td>{{ $ticket->email }}</td>
        </tr>
        <tr>
            <td><strong>Phone:</strong></td>
            <td>{{ $ticket->phone }}</td>
        </tr>
        <tr>
            <td><strong>Description:</strong></td>
            <td>{{ $ticket->description }}</td>
        </tr>
    </tbody>
</table>

<br><br>

<a href="{{ route('tickets.show', $ticket->id) }}">Click here to view your ticket</a>

<br>

<p>Thank you.</p>
```

Now let's go back to the `app/Mail/TicketCreated.php` and pass `$ticket` to the view.

```php
/**
 * Build the message.
 *
 * @return $this
 */
public function build()
{
    return $this->subject('New support ticket opened: ' . $this->ticket->ref)
        ->view('mails.ticket-created', [
            'ticket' => $this->ticket
        ]);
}
```

Note that we have added a subject to our email too. Read [Writing Mailables](https://laravel.com/docs/8.x/mail#writing-mailables) for more details on building up your email.

Once our **Mail** is ready to be sent, let's go back to the `store()` method of the **TicketController** to send it immediately after creating the ticket.

```php
namespace App\Http\Controllers;

use Mail;
use App\Models\Ticket;
use Illuminate\Http\Request;

class TicketController extends Controller
{
  ...

  public function store(Request $request)
  {
      ...

      if ($ticket->save()) {
          // Send the email to customer
          Mail::to($ticket->email)->send(new \App\Mail\TicketCreated($ticket));

          return redirect(route('tickets.show', $ticket->id))
              ->with('success', 'Your ticket is created successfully. Please write down the reference number to check the ticket status later.');
      }

      ...
  }

  ...
}
```

> Please note that the above sample does not show the entire code of the **TicketController**. Three dots `...` indicates the missing code from your controller.

Note how it uses the `Mail` facade to send a new email to a customer's email address by creating an object of **TicketCreated** mail. We pass the newly created ticket to the constructor of **TicketCreated** class, which will be used to build the email content.

Don't forget to add `use Mail;` at the top of the file right below the namespace declaration.

That's all we have to do to send an email containing the ticket details to the customer. Open a new ticket and go to the **Mailtrap** inbox to see your email.


## Check ticket status

Once after you open your support ticket, you wait for a reply from the support agents. However, there should be a way for you to go back to the support system and check what is ging on with your support ticket. This is where we need the reference number.

Let's build a way for customers to come back at a later time and check their ticket status using the reference number. Open the `resources/views/welcome.blade.php` file and add a form with an input box and a **View Ticket** button right below where the **Open New Ticket** button.

```php
@extends('layouts.app')

@section('content')
<div class="text-center mt-5">
    <h1>Support System</h1>
    <div class="mt-5">
        <a href="{{ route('tickets.create') }}" class="btn btn-primary">Open New Ticket</a>
    </div>
    <div class="mt-5">
        <p>
            Check the status of your ticket:
        </p>
        <div class="container">
            <form class="" action="{{ route('tickets.search') }}" method="get">
                <div class="row">
                    <div class="col-8">
                        <input type="text" name="reference" value="" class="form-control" placeholder="Enter ticket reference">
                    </div>
                    <div class="col-4">
                        <button type="submit" name="view" class="btn btn-success w-100">View Ticket</button>
                    </div>
                </div>
            </form>
        </div>
    </div>
</div>
@endsection
```

> Note that we use the **GET** method for submitting our form. Usually, the search forms are for getting (requesting) some data from the server. Due to this reason, we do not use the **POST** method. Also, the **GET** request adds the form parameters to **query string**, making it a unique resource identifier for the results we are getting. This can be helpful if you are sharing the search result with someone else. You can simply copy the browser URL after submitting the form and share it with anyone. This would be a good point to keep in mind when you are adding a search form to any other web application.

![Search Ticket](../../../../images/support-search-ticket.png)

Next, we need to add the `tickets.search` route to the `routes/web.php` file. Add the following line right before the resource routes of the **TicketController**

```php
Route::get('/tickets/search', 'App\Http\Controllers\TicketController@search')->name('tickets.search');
Route::resource('/tickets', 'App\Http\Controllers\TicketController');
```

The reason for adding this before the resource routes is to avoid **Laravel** match the path `/tickets/search` to the `/tickets/{ticket}` and assume the word `search` is an ID of a ticket. This is also a point you have to be careful about when defining your routes.

Now, let's add the `search()` method to the **TicketController**, which will take care of finding the ticket and redirect the user to ticket view. If the ticket is not found it redirects the user back to the landing page with a **not found** status message.

```php
public function search(Request $request)
{
    $ticket = Ticket::where('ref', $request->query('reference'))->first();

    if ($ticket) {
        return redirect(route('tickets.show', $ticket->id));
    }

    return redirect()->back()->with('error', 'Sorry! We could not find the ticket you are looking for. Please check the reference number.');
}
```

Now it shows the ticket when you enter a valid reference number. If not it shows something like this:

![Ticket Not Found](../../../../images/support-ticket-not-found.png)


The first part of this tutorial ends here. We will discuss how to develop the functionality for the **Support Agent** to manage and reply to the tickets in **Part 2**.

Happy coding!!
