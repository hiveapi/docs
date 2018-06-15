# Query Parameters: Search

Below you see how to setup a Search Query Parameter, on a Model: 

1. First, you need to define searchable fields on the Model Repository 

```php
<?php

namespace App\Containers\User\Data\Repositories;

class UserRepository extends Repository
{
    protected $fieldSearchable = [
        'name'  => 'like',
        'id'    => '=',
        'email' => '=',
    ];
}
```
	 
2. Next, you need to create a basic `List` and `Search Task`

```php
<?php

namespace App\Containers\User\Tasks;

class ListUsersTask extends Task
{
    private $userRepository;

    public function __construct(UserRepositoryInterface $userRepository)
    {
        $this->userRepository = $userRepository;
    }

    public function run()
    {
        return $this->userRepository->paginate();
    }
}
	 
```

3. Create an `Action` to call this Task

```php
<?php

namespace App\Containers\User\Actions;

class ListAndSearchUsersAction extends Action
{
    public function run()
    {
        return Hive::call(ListUsersTask::class, [], ['applyRequestCriteria']);
    }
} 
```

4. Use the Action from a Controller

```php
<?php

public function listAllUsers(Request $request)
{
    $users = Hive::call(ListAndSearchUsersAction::class);

    return $this->transform($users, UserTransformer::class);
} 
```

5. Call it from anywhere as follow `GET http://api.hive.local/users?search=example@test.com`
