# Notifications

Notifications allows you to inform the client about a state changes in your application. Laravel supports sending such 
notifications across a variety of channels such as mail, SMS, Slack, Databases, ... 

When using the Database channel the notifications will be stored in a database to be displayed in your client interface.

For more details refer to the [official Laravel documentation](https://laravel.com/docs/notifications).

## Principles

- Containers **MAY** or **MAY NOT** have one or more Notification.
- The Ship layer **MAY** contain Application general Notifications.

## Rules

- All Notifications **MUST** extend from `App\Ship\Parents\Notifications\Notification`.

## Folder Structure

```
app
    Containers
        {container-name}
            Notifications
                UserRegisteredNotification.php
                ...
    Ship
        Notifications
            SystemFailureNotification.php
            ...
```

## Code Samples

Example for a simple Notification

```php
<?php

namespace App\Containers\User\Notifications;

use App\Containers\User\Models\User;
use App\Ship\Parents\Notifications\Notification;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;

class BirthdayReminderNotification extends Notification implements ShouldQueue
{
    use Queueable;

    protected $notificationMessage;

    public function __construct($notificationMessage)
    {
        $this->notificationMessage = $notificationMessage;
    }
    
    public function toArray($notifiable)
    {
        return [
            'content' => $this->notificationMessage,
        ];
    }

    public function toMail($notifiable)
    {
        // $notifiable is the object you want to notify "e.g. user"
        return (new MailMessage)
            ->subject("Happy Birthday")
            ->line("Hi, $notifiable->name")
            ->line($this->notificationMessage);
    }

    public function toSms($notifiable)
    {
        // ...
    }
    
    // ...
}
```

### Using a Notification within an Action or Task

Notifications can be sent from `Actions` or `Tasks` using the `Notification` Facade.  

```php
\Notification::send($user, new BirthdayReminderNotification($notificationMessage));
```

Alternatively, you can use the `Illuminate\Notifications\Notifiable` trait on the notifiable object (e.g., the `User`) 
and then call it as follow:

```php
// call notify, found on the Notifiable trait
$user->notify(new BirthdayReminderNotification($notificationMessage));
```

## Select Channels

To select a notification channel, HiveApi provides the `app/Ship/Configs/notification.php` config file, where you can 
specify an array of supported channels (e.g., SMS, Email, WebPush, ...), to be used for all your notifications.

If you wan to override the configuration for specific `Notification` classes, or if you prefer to defined the channels 
within each `Notification` class itself, you can override the `public function via($notifiable)` in respective class 
and define your channels. 

Checkout the [Laravel Notification Channels documentation](http://laravel-notification-channels.com) for list of 
supported integrations.

## Queueing a Notification 

In order to queue a notification for later use, you should use `Illuminate\Bus\Queueable` and implement 
`Illuminate\Contracts\Queue\ShouldQueue`.

## Use Database Channel  

First you need to generate a notification migration file that holds your `Notifications` to be displayed for your clients.
You can easily call respective Artisan command `php artisan notifications:table` or use `php artisan hive:generate:migration`. 
Then run `php artisan migrate` to migrate your database,
 
Please note that `HiveApi` already provides the `xxxxxx_create_notifications_table.php` in the default migrations files 
directory `app/Ship/Migrations/`. So you don't need to create / configure it manually. Nice, isn't it?
