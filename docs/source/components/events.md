# Events

Events

 - provide a simple `Observer` implementation, allowing you to subscribe and listen for various events that occur in 
 your application.
 - are classes that can be fired from anywhere in your application.
 - will usually be bound to one or many `Events Listeners` classes or has those Listeners registered to listen to it.
 - "fire" is the term that is usually used to call an Event.

More details can be found at the [official Laravel documentation](https://laravel.com/docs/events).

## Principles

- Events **CAN** be fired from `Actions` or `Tasks`. They are preferable to choose one place only (`Tasks` are recommended).
- Events **SHOULD** be created inside the Containers. However, generic Events **CAN** be created in the Ship layer.

### Rules

- All Events **MUST** extend from `App\Ship\Parents\Events\Event`.

### Folder Structure

```
app
    Containers
        {container-name}
            Events
                SomethingHappenedEvent.php
            Listeners
                ListenToMusicListener.php
    Ship
        Events
            GlobalStateChanged.php
            SomethingBigHappenedEvent.php
```

## Usage

In Laravel you can create and register events in multiple way. Below is an example of an Event that handles itself. 

```php
<?php

namespace App\Containers\User\Events;

use App\Containers\User\Models\User;
use App\Ship\Parents\Events\Event;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Support\Facades\Log;

class UserRegisteredEvent extends Event implements ShouldQueue
{
    protected $user;

    public function __construct(User $user)
    {
        $this->user = $user;
    }

    public function handle()
    {
        Log::info('New User registration. ID = ' . $this->user->getHashedKey() . ' | Email = ' . $this->user->email . '.');
        // ...
    }

    public function broadcastOn()
    {
        return new PrivateChannel('channel-name');
    }
}
```

You will get more benefits creating `Events Listeners` for each Event. To do this you will need to extend this 
EventsProvider `HiveApi\Core\Abstracts\Providers\EventsProvider`.

### Dispatch Events

You can dispatch an Event from anywhere you want (ideally from `Actions` and `Tasks`). Consider the following example 
for dispatching the Event class from the example above.

```php
<?php

// using helper function
event(New UserEmailChangedEvent($user));

// manually
\App::make(\Illuminate\Contracts\Bus\Dispatcher\Dispatcher::class)->dispatch(New UserEmailChangedEvent($user));
```

### Queueing an Event

Events can implement `Illuminate\Contracts\Queue\ShouldQueue` to be queued.

### Handling an Event

You can handle jobs on dispatching an event. To do so you need to implement one of the following interfaces:

- `HiveApi\Core\Abstracts\Events\Interfaces\ShouldHandleNow`
- `HiveApi\Core\Abstracts\Events\Interfaces\ShouldHandle`

This will force you to implement the `handle` method and will make HiveApi execute the method upon dispatching the event. 

- The `ShouldHandleNow` Interface will make the event execute the handle method as soon as the event gets dispatched.
- The `ShouldHandle` Interface will create an event job and execute the handle method async (through Laravel jobs).

```php
<?php 

namespace App\Containers\Example\Events;

use HiveApi\Core\Abstracts\Events\Interfaces\ShouldHandle;
use App\Ship\Parents\Events\Event;

class ExampleEvent extends Event implements ShouldHandle
{
    /**
     * If ShouldHandle interface is implemented this variable
     * sets the time (in seconds or timestamp) to wait before a job is executed
     *
     * @var \DateTimeInterface|\DateInterval|int|null $jobDelay
     */
    public $jobDelay = 60;

    /**
     * If ShouldHandle interface is implemented this variable
     * sets the name of the queue to push the job on
     *
     * @var string $jobQueue
     */
    public $jobQueue = "example_queue";

    public function handle()
    {
        // Do some handling here
    }
}
```

### Broadcasting

To define Broadcasting route go to `app/Ship/Boardcasts/Routes.php`.
