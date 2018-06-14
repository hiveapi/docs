# Magical Call

This _magical_ function allows you to `call()` any `Action` or `Task` `run()` function, from anywhere. Using the 
`Hive::call()` Facade.

The function `call()` is mainly used for calling HiveApi `Actions` from `Controllers`, and calling HiveApi `Tasks`from 
`Actions`.

Each `Action` knows which UI called it, using `$this->getUI()`. This may be useful for handling the same `Action` 
differently based on the `UI` type (`WEB` or `API`). This will work when calling the `Action` from `Controllers` and 
`Commands` using the magical `call()` function. 

## Usage

In the first argument you can pass the class full name, as follow `App\Containers\User\Tasks\CreateUserTask::class`, 
or you can pass the container name and class name, as follow `User@CreateUserTask`.

Using the "string based" style (i.e., `containerName@className`)  helps removing direct dependencies between containers. 
The `call()` function, in turn, will verify the Container exist before calling the function and inform the user to 
install Container if not exist.

A huge downside of the "string-based" approach, however, is that you will lose auto-completion features from your IDE!

When a class is directly called using its full name, a warning will be logged informing you to use the "string based 
caller style". This message, however, can be disabled by changing the flag `hive.logging.log-wrong-hive-caller-style` 
in the `Ship/Configs/hive.php` file accordingly. 

```php
<?php

// Call "AssignUserToRoleTask" Task from the "Authorization" Container using the hiveapi caller style 
Hive::call('Authorization@AssignUserToRoleTask');

// Call "AssignUserToRoleTask" Task from the "Authorization" Container using class full name.
// This will cause to add an INFO entry to the log file! 
Hive::call(\App\Containers\Authorization\Tasks\AssignUserToRoleTask::class);
```

### Example Basic Usage

```php
$foo = \HiveApi\Core\Foundation\Facades\Hive::call('Container@ActionOrTask');
```

- From `Controllers` and `Actions` you can use the `$this->call('Container@ActionOrTask')` instead of the Facade 
but it is not recommended.
- The magical `call()` function accepts the class full namespace (`\App\Containers\User\Tasks\GetAllUsersTask::class`) 
and the HiveApi caller style (`Containers@GetAllUsersTask`). 
- There is also a `transactionalCall()` method available, that wraps everything in a `DB::Transaction` (see below).

### Passing arguments to the `run()` function

```php
$foo = Hive::call('Container@ActionOrTask', [$runArgument1, $runArgument2, $runArgument3]);
```

### Calling other functions before calling the `run()`

```php
$foo = Hive::call('Container@ActionOrTask', [$runArgument], ['otherFunction1', 'otherFunction2']);
```

### Calling other functions and pass them arguments before calling the `run()`

```php
<?php
$foo = Hive::call('Container@ActionOrTask', [$runArgument], [
    [
       'function1' => ['function1-argument1', 'function1-argument2']
    ],
    [
       'function2' => ['function2-argument1']
    ],
]);

$foo = Hive::call('Container@ActionOrTask', [$runArgument], [
    'function-without-argument',
    [
      'function1' => ['function1-argument1', 'function1-argument2']
    ],  
]);

$foo = Hive::call('Container@ActionOrTask', [], [
    'function-without-argument',
    [
      'function1' => ['function1-argument1', 'function1-argument2']
    ],
    'another-function-without-argument',
    [
      'function2' => ['function2-argument1', 'function2-argument2', 'function2-argument3']
    ],
]);
```

## Transactional Magical Call

Sometimes, you want to wrap a call into one `Database Transaction` (see the  
[official Laravel Documentation](https://laravel.com/docs/5.6/database#database-transactions)).

Consider the following example: You want to create a new `Team` and automatically assign yourself (i.e., your own 
`User`) to this newly created `Team`. Your `CreateTeamAction` may call a dedicated `CreateTeamTask` and a 
`AssignMemberToTeamTask` afterwards. However, if the `AssignMemberToTeamTask` fails, for unknown reasons, you may want 
to "rollback" (i.e., remove) the newly created `Team` from the database in order to keep the database in a valid state.
  
That's where `DB::transactions` comes into play!
 
HiveApi provides a `transactionalCall($class, $params, $extraMethods)` method with the similar parameters as already 
known from the  `call()` method. Internally, this method calls this `call()` method anyways, but wraps it into a 
`DB::transaction`.
 
If any `Exception` occurs during the execution of the `$class` to be called, everything done in this context is 
automatically rolled-back from the database. However, respective operations on the file system (e.g., you may also 
have uploaded a profile picture for this `Team` already that needs to be removed in this case) need to be performed 
manually!
 
Typically, you may want to use the `transactionalCall()` on the `Controller` level!

## Use Case Example

```php
<?php

return Hive::call('User@ListUsersTask', [], ['ordered']);
// can be called this way as well Hive::call(ListUsersTask::class, [], ['ordered']);

return Hive::call('User@ListUsersTask', [], ['ordered', 'clients']);

return Hive::call('User@ListUsersTask', [], ['admins']);

return Hive::call('User@ListUsersTask', [], ['admins', ['roles' => ['manager', 'employee']]]);
```

### The ListUsersTask class

```php
<?php

namespace App\Containers\User\Tasks;

use App\Containers\User\Data\Criterias\AdminsCriteria;
use App\Containers\User\Data\Criterias\ClientsCriteria;
use App\Containers\User\Data\Criterias\RoleCriteria;
use App\Containers\User\Data\Repositories\UserRepository;
use App\Ship\Criterias\Eloquent\OrderByCreationDateDescendingCriteria;
use App\Ship\Parents\Tasks\Task;

class ListUsersTask extends Task
{
    private $userRepository;

    public function __construct(UserRepository $userRepository)
    {
        $this->userRepository = $userRepository;
    }

    public function run()
    {
        return $this->userRepository->paginate();
    }

    public function clients()
    {
        $this->userRepository->pushCriteria(new ClientsCriteria());
    }

    public function admins()
    {
        $this->userRepository->pushCriteria(new AdminsCriteria());
    }

    public function ordered()
    {
        $this->userRepository->pushCriteria(new OrderByCreationDateDescendingCriteria());
    }

    public function withRole($roles)
    {
        $this->userRepository->pushCriteria(new RoleCriteria($roles));
    }
}
```
