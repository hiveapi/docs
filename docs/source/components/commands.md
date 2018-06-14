# Commands

A command

* is a Laravel `artisan` command. Laravel has it's own default commands and you create your own application-specific 
commands as well.
* provides a way to interact with the Laravel application.
* can be scheduled by a Task scheduler, like CronJob or by the Laravel built in wrapper of the Cron Job "laravel scheduler".
* could be Closure based or Class based.
* "dispatch" is the term that is usually used to call a Command.

## Principles

- Containers **MAY** or **MAY NOT** have one or more Commands.
- Every Command **SHOULD** call an `Action` to perform its job.
- Commands itself **SHOULD NOT** contain any business logic.
- The `Ship` layer **MAY** contain application wide commands.

## Rules

- All Commands **MUST** extend from `App\Ship\Parents\Commands\ConsoleCommand`.

## Folder Structure

```
app
    Containers
        {container-name}
            UI
                CLI
                    Commands
                        SayHelloCommand.php
                        ...
    Ship
        Commands
            GeneralCommand.php
            ...
```

## Code Samples

```php
<?php

namespace App\Containers\Welcome\UI\CLI\Commands;

use App\Ship\Parents\Commands\ConsoleCommand;

class SayHelloCommand extends ConsoleCommand
{

    protected $signature = 'hive:welcome';

    protected $description = 'Just saying "Hi"';

    public function handle()
    {
        $this->info('Welcome to HiveApi'); // green color
        $this->line('Welcome to HiveApi'); // normal color
    }
}
```

### Calling a Command from the Console

You can call your custom commands like any other artisan command:

```shell
php artisan hive:welcome
```

### Calling a Command from your Application

You can call a command from your own application code like this:
```php
<?php
Artisan::call('hive:welcome');
```

### Schedule Commands Execution

To schedule the execution of a Command checkout the [Tasks Scheduling](./../miscellaneous/tasks-scheduling.html) page.

## Define Consoles Routes

To define Console route go to `app/Ship/Commands/Routes.php`.
