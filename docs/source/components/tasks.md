# Tasks

Read from the [Porto SAP Documentation (#Tasks)](https://github.com/Mahmoudz/Porto#Tasks).

## Rules

- All Tasks **MUST** extend from `App\Ship\Parents\Tasks\Task`.

### Folder Structure

```
app
    Containers
        {container-name}
            Tasks
                ConfirmUserEmailTask.php
                GenerateEmailConfirmationUrlTask.php
                SendConfirmationEmailTask.php
                ValidateConfirmationCodeTask.php
                SetUserEmailTask.php
                ...
```

## Code Sample

Simplified example for `FindUserByIdTask`:

```php
<?php

namespace App\Containers\User\Tasks;

use App\Containers\User\Data\Repositories\UserRepository;
use App\Ship\Exceptions\NotFoundException;
use App\Ship\Parents\Tasks\Task;
use Exception;

class FindUserByIdTask extends Task
{
    private $userRepository;

    public function __construct(UserRepository $userRepository)
    {
        $this->userRepository = $userRepository;
    }

    public function run($id)
    {
        try {
            return $this->userRepository->find($id);
        } catch (Exception $e) {
            throw new NotFoundException();
        }
    }
}
```

### Calling the Task from an Action 

```php
<?php

namespace App\Containers\User\Actions;

use App\Containers\User\Models\User;
use App\Containers\User\Tasks\FindUserByIdTask;
use App\Ship\Exceptions\NotFoundException;
use App\Ship\Parents\Actions\Action;
use App\Ship\Transporters\DataTransporter;
use HiveApi\Core\Foundation\Facades\Hive;

class FindUserByIdAction extends Action
{
    public function run(DataTransporter $data): User
    {
        $user = Hive::call(FindUserByIdTask::class, [$data->id]);

        return $user;
    }
}
```

Note that the `Task` is called via the `Hive` Facade. All `Controllers`, `Actions`, `Tasks` and `SubActions`, however, 
also implement the `CallableTrait`, which allows you to directly `call()` another class via `$this->call(Classname::class)`.
