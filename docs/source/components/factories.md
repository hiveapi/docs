# Factories

Factories (short name for Models Factories) are used to generate fake data with the help of `Faker` to be used for 
testing purposes. Factories are mainly used from Tests.

## Rules

- Factories **SHOULD** be created in the containers.
- A Factory is just a plain PHP script. There are no classes or namespaces required.

## Folder Structure

```
app
    Containers
        {container-name}
            Data
                Factories
                    UserFactory.php
                    ...
```

## Code Samples

```php
<?php

// User
$factory->define(App\Containers\User\Models\User::class, function (Faker\Generator $faker) {
    return [
        'name'     => $faker->name,
        'email'    => $faker->email,
    ];
});
```

### Calling the Factory from a Test Class

```php
<?php

// creating 4 users
factory(User::class, 4)->create();
```

### Example with Relationships

```php
<?php

$countries = Country::all();

// creating 3 rewards and attaching country relation to them
$rewards = factory(Reward::class, 3)->make()->each(function ($reward) use ($countries) {
    $reward->save();
    $reward->countries()->attach([$countries->random(1)->id, $countries->random(1)->id]);
    $reward->save();
});
```

Use make instance of `create()` and pass any data you want, then `save()` after establishing the relationship.

### Usage while overriding some values

```php
<?php

// creating single Offer and setting a user id
$offer = factory(Offer::class)->make();
$offer->user_id = $user->id;
$offer->save();

// ANOTHER EXAMPLE:
// creating multiple Accounts
$users = factory(Account::class, 3)->make()->each(function ($account) use ($user) {
    $account->user_id = $user->id;
    $account->save();
});
```

For more information about the Model Factories read the 
[official Laravel documentation](https://laravel.com/docs/master/testing#model-factories).
