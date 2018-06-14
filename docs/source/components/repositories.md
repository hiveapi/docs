# Repositories

The Repository classes are an implementation of the `Repository Design Pattern`. Their major roles are separating the 
business logic from the data (or the data access Task). Repositories save and retrieve `Models` to/from the underlying 
storage mechanism (i.e., databases).

The Repository is used to separate the logic that retrieves the data and maps it to a `Model`, from the business logic 
that works on the `Model`.

## Principles

- Every `Model` **SHOULD** have their own Repository.
- A `Model` **SHOULD** always get accessed through its Repository. You should never directly access the `Model`.

## Rules

- All Repositories **MUST** extend from `App\Ship\Parents\Repositories\Repository`. Extending from this class will give 
access to functions like `find()`, `create()`, `update()` and much more.
- The name of the Repository **SHOULD** be same like its `Model` name (Model: `Foo` -> Repository: `FooRepository`).
- If a Repository belongs to a Model whose name is not equal to its Container name, then the Repository must set the 
`$container` property manually like this: `$container = 'ContainerName'`.

## Folder Structure

```
app
    Containers
        {container-name}
            Data
                Repositories
                    UserRepository.php
```

## Code Samples

Example for the `UserRepository`

```php
<?php

namespace App\Containers\User\Data\Repositories;

use App\Containers\User\Contracts\UserRepositoryInterface;
use App\Containers\User\Models\User;
use App\Ship\Parents\Repositories\Repository;

class UserRepository extends Repository
{
    protected $fieldSearchable = [
        'name'  => 'like',
        'email' => '=',
    ];
}
```

### Using the Repository

```php
<?php

// paginate the data by 10
$users = $userRepository->paginate(10);

// search by 1 field
$cars = $carRepository->findByField('color', $color);

// searching multiple fields
$offer = $offerRepository->findWhere([
    'offer_id' => $offer_id,
    'user_id'  => $user_id,
])->first();
```

### Manually "linking" a Model and its Repository

If the Repository belongs to Model with a name different than its Container name, the Repository class of that Model 
must manually set the property `$container` and define the Container name.

```php
<?php

namespace App\Containers\Authorization\Data\Repositories;

use App\Ship\Parents\Repositories\Repository;

class RoleRepository extends Repository
{
    protected $container = 'Authorization'; // the container name. Must be set when the model has different name than the container

    protected $fieldSearchable = [
    ];
}
```

### Other Properties:

#### API Query Parameters Property

To enable query parameters (`?search=text`, ...) in your API you need to set the property `$fieldSearchable` on the 
Repository class, to instruct the querying on your model.

```php
<?php

	protected $fieldSearchable = [
	  'name'  => 'like',
	  'email' => '=',
	];
```

#### All Other Properties

HiveApi uses the `andersao/l5-repository` package, to provide a lot of powerful features to the repository class. To 
learn more about all the properties you can use, visit the `andersao/l5-repository` package 
[documentation](https://github.com/andersao/l5-repository).
