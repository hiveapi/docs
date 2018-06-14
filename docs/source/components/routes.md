# Routes 

Read from the [Porto SAP Documentation (#Routes)](https://github.com/Mahmoudz/Porto#Routes).

## Rules

- The API Routes files **MUST** be named according to their API version, exposure and functionality. Examples are 
`CreateOrder.v1.public.php`, `FulfillOrder.v2.public.php`, `CancelOrder.v1.private.php`...
- Web Routes files are pretty similar to API route files but they can be named anything.

## Folder Structure

```
app
    Containers
        {container-name}
            UI
                API
                    Routes
                        CreateItem.v1.public.php
                        DeleteItem.v1.public.php
                        CreateItem.v2.public.php
                        DeleteItem.v1.private.php
                        ApproveItem.v1.private.php
                WEB
                    Routes
                        main.php
                        ...
```

### API Routes

Example for the `User Login` API Endpoint

```php
<?php

$router->post('/login', [
    'uses' => 'Controller@loginUser',
]);
```

Example for a protected route to `List All Users` API Endpoint

```php
<?php

$router->get('users', [
    'uses'       => 'Controller@listAllUsers',
    'middleware' => [
        'api.auth', // use the authentication middleware to protect this endpoint!
    ]
]);
```

### Difference between Public & Private routes files

HiveApi has two different types of endpoints:
- `Public` (External) endpoints mainly provided for third parties clients, and 
- `Private` (Internal) endpoints for your own applications.

This will help generating separate documentations for each type and keep your internal API endpoints private.

## Web Routes

Example Endpoint to display a `Hello View` in the browser

```php
<?php

$router->get('/hello', [
    'uses' => 'Controller@sayHello',
]);
```

In all the Web `Routes` files the `$router` variable is an instance of the default Laravel Router `Illuminate\Routing\Router`.

## Protecting Endpoints:

Checkout the [Authorization](./../features/authorization) Page.
