# Building A Support System - Part 3

This is the third part of a series of tutorials guiding to develop an **Online Support System**.
- [Part 1](./building-a-support-system-1.md)
- [Part 2](./building-a-support-system-2.md)

In **Part 3** we focuses on developing an API for the support system.


## Requirement

The support system requires a mobile application to let users report issues and get help using their mobile phones. There is a separate team to develop the mobile application. You are required to provide a way for the mobile application to communicate with the support system.


## The API

The communication between two or multiple software is done using an API (Application Programming Interface). There are multiple **API** specifications and protocols.
- Remote Procedure Call (RPC)
- Service Object Access Protocol (SOAP)
- GraphQL
- Representational State Transfer (REST)

We focus on **REST** APIs in this tutorial. Please read the [REST API Basics](../../api/rest-api-basics.md) guide to understand the REST API basics and how they work.


There is a set of best practices that you should be aware of while developing REST APIs. Please read [REST API Best Practices](../../api/rest-api-best-practices.md) guide to learn them.



## Create Ticket API

With the knowledge of the REST API development, let's try to develop an API endpoint for creating new support tickets. Create a new controller for the `tickets` API requests.

```sh
php artisan make:controller API/V1/TicketsController
```

We select a separate namespace for the API controllers from the other controllers. That will help us manage our code better for API and regular web requests. The `V1` indicates the first version of the API. If we change the API endpoints later to accommodate new requirements, we can version them `V2`, `V3`, and so on.

Open the `app/Http/Controllers/API/V1/TicketController` file and add the `store()` method to save a new ticket.

```php
namespace App\Http\Controllers\API\V1;

use App\Http\Controllers\Controller;
use App\Models\Ticket;
use Illuminate\Http\Request;

class TicketController extends Controller
{
    $this->validate($request, [
        'customer_name' => 'required|max:200',
        'email' => 'required|email',
        'description' => 'required',
    ]);

    $data = $request->only([
        'customer_name',
        'email',
        'phone',
        'description',
    ]);

    $data['ref'] = sha1(time());
    $data['status'] = 0;

    $ticket = Ticket::create($data);

    if ($ticket) {
        // dispatch the TicketCreated event
        \App\Events\TicketCreated::dispatch($ticket);

        return response()->json([
            'data' => $ticket,
            'message' => 'Your ticket is created successfully. Please write down the reference number to check the ticket status later.'
        ]);
    }

    return response()->json([
        'data' => null,
        'message' => 'Oops! Could not create your ticket. Please try later.'
    ], 500);
}
```

