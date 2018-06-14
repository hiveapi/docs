# Controllers

Read from the [Porto SAP Documentation (#Controllers)](https://github.com/Mahmoudz/Porto#Controllers).

## Rules

- All `APIController` **MUST** extend from `App\Ship\Parents\Controllers\ApiController`.
- All `WebController` **MUST** extend from `App\Ship\Parents\Controllers\WebController`.
- Controllers **SHOULD** use the `call()` function to call an `Action`. Do not manually inject the Action and invoke 
the `run()` method.
- Controllers **SHOULD** pass the `Transporter` object to the `Action` instead of passing data from the request. The 
`Transporter` object can be derived from the `Request`. `Transporters` are the best classes to store the state of the 
`Request` during its lifecycle.

## Folder Structure

```
app
    Containers
        {container-name}
            UI
                API
                    Controllers
                        Controller.php
                WEB
                    Controllers
                        Controller.php
```

## Code Sample

`Controller` for the `User` Container (`WEB`)

```php
<?php

class Controller extends WebController
{
    public function sayWelcome()
    {
        return view('welcome');
    }
}
```

`Controller` for the `User` Container (`API`)

```php
<?php

class Controller extends ApiController
{
    public function registerUser(RegisterUserRequest $request)
    {
        $user = Hive::call(RegisterUserAction::class, [$request->toTransporter()]);

        return $this->transform($user, UserTransformer::class);
    }

    public function deleteUser(DeleteUserRequest $request)
    {
        $user = Hive::call(DeleteUserAction::class, [$request->toTransporter()]);

        return $this->deleted($user);
    }
}
```

Note that `Actions` are called by using `Hive::call()`, which automatically invokes the `run()` function in the `Action` 
as well inform the action which UI called it, (`$this->getUI()`) in case you wanna handle the same `Action` differently 
based on the UI type.

The second parameter of the `call()` function is an array of the `Action` parameters in order. When you need to pass 
data to the Action, it is recommended to pass the `Transporter` object as it should be the place that holds the state 
of your current request.

Refer to the [Magical Call](./../miscellaneous/magical-call) page for more info and examples on how to use the `call()` 
function.

### Calling a Controller function from a Route file

```php
<?php

$router->post('login', [
    'uses' => 'Controller@loginUser',
]);

$router->post('logout', [
    'uses'       => 'Controller@logoutUser',
    'middleware' => [
        'api.auth',
    ],
]);
```

## Controller Response Builder Helper Functions

Many helper function are there to help you build your `Response` faster, those helpers exist in the 
`vendor/hiveapi/core/src/Traits/ResponseTrait.php`.

#### Some functions

`transform()` is the most useful function, which you will be using in most cases.

- First required parameter accepts data as `object` or `Collection` of objects.
- Second required parameter is the `Transformer` class to be applied.
- Third (optional) parameter take the `includes` that should be returned by the response _($availableIncludes and 
$defaultIncludes in the `Transformer` class)_.  
- Fourth (optional) parameter accepts meta data to be injected in the `Response`.

```php
<?php 
// $user is a User Object
return $this->transform($user, UserTransformer::class);

// $orders is a Collection of Order Objects
return $this->transform($orders, OrderTransformer::class, ['products', 'recipients', 'store', 'invoice'], ['foo' => 'bar']);
```

`withMeta` allows including meta data in the response.

```php
<?php 
$metaData = ['total_credits', 10000];

return $this->withMeta($metaData)->transform($receipt, ReceiptTransformer::class);
```

`json` allows passing data (as array) to be represented as json.

```php
<?php 
return $this->json([
    'foo' => 'bar'
]);
```

**Other functions**

- accepted
- deleted
- noContent

Some functions might not be documented here, so refer to the `vendor/hiveapi/core/src/Traits/ResponseTrait.php` and check 
the public functions.
