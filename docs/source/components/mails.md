# Mails

The Mail component allows you to define an email and send it whenever needed. For more details refer to the 
[offiicial Laravel documentation](https://laravel.com/docs/mail).

## Principles

- Containers **MAY** or **MAY NOT** have one or more Mail.
- The Ship **MAY** contain general Mails.

## Rules

- All Notifications **MUST** extend from `App\Ship\Parents\Mails\Mail`.
- Email Templates must be placed inside the `Mails/Templates` directory within the container (i.e., 
`app/Containers/{container}/Mails/Templates`).

## Folder Structure

```
app
    Containers
        {container-name}
            Mails
                UserRegisteredMail.php
                Templates
                    user-registered.blade.php
    Ship
        Mails
            SomeMail.php
            Templates
                some-template.blade.php
```

## Code Samples

```php
<?php

namespace App\Containers\User\Mails;

use App\Containers\User\Models\User;
use Illuminate\Bus\Queueable;
use App\Ship\Parents\Mails\Mail;
use Illuminate\Contracts\Queue\ShouldQueue;

class UserRegisteredMail extends Mail implements ShouldQueue
{
    use Queueable;

    protected $user;

    public function __construct(User $user)
    {
        $this->user = $user;
    }

    public function build()
    {
        return $this->view('user::user-registered')
            ->to($this->user->email, $this->user->name)
            ->with([
                'name' => $this->user->name,
            ]);
    }
}
```

#### Sending the Mail from an Action

Notifications can be sent from `Actions` or `Tasks` using Laravels `Mail` Facade.

```php
Mail::send(new UserRegisteredMail($user));
```

## Email Templates

Templates should be placed inside the `Mails/Templates` folder. To access a Mail template (i.e., same as loading a view) 
you must call the container name then the view name.   

In the example below we are using the `user-registered.blade.php` template in the `User` Container.

```php
$this->view('user::user-registered')
```

## Configure Emails

Open the `.env` file and set the `FROM_MAIL_ADDRESS` and `MAIL_FROM_NAME` keys. These keys, in turn, will be used 
globally whenever the `from` function is not called in the Mail. 

```
MAIL_FROM_ADDRESS=support@example.com
MAIL_FROM_NAME="Support"
```

To use different email address in some classes, simply add `->to($this->email, $this->name)` to the `build` function in 
your Mail class. 

By default HiveApi is configured to use Log Driver `MAIL_DRIVER=log`, you can change that from the `.env` file.

## Queueing Notifications for Later Use 

To queue a notification you should use `Illuminate\Bus\Queueable` and implement `Illuminate\Contracts\Queue\ShouldQueue`.
