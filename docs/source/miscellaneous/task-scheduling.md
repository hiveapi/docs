# Task Scheduling

* is a script executor program, such as `Cron Job`. A Cron Job is a time-based scripts scheduler in Unix-like computer 
operating systems.
* its role is to schedule the execution of `Artisan` Commands", periodically at fixed times, dates, or intervals.
* Laravel has a wrapper around the "Cron Job" called the Laravel scheduler. This allows the framework to schedule class 
like and Artisan Commands, Queued Jobs in addition to custom Shell Commands, to run later.

Below is a quick guide for how to schedule some scripts execution such as (custom Shell Commands, Laravel Commands, 
Laravel [Jobs](https://laravel.com/docs/master/queues), and other classes), in order to run at specific intervals or dates.

## Setup Server

First you need to setup your server by adding a single Cron entry as follow:

1) On the command line, type `crontab -e`

2) At the last line add the following: `* * * * * php /path/to/your/project/artisan schedule:run >> /dev/null 2>&1`

More details regarding the configuration can be found in the 
[official Laravel documentation](https://laravel.com/docs/master/scheduling#introduction). 

## Setup and Configure your Application

First you need to create some commands, that needs to be scheduled. The can be created in the Containers 
`(app/Containers/{container-name}/UI/CLI/Commands)` or in the Ship (`app/Ship/Commands`). See the 
[Commands Page](./../components/commands.html) for more details.

Once you have your command ready, go to `app/Ship/Kernels/ConsoleKernel.php` and start adding the commands you need 
to schedule inside the `schedule` function.

Note that you do not need to register the commands with the `$commands` property or point to them in the `commands()` 
function. HiveApi will do that automatically for you.

Example:

```php
<?php
    protected function schedule(Schedule $schedule)
    {
         $schedule->command('hive:welcome')->everyMinute();
         $schedule->job(new myJob)->hourly();
         $schedule->exec('touch me.txt')->dailyAt('12:00');
         // ...
    }
``` 

The [official Laravel documentation](https://laravel.com/docs/scheduling#defining-schedules) provides more details.
