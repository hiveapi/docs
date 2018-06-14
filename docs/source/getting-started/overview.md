# Overview 

## Quickstart

When a HTTP request is received, it first hits your predefined Endpoint (each endpoint has its own Route file).

### Sample Route Endpoint

```php
<?php

$router->get('/hello', [
    'uses' => 'Controller@sayHello',
]);
```

After the user makes a request to the endpoint `[GET] api.hive.develop/v1/hello` it calls the function (`sayHello()`) 
in the respective `Controller` class.

### Sample Controller Function

```php
<?php

class Controller extends ApiController
{
	public function sayHello(SayHelloRequest $request)
	{
            $helloMessage = Hive::call(SayHelloAction::class);

            $this->json([
                $helloMessage
            ]);
	}
}
```

This `sayHello()` function takes a `Request` class `SayHelloRequest` and automatically checks, if the user has the 
proper role (or permission) to call this endpoint. An `Exception` is immediately thrown, if the user does not have 
the proper access level. Otherwise, the actual function is executed.

In this context, the function calls an `Action` (`SayHelloAction`) to perform the actual business logic.

### Sample Action

```php
<?php 

class SayHelloAction extends Action
{
	public function run()
	{
	    return 'Hello World!';
	}
}
```

An `Action` can do anything then (maybe) return a result. When the `Action` finishes its execution, the `Controller` 
function gets ready to build a `Response` and return this to the client that called the endpoint.

`Json` responses can be built using the helper function `json()` (`$this->json(['foo' => 'bar']);`).

### Sample User Response

```json
[
    "Hello World!"
]
```