You are familiar with the first few lines, that is for validation. Then we use a [different approach to get input data](https://laravel.com/docs/8.x/requests#retrieving-a-portion-of-the-input-data) from the request. Then we dispatch the **TicketCreated** event after creating the ticket. That will trigger the **SendNewTicketMail** listener as we discussed when we add events.

Lastly, it sends a **JSON** response to the client.

The newly created API should have a URL to access it. Similar to adding web routes to the `routes/web.php`, you should add the API routes to the `routes/api.php` file. Open it and add:

```php
Route::group([
    'namespace' => 'App\Http\Controllers\API\V1',
    'prefix' => 'v1',
], function() {
    Route::post('/tickets', 'TicketController@store');
});
```

Now our API endpoint is available at: `http://localhost:8000/api/v1/tickets`


## Testing the APIs

It's time to test our newly created API. There are many tools you can use to test your APIs during development. In this tutorial, we will learn two of them. The `curl` command-line tool and [Postman] graphical tool.

### Testing API with cURL

`curl` is a command-line tool available in both Mac and Linux. You can use it to send HTTP requests and display the output. Let's make a `POST` request to the `/tickets` API to create a new ticket.

```sh
curl -X POST -H "Accept: application/json" -H "Content-Type: application/json" -d '{"customer_name":"Brad Pitt","email":"brad@test.com","description":"My TV is not working"}' http://localhost:8000/api/v1/tickets
```

Note that we use the `POST` method, as this API is for adding new data to the `tickets` resource collection. The `-X` is used to mention the HTTP request method. The `-H` option is used to add request headers. We set the `Accept` header to `application/json` to indicate we expect the response in **JSON** format. Also, the `Content-Type` header indicates that the request body contains **JSON** data.

The output would look like:

```
{"data":{"customer_name":"Brad Pitt","email":"brad@test.com","description":"My TV is not working","ref":"262d2766620ec17d1da4f3e97abe36737dd6b796","status":0,"updated_at":"2021-08-04T10:09:03.000000Z","created_at":"2021-08-04T10:09:03.000000Z","id":6},"message":"Your ticket is created successfully. Please write down the reference number to check the ticket status later."}
```

You can see that the ticket is successfully created and the ticket details are available in the `data` parameter of the response. However, this output is not so human-readable. We can improve the readability by formatting the **JSON** output with the tool `json_pp`.

```sh
curl -X POST -H "Accept: application/json" -H "Content-Type: application/json" -d '{"customer_name":"Brad Pitt","email":"brad@test.com","description":"My TV is not working"}' http://localhost:8000/api/v1/tickets | json_pp -json_opt pretty         
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   464    0   374  100    90   6678   1607 --:--:-- --:--:-- --:--:--  8285
{
   "message" : "Your ticket is created successfully. Please write down the reference number to check the ticket status later.",
   "data" : {
      "created_at" : "2021-08-04T10:10:28.000000Z",
      "ref" : "5b530bb437eb73789bd60e12a75742586436213b",
      "customer_name" : "Brad Pitt",
      "id" : 8,
      "description" : "My TV is not working",
      "email" : "brad@test.com",
      "updated_at" : "2021-08-04T10:10:28.000000Z",
      "status" : 0
   }
}
```

Please refer to `man` page of the `curl` to get yourself familiar with using it.

```sh
man curl
```

Also, you can find more details on `json_pp` in the `man` page of it.

```sh
man json_pp
```


### Testing API with Postman

Postman is used by a large number of API developers worldwide to test their APIs. It can be used as a client, a mock server as well a test automator for APIs. Please refer to the following guides to get yourself familiar with it and learn how to use it for API testing.

- [Sending your first request](https://learning.postman.com/docs/getting-started/sending-the-first-request/)
- [Building requests](https://learning.postman.com/docs/sending-requests/requests/)
- [Receiving responses](https://learning.postman.com/docs/sending-requests/responses/)
- [Using variables](https://learning.postman.com/docs/sending-requests/variables/)

**Exercise 1**: Find out the HTTP status code of the response when validation fails.

**Exercise 2**: Develop an API endpoint to get the list of tickets (`GET /tickets`)

**Exercise 3**: Add search by reference feature to `GET /tickets` API.


### Writing test cases for APIs

The Postman and cURL are great tools when testing your API in development. But, in the long run, you need to have a better way of assuring the quality of your API. Doing regression testing on the existing APIs every time you add a new API or change something is time-consuming. The Laravel supports writing HTTP tests to help you in this.

Before you start, you need to set up your test environment. Please refer to  [Testting: Getting Started](https://laravel.com/docs/8.x/testing) guide to learn how to set up the environment and create new tests.

Before creating any tests, open the `tests` folder of your project and you will see few example tests. We can run those tests with the following command:

```sh
./vendor/bin/phpunit
```

The output will be similar to:

```sh
PHPUnit 9.5.5 by Sebastian Bergmann and contributors.

..                                                                  2 / 2 (100%)

Time: 00:00.257, Memory: 20.00 MB

OK (2 tests, 2 assertions)
```

Note that it indicates each test with a dot (`.`). The failed tests are indicated with letter `F` and errors are indicated with letter `E`. This time all your tests are passing, hence two dots. Also, it shows a summary at the end `OK (2 tests, 2 assertions)`.


Let's create a test for the `POST /tickets` API.

```sh
php artisan make:test API/V1/TicketsTest
```

This creates the `TicketsTest.php` file inside the `tests/Feature/API/V1` folder. Open it and you will see an example test inside:

```php
public function test_example()
{
    $response = $this->get('/');

    $response->assertStatus(200);
}
```

Now if you run `./vendor/bin/phpunit` again:

```sh
PHPUnit 9.5.5 by Sebastian Bergmann and contributors.

...                                                                 3 / 3 (100%)

Time: 00:00.266, Memory: 22.00 MB

OK (3 tests, 3 assertions)
```

You can see it runs our new test too. Great, let's change the example so that it really tests the `POST /tickets` API.

```php
public function test_create_new_ticket()
{
   $response = $this->withHeaders([
       'Accept' => 'application/json',
   ])->post('/api/v1/tickets', [
       'customer_name' => 'Jasper Jorden',
       'email' => 'jasper@gmail.com',
       'phone' => '+112233445566',
       'description' => 'There is something wrong with my computer',
   ]);

   $response->assertStatus(200);
}
```

Running `./vendor/bin/phpunit` again you would notice that your tests are still passing. Still we are asserting only the HTTP status code of the response. You can use more assertions to make your test stronger. Let's check for the response data to make sure it returns the newly created ticket. Add following line right below the `assertStatus()` function call.

```php
$response->assertJson([
   'data' => [
       'email' => 'jasper@gmail.com',
   ]
]);
```

This asserts that the response contains a JSON object with value of `data` property contains another object with the value of `email` property equal to `jasper@gmail.com`. This is one way you can verify the ticket is created with the input data you sent to the API.

Learn more assertions at [HTTP Tests](https://laravel.com/docs/8.x/http-tests) guide in Laravel official documentation.


**Exercise 4**: Write test cases for the `GET /tickets` API you developed in **Exercise 2** and **Exercise 3**.

**Exercise 5**: Develop APIs for the functions at support agent portal and write test cases for them.
