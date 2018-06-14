# Hash IDs

Hashing your internal IDs, is very helpful feature for securing your application (i.e., to prevent some kind of attacks) 
and business reasons (to hide the real total records from your competitors).

## Enable Hash ID

Set the `HASH_ID=true` in the `.env` file. Also with the feature make sure to always use the `getHashedKey()` on any 
model, whenever you need to return an ID (mainly within [Transformers](./../components/transformers.html)) whether 
hashed ID or not.

## Example:

```php
'id' => $user->getHashedKey(),
```

Note that if the feature is set to false (e.g., `HASH_ID=false`) the `getHashedKey()` will return the normal (unhashed) ID.

## Usage

There are 2 ways an ID's can be passed to your system via the API:

- Via URL Segment: (e.g., `www.hive.local/items/abcdef`).
- Via URL Query Parameters (e.g., `GET www.hive.local/items?id=abcdef`.

In both cases, however, you will need to inform your API about specific parts of that need to be decoded..

Checkout the [Requests](./../components/requests.html) page. After setting the `$decode` and `$urlParameters` properties on 
your `Request` class, the ID will be automatically decoded for you, to apply validation rules on it or/and use it from 
your controller (`$request->id` will now return the decoded ID).

## Configuration

You can change the default length and characters used in the ID from the config file `app/Ship/Configs/hashids.php` or 
in the `.env` file by editing the `HASH_ID_LENGTH` value.

You can set the `HASH_ID_KEY` in the `.env` file to any random string. You can generate this from any of the online 
random string generators, or run `head /dev/urandom | tr -dc 'A-Za-z0-9!"#$%&'\''()*+,-./:;<=>?@[\]^_{|}~' | head -c 32  ; echo` 
on the linux commandline. HiveApi defaults to the `APP_KEY` should this not be set.

The `HASH_ID_KEY` acts as the salt during the process of hashing IDs. This should never be changed in production as it 
renders all previously generated IDs useless!   

## Testing

In your tests you must hash the ID's before making the calls, because if you tell your Request class to decode an ID 
for you, it will throw an exception when the ID is not encoded.

### For Parameter ID's

Always use `getHashedKey()` on your models when you want to get the ID.

Example:

```php
$data = [
    'roles_ids' => [
        $role1->getHashedKey(),
        $role2->getHashedKey(),
    ],
    'user_id'   => $randomUser->getHashedKey(),
];
$response = $this->makeCall($data);
```

*Or you can do this manually `Hashids::encode($id);`. *

### For URL ID's

You can use this helper function `injectId($id, $skipEncoding = false, $replace = '{id}')`.

Example:

```php
$response = $this->injectId($admin->id)->makeCall();
```

More details on the [Tests Helpers]({{ site.baseurl }}{% link _docs/miscellaneous/tests-helpers.md %}) page.

## Availability

You can apply the `HiveApi\Core\Traits\HashIdTrait` to any `Model` or class, in order to have the `encode` and `decode` 
functions ready set up. By default you have access to these functions `$this->encode($id)` and  `$this->decode($id)` 
from all your Tests class and Controllers.
