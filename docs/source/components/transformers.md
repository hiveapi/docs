# Transformers

Read from the [Porto SAP Documentation (#Transformers)](https://github.com/Mahmoudz/Porto#Transformers).

## Rules

- All API responses **MUST** be formatted via a `Transformer`.
- Every Transformer **SHOULD** extend from `App\Ship\Parents\Transformers\Transformer`.
- Each Transformer **MUST** have a `transform()` function.

## Folder Structure

```
app
    Containers
        {container-name}
            UI
                API
                    Transformers
                        UserTransformer.php
```

## Code Samples


```php
<?php

namespace App\Containers\Item\UI\API\Transformers;

use App\Containers\Item\Models\Item;
use App\Ship\Parents\Transformers\Transformer;

class ItemTransformer extends Transformer
{

    protected $availableIncludes = [
        'images',
    ];

    protected $defaultIncludes = [
        'roles',
    ];

    public function transform(Item $item)
    {
        $response = [
            'object'      => 'Item',
            'id'          => $item->getHashedKey(),
            'name'        => $item->name,
            'description' => $item->description,
            'price'       => $item->price,
            'weight'      => $item->weight,
            'created_at'  => $item->created_at,
            'updated_at'  => $item->updated_at,
        ];

        return $response;
    }

    public function includeImages(Item $item)
    {
        return $this->collection($item->images, new ItemImageTransformer());
    }

    public function includeRoles(User $user)
    {
        return $this->collection($user->roles, new RoleTransformer());
    }
}
```


### Using a Transformer to Return Data from a Controller

```php
<?php

    public function getAllClients(GetAllUsersRequest $request)
    {
        $users = Hive::call(GetAllClientsAction::class);

        return $this->transform($users, UserTransformer::class);
    }
```

<a name="relationships-include"></a>

## Relationships (Includes)

Loading relationships with the Transformer (calling other Transformers) can be done in 2 ways:

1. The `Client` can specify the relationships to be included via Query Parameters.
2. The `Developer` can define relationships to be automatically included.

### Apply Relationships via Query Parameters

The clients can request data with their relationships directly when calling the API by adding the `?include=x` query 
parameter. The Transformer, in turn, needs to have the `availableIncludes` defined with their functions like this:

```php
<?php

namespace App\Containers\Account\UI\API\Transformers;

use App\Ship\Parents\Transformers\Transformer;
use App\Containers\Account\Models\Account;
use App\Containers\Tag\Transformers\TagTransformer;
use App\Containers\User\Transformers\UserTransformer;

class AccountTransformer extends Transformer
{
    protected $availableIncludes = [
        'tags',
        'user',
    ];

    public function transform(Account $account)
    {
        return [
            'id'       => $account->id,
            'url'      => $account->url,
            'username' => $account->username,
            'secret'   => $account->secret,
            'note'     => $account->note,
        ];
    }

    public function includeTags(Account $account)
    {
        return $this->collection($account->tags, new TagTransformer());
    }

    public function includeUser(Account $account)
    {
        return $this->item($account->user, new UserTransformer());
    }
}
```

In order to get the `Tags` with the response when `Accounts` are requested, the clients needs to pass the 
`?include=tags` parameter with the `GET` request. To get Tags with User use the a comma separated list `?include=tags,user`.

### Apply Relationships from Application Code

From the controller you can dynamically set the `DefaultInclude`:

```php
<?php

    public function getAllClients(GetAllUsersRequest $request)
    {
        $users = Hive::call(GetAllClientsAction::class);

        return $this->transform($users, UserTransformer::class, ['tags', 'account']);
    }
```

You need to have `includeTags()` and `includeAccount()` functions defined on the transformer. If you want to include a 
relation with every response from this transformer you can define the relation directly in the transformer by adding it 
to the `$defaultIncludes`.

```php
<?php

    protected $availableIncludes = [
        'users',
    ];
    
    protected $defaultIncludes = [
        'tags',
    ];
    
    // ..
```

## Helper Functions for Transformers

- `user()` : returns the currently authenticated `User`.
- `ifAdmin($adminResponse, $clientResponse)` : merges normal client response with the admin extra or modified results, 
when current authenticated user is an `admin` user.

For more information about the Transformers read the 
[official package documentation](http://fractal.thephpleague.com/transformers/).
