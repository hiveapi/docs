# Requests

Read from the [**Porto SAP Documentation (#Requests)**](https://github.com/Mahmoudz/Porto#Requests).

## Rules

- All Requests **MUST** extend from `App\Ship\Parents\Requests\Request`.
- A Request **MUST** have a `rules()` function, returning an array and an `authorize()` function to check for 
authorization (can return `true` when no authorization is required).

## Folder Structure

```
app
    Containers
        {container-name}
            UI
                API
                    Requests
                        UpdateUserRequest.php
                        DeleteUserRequest.php
                        ...
                WEB
                    Requests
                        UpdateUserRequest.php
                        DeleteUserRequest.php
                        ...
```

## Code Samples

See an example for the `UpdateUserRequest` class:

```php
<?php

namespace App\Containers\User\UI\API\Requests;

use App\Ship\Parents\Requests\Request;

class UpdateUserRequest extends Request
{

    protected $access = [
        'permission' => '',
        'roles'      => 'admin',
    ];

    protected $decode = [
    ];

    protected $urlParameters = [
    ];

    public function rules()
    {
        return [
            'email'    => 'email|unique:users,email',
            'password' => 'min:100|max:200',
            'name'     => 'min:300|max:400',
        ];
    }

    public function authorize()
    {
        return $this->check([
            'hasAccess|isOwner',
        ]);
    }
}
```

The properties of the `Request` class will be discussed in the further course of this section.

## Using Requests in the Contorller

```php
<?php

public function updateUser(UpdateUserRequest $updateUserRequest)
{
    $data = $updateUserRequest->all();
    // or
    $name = $updateUserRequest->name;
    // or
    $name = $updateUserRequest['name'];
}
```

By just injecting the `Request` class you already applied the validation and authorization rules. When you need to 
pass data to the `Action`, you should pass the `Request` object as it is to the `Action` parameter. A more elaborate 
approach, however, would be to "morph" the `Request` to a `Transporter` class! 

## Request Properties

HiveApi adds some new properties to the existing `Request` class from Laravel. Each of these properties are very useful 
for some situations, and let you achieve your goals faster and cleaner. Below we will see a description for each property:

### decode

The `$decode` property is used for decoding hashed IDs from any Request on the fly if you have enabled the HashID 
feature provided by HiveApi. Most probably you are passing or allowing your users to pass hashed (encoded) IDs into 
your application in order to "hide" your true IDs. Thus these IDs needs to be decoded somewhere. HiveApi has a property 
on its `Request` component to specify those hashed IDs in order to decode them before applying the validation rules.

#### Example:

```php
<?php

namespace App\Containers\Authorization\UI\API\Requests;

use App\Ship\Parents\Requests\Request;

class AssignUserToRoleRequest extends Request
{

    protected $decode = [
        'id',
    ];
    
    // ...
}
```

> Heads up!
> 
> Validations rules that relies on your ID like (`exists:users,id`) will not work unless you decode your ID before 
passing it to the validation!

### urlParameters

The `$urlParameters` property is used for applying validation rules on the URL parameters. Laravel, by default, does 
not allow validating URL parameters (i.e., the `id` in `/stores/{id}/items`). In order to be able to apply validation 
rules on URL parameters you can simply define your URL parameters in the `$urlParameters` property. This will also 
allow you to access those parameters directly from the Controller in the same way you access the data from the `Request`.

#### Example:

```php
<?php

namespace App\Containers\Store\UI\API\Requests;

use App\Ship\Parents\Requests\Request;

class GetItemFromShopRequest extends Request
{

    /**
     * Defining the URL parameters (`/stores/{store_id}/items/{item_id}`) allows applying
     * validation rules on them and allows accessing them like request data.
     *
     * @var  array
     */
    protected $urlParameters = [
        'store_id',
        'item_id',
    ];

    public function rules()
    {
        return [
            'store_id' => 'required|integer', // url parameter
            'item_id'  => 'required|min:35|max:45', // url parameter
        ];
    }

    // ...
}
```

### access

The `$access` property, allows to define set of `Roles` and `Permissions` a client accessing the API **must** have in 
order to access this endpoint. The `$access` property is used by the `hasAccess` function defined below in the 
`authorize` function, to check if the user has the necessary `Roles` and `Permissions` to call this endpoint (i.e., access 
the controller function, where this `Request` object is injected).

#### Example:

```php
<?php

namespace App\Containers\User\UI\API\Requests;

use App\Ship\Parents\Requests\Request;

class DeleteUserRequest extends Request
{
    /**
     * Define which Roles and/or Permissions has access to this request.
     *
     * @var  array
     */
    protected $access = [
        'permission' => 'delete-users|another-permissions',
        'roles' => ['manager','admin']
    ];

    public function authorize()
    {
        return $this->check([
            'hasAccess|isOwner',
            'isKing',
        ]);
    }
}
```

If you do not like the `laravelish` style with `|` in order to separate the different `roles` or `permissions` (e.g., 
see the example above), you can also use the `array notation`.

## How the Authorize Function Works

The `authorize` function calls a `check` function, which accepts an array of functions names, each returning a boolean. 
In the example above, three functions (i.e., `hasAccess`, `isOwner`, and `isKing`) are called.

The separator `|` between the functions indicates an `OR` operation, so if any of the functions `hasAccess` or `isOwner` 
returns `true`, the user will gain access and only when both return `false` the user will be prevented from accessing 
this endpoint.

Furthermore, if `isKing` (i.e., a custom function that could be implemented by you) returns `false`, no matter what all 
other functions returns, the user will be prevented from accessing this endpoint, because the default operation between 
all functions in the array is `AND`.

### Add Custom Authorize Functions

The best way to add a custom authorize function is through a Trait, which can be added to your `Request` classes. In 
the example below we create a Trait named `isKingPermissionTrait` with a single method called `isKing`.

The `isKing()` method, in turn, calls a Task to verify that the current user is a king (e.g., if the user has the 
proper `Role` assigned).

```
<?php
namespace App\Containers\User\Traits;

use Apiato\Core\Foundation\Facades\Apiato;

trait isKingPermissionTrait
{
    public function isKing()
    {
        // The task needs to be implemented properly!
        return Apiato::call('User@CheckIfUserHasProperRoleTask', [$this->user(), ['king']]);
    }
}
```

Now, add the newly created Trait to the Request to use the `isKing` function in the authorization check.
```
<?php

namespace App\Containers\User\UI\API\Requests;

use App\Containers\User\Traits\isKingPermissionTrait;
use App\Ship\Parents\Requests\Request;

class FindUserByIdRequest extends Request
{
    use isKingPermissionTrait;

    // ...
    
    public function authorize()
    {
        return $this->check([
            'isKing',
        ]);
    }
}
```

Now, the Request uses the newly created `isKing` method to check the proper access rights.

## Allow a Role to access every endpoint

You can allow some `Roles` to access every endpoint in the system without having to define that role in each `Request` 
object. This is useful you want to let users with `Admin` role access everything.

To do this define those roles in `app/Ship/Configs/hive.php` as follow:

```php
'requests' => [
    'allow-roles-to-access-all-routes' => ['admin',],
],
```

This will append the `admin` role to all roles access in every `Request`.

## Request Helper Functions

HiveApi also provides some helpful functions by default, so you can use them whenever you need them.

### hasAccess

The `hasAccess` function, decides if the the user has access to this endpoint based on the `$access` property.

- If the user has any `Role` or `Permission` defined in the  access` property, he will be given access.
- If you need more or less roles/permissions just add `|` between each permission.
- If you do not need to set a roles/permissions just set `'permission' => ''` or  `'permission' => null`.

### isOwner

The `hasAccess` function, checks if the passed URL ID is the same as the User ID of the request.

#### Example:

Let's say we have an endpoint `api.example.develop/v1/users/{ID}/delete` that deletes a specified user. And we only 
need users to delete their own user accounts.

With `isOwner`, the user of ID 1 can only call `/users/1/delete` and won't be able to call `/users/2/delete` or any 
other ID. This also works with hashed IDs!

### getInputByKey

Use this method to get data from within the `$request` by entering the name of the field. This function behaves 
like `$request->input('key.here')`, however, it works on the `decoded` values instead of the original data.

Consider the following `Request` data in case you are passing `application/json` data instead of `x-www-form-urlencoded`:

```json
{
  "data" : {
    "name"  : "foo",
    "description" : "bar"
  },
  "id" : "a2423nadabada0"
}
```

Calling `$request->input('id')` would return `"a2423nadabada0"`, however `$request->getInputByKey('id')` would return the
decoded value (e.g., `4`).

Furthermore, one can define a `default` value to be returned, if the key is not present (or not set), like so:
`$request->getInputByKey('data.name', 'undefined name')`

### sanitizeData

Especially for `PATCH` requests, you like to submit only the fields, to be changed to the API in order to:

a) minimize the traffic
b) partially update the respective resource

Checking for the presence (or absence) of specific keys in the request typically results in huge `if` blocks, like so:

```php
<?php
// ...
if($request->has('data.name')) {
   $data['name'] = $request->input('data.name'); // or better use getInputByKey()
}
if($request->has('data.description')) {
   $data['description'] = $request->input('data.description'); // or better use getInputByKey()
}
// ...
```

To avoid those `if` blocks, use `array_filter($data)` in order to remove `empty` fields from the request. However, 
in PHP `false` and `''` _(empty string)_ are also considered as `empty` resulting in removing those fields from the 
request data (which is clearly not what you want).

You can read more about this problem [here](https://github.com/apiato/apiato/issues/186).

In order to simplify sanitizing your `Request Data` when using `application/json` instead of `x-www-form-urlencoded`, 
HiveApi offers a convenient `sanitizeInput(array $fields)` method.

Consider the following `Request` data:

```json
{
	"data" : {
		"is_private" : false,
		"description" : "this is a rather long description text",
		"a" : null,
		"b" : 3453,
		"foo" : {
			"a" : "a",
			"b" : "b",
			"c" : 1234
		},
		"bar" : [
		    "a", "b", "c"
		]
	}
}
```

The method lets you specify a list of `$fields` to be accessed and extracted from the `$request`. This is done by using 
the `DOT notation`. Finally, call the `sanitizeInput()` method on the `$request`:

```php
$fields = ['data.name', 'data.description', 'data.is_private', 'data.blabla', 'data.foo.c'];
$data = $request->sanitizeInput($fields);
```

The extracted `$data` looks like this:

```php
[
  "data" => [
    "is_private" => false
    "description" => "this is a rather long description text"
    "foo" => [
      "c" => 1234
    ]
  ]
]
```

Note that `data.blabla` is not within the `$data` array, as it was not present within the `$request`. Furthermore, all
other fields from the `$request` are omitted as they are not specified. So basically, the method creates some kind of
`filter` on the `$request`, only passing the defined values. Furthermore, the DOT notation allows you to easily specify
the fields to would like to pass through. This makes partially updating an resource quite easy!

> Heads Up:
> 
> Note that the `fillable fields` of an entity can be easily obtained with `$entity->getFillable()`!

### mapInput 

Sometimes you might want to map input from the request to other fields in order to automatically pass it to a `Action` 
or `Task`. Of course, you can manually map those fields, but you can also rely on the `mapInput(array $fields)` helper 
function.

This helper, in turn, allows to "redefine" keys in the request for subsequent processing. Consider the following 
example request:

```json
{
	"data" : {
		"name" : "John Doe"
	}
}
```

Your `Task` to process this data, however, requests the field `data.name` as `data.username`. You can call the the helper 
like this:

```php
$request->mapInput([
    'data.name' => 'data.username',
]);
```

The resulting structure would look like this:
```json
{
	"data" : {
		"username" : "John Doe"
	}
}
```

## Storing Data on the Request

During the `Request` lifecycle you may want to store some data on the request object and pass it to other `SubActions` 
(or maybe if you prefer to Tasks). To store (additional) data on the `Request` you may use:

```php
$request->keep(['someKey' => $someValue]);
```

To retrieve the data back at any time during the request lifecycle use:

```php
$someValue = $request->retrieve('someKey')
```
