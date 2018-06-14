# Jobs

A Job 

* is a simple class that can execute one specific task. 
* is a name given to a class that is usually created to be queued (its execution is usually deferred for later, after the 
execution of previous Jobs are completed).
* can be scheduled to be executed later by a queuing mechanism (queue system like beanstalkd).
* class is dispatched, it performs its specific job and dies.
* Laravel's queue worker will process every Job as it is pushed onto the queue.
 
More information can be found in the [official Laravel documentation](https://laravel.com/docs/queues).

## Rules

- All Jobs **MUST** extend from `App\Ship\Parents\Jobs\Job`.
- A Container **MAY** have more than one Job.

## Folder Structure

```
app
    Containers
        {container-name}
            Jobs
                DoSomethingJob.php
                DoSomethingElseJob.php
```

## Code Samples

`CreateAndValidateAddressJob`

```php
<?php

namespace App\Containers\Shipment\Jobs;

use App\Port\Job\Abstracts\Job;

class CreateAndValidateAddressJob extends Job
{
    private $recipients;

    public function __construct(array $recipients)
    {
        $this->recipients = $recipients;
    }

    public function handle()
    {
        foreach ($this->recipients as $recipient) {
            // do whatever you like
        }
    }
}
```

### Calling a Job from an Action

```php
<?php

// using helper function
dispatch(new CreateAndValidateAddressJob($recipients));

// manually
App::make(\Illuminate\Contracts\Bus\Dispatcher\Dispatcher::class)->dispatch(new CreateAndValidateAddressJob($recipients));
```

### Execute Jobs

For running your Jobs checkout the [Tasks Queuing](./../miscellaneous/tasks-queuing.html) page.
