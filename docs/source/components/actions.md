# Actions

Read the section in the  [Porto SAP Documentation (#Actions)](https://github.com/Mahmoudz/Porto#Actions).

## Rules

- All Actions **MUST** extend `App\Ship\Parents\Actions\Action`.

## Folder Structure

```
app
    Containers
        {container-name}
            Actions
                CreateUserAction.php
                DeleteUserAction.php
                ...
```

## Code Sample

Simplified Example from the `RegisterUserAction`

```php
<?php

namespace App\Containers\User\Actions;

class RegisterUserAction extends Action
{
    public function run(DataTransporter $data): User
    {
        // create user record in the database and return it.
        $user = Hive::call(CreateUserByCredentialsTask::class, [
            $isClient = true,
            $data->email,
            $data->password,
            $data->name,
        ]);

        Mail::send(new UserRegisteredMail($user));

        return $user;
    }
}
```

> Heads up!
> 
> Instead of passing these parameters `string $email, string $password, string $name, bool $isClient = false` from 
> place to another over and over, consider using the Transporters classes (simple DTOs "Data Transfer Objects"). For 
> more details read the [Transporters](./../components/transporters.html) page.

Injecting each Task in the constructor and then using it below through its property is really boring and the more 
Tasks you use the worse it gets. So instead you can use the function `call` to call whichever Task you want and pass 
any parameters to it.

The Action itself was also called using `Hive::call()` from the Controller, triggering `run()` function.

Refer to the [Magical Call](./../miscellaneous/magical-call.html) page for more info and examples on how to properly use the 
`call()` function.

## Examples

```php
<?php

namespace App\Containers\User\Actions;

use App\Containers\User\Tasks\DeleteUserTask;
use App\Ship\Parents\Actions\Action;

class DeleteUserAction extends Action
{
    public function run($userId)
    {
        return Hive::call(DeleteUserTask::class, [$userId]);
    }
}
```

```php
<?php

namespace App\Containers\Email\Actions;

use App\Containers\Xxx\Tasks\Sample1Task;
use App\Containers\Xxx\Tasks\Sample2Task;
use App\Ship\Parents\Actions\Action;

class DemoAction extends Action
{
    public function run($xxx, $yyy, $zzz)
    {
        $foo = Hive::call(Sample1Task::class, [$xxx, $yyy]);

        $bar = Hive::call(Sample2Task::class, [$zzz]);
    }
}
```

#### Calling an Action from a Controller

```php
<?php 

public function deleteUser(DeleteUserRequest $request)
{
    $user = Hive::call(DeleteUserAction::class, [$request->xxx, $request->yyy]);

    return $this->deleted($user);
}
```

The same Action **MAY** be called by multiple Controllers (`API`, `WEB` and `CLI`).
