# Criteria 

Criteria are classes used to hold and apply query condition when retrieving data from the database through a `Repository`. 
Without using a Criteria class, you can add your query conditions to a Repository or to a Model as scope. However, by 
using Criteria, your query conditions can be shared across multiple Models and Repositories. It allows you to define the 
query condition once and use it anywhere in the App.

## Principles

- Every Container **MAY** have its own Criteria. However, shared Criteria **SHOULD** be created in the `Ship` layer.
- A Criteria **MUST NOT** contain any extra code, if it needs data, the data **SHOULD** be passed to it from the `Action` 
or `Task`. It **SHOULD NOT** run (i.e., execute) any Task for data.

## Rules

- All Criteria **MUST** extend from `App\Ship\Parents\Criterias\Criteria`.
- Every Criteria **SHOULD** have an `apply()` function.
- A simple query condition example `"where user_id = $id"`, this can be named `ThisUserCriteria`, and used with all 
Models who has relations with the `User` Model.

## Folder Structure

```
app
    Containers
        {container-name}
            Data
                Criterias
                    ColourRedCriteria.php
                    RaceCarsCriteria.php
                    ...
    Ship
        Criterias
            Eloquent
                CreatedTodayCriteria.php
                NotNullCriteria.php
                ...
```

## Code Samples

Example for a shared Criteria (in the `Ship` layer)

```php
<?php

namespace App\Ship\Criterias\Eloquent;

use App\Ship\Parents\Criterias\Criteria;
use Prettus\Repository\Contracts\RepositoryInterface as PrettusRepositoryInterface;

class NotNullCriteria extends Criteria
{
    private $field;

    public function __construct($field)
    {
        $this->field = $field;
    }

    public function apply($model, PrettusRepositoryInterface $repository)
    {
        return $model->whereNotNull($this->field);
    }
}
```

#### Calling a Criteria from within a `Task`

```php
<?php

public function run()
{
    $this->userRepository->pushCriteria(new NotNullCriteria('email'));

    return $this->userRepository->paginate();
}
```

For more information about Criteria read the [official package documentation](https://github.com/andersao/l5-repository#create-a-criteria).
