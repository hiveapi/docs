# Transporters

Transporters is a name chosen by HiveApi for DTO's (Data Transfer Objects). The latter are used to pass user data 
(coming from `Requests`, `Commands`, or other components) from one place to another (`Actions` to `Tasks` / `Controller` 
to `Action` / `Command` to `Action` / ...).

They are very useful for reducing the number of parameters in functions, which prevents the duplication of long 
parameters.   

HiveApi relies on this [third-party package](https://github.com/fireproofsocks/dto) as DTO. Refer to the 
[dto package wiki](https://github.com/fireproofsocks/dto/wiki) for more details.

## Rules

- All Transporters **MUST** extend from `App\Ship\Parents\Transporters\Transporter`.

## Folder Structure

```
app
    Containers
        {container-name}
            Data
                Transporters
                    CreateUserTransporter.php
```

## Code Sample

```php
<?php

namespace App\Containers\Authentication\Transporters;

use App\Ship\Parents\Transporters\Transporter;

class ProxyApiLoginTransporter extends Transporter
{

    /**
     * @var array
     */
    protected $schema = [
        'properties' => [
            'email',
            'password',
            'client_id',
            'client_password',
            'grant_type',
            'scope',
        ],
        'required'   => [
            'email',
            'password',
            'client_id',
            'client_password',
        ],
        'default'    => [
            'scope' => '',
        ]
    ];
}
```

### Using a Transporter within a Controller

Normally you would use it like this

```php
<?php 
$dataTransporter = new DataTransporter($request);
$dataTransporter->bearerToken = $request->bearerToken();

Hive::call(ApiLogoutAction::class, [$dataTransporter]);
```

Since this example above has some required data, that data must be sent to the constructor:

```php
<?php
$dataTransporter = new ProxyApiLoginTransporter(
    array_merge($request->all(), [
        'client_id'       => Config::get('authentication-container.clients.web.admin.id'),
        'client_password' => Config::get('authentication-container.clients.web.admin.secret')
    ])
);

$result = Hive::call(ProxyApiLoginAction::class, [$dataTransporter]);
```

### Creating a Transporter for Tests

```php
<?php

$data = [
	'foo' => 'bar'
];

$transporter = new DataTransporter($data);
$action = App::make(RegisterUserAction::class);

$user = $action->run($transporter);
```

## Automatically Transforming a Request to a Transporter

If you want to directly transform a `Request` to a `Transporter` you can simply call

```php
$transporter = $request->toTransporter();
```

This method does take the `protected $transporter` of the `Request` class into account. If none is defined, a 
regular `DataTransporter` will be created.

Note, that `$transporter` will now have all fields from `$request` - so you can directly access them. In order to do so, 
you can call:

```php
<?php
// "simple" access via direct properties
$name = $transporter->name;

// complex access via method
$username = $transporter->getInputByKey('your.nested.username.field');
```

Of course, you can also "sanitize" the data, like you would have done in the `Request` classes by using 
`sanitizeData(array)`. Finally, if you need to access the original `Request` object, you can access it via

```php
$originalRequest = $transporter->request;
```

## Data Access

### Set Data

You can set data of a Transporter in many ways

```php
$dataTransporter = new DataTransporter($request);
$dataTransporter->bearerToken = $request->bearerToken();
```

If the data is defined as required like this on the Transporter:

```php
<?php
    protected $schema = [
        'type' => 'object',
        'properties' => [
            'email',
            'password',
            'clientId',
            'clientPassword',
        ],
        'required'   => [
            'email',
            'password',
            'clientId',
            'clientPassword',
        ],
    ];
```
 
Then can set data on the Transporter like this:
 
```php
$dataTransporter = new ProxyApiLoginTransporter(
    array_merge($request->all(), [
        'clientId'       => Config::get('authentication-container.clients.web.admin.id'),
        'clientPassword' => Config::get('authentication-container.clients.web.admin.secret')
    ])
);
```

### Get Data
 
To get all data from the Transporter you can call `$data->toArray()` or `$data->toJson()`. There are many other 
functions on the class. To get specific data just call the data name, as you would when accessing data from a Request 
object `$data->username`.

## Instance Access

### Set Instances

Passing Objects does not work, because the third-party package cannot hydrate it. In order to pass an instances from 
one place to another within a Transporter object, you can do the following:

```php
$transporter = new DataTransporter();
$transporter->setInstance("command_instance", $this);
```

> Heads up!
>
> Although you can set instances this way, they do not appear when calling `toArray()` or other similar functions, 
since they cannot be hydrated. See below how you can get the instance form the Transporter object.

### Get Instances

```php
$console = $data->command_instance;
```
 