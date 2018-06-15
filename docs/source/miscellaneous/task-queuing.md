# Task Queuing

* Queues System, which executes `Jobs` one by one once he receives it or once it's scheduled (after being serialized 
and stored in a string somewhere). 
* to be able to queue the Jobs you need a Queuing System such as Beanstalkd, Redis, Amazon SQS or simply the Database.
* Laravel has a "queue worker" that will process new Jobs as they are pushed onto the queue system, ("queue:work" and 
"queue:listen"). Its job is to push the jobs to the queue system in order to be executed later.
* to keep the "queue worker `php artisan queue:work` command, running permanently in the background, you should use a 
process monitor such as "Supervisor" to ensure that the queue worker does not stop running. It will simply make sure 
to execute the `php artisan queue:work` command.
* so its role is to schedule the execution of Artisan Command, Jobs, Event Listeners, and some other classes, at 
specific intervals or dates using the third party Queueing System.
 
More info can be found at the [official Laravel documentation](https://laravel.com/docs/queues).

HiveApi detects by default, which queue you are planning to use (based on the configurations), and creates the required 
migration files (if driver `databaes` is used)

```php
if (Config::get('queue.default') == 'database')
{
   // do something
}
```

## Beanstalkd

In order to use Beanstalkd as your queue driver, you need to require the `"pda/pheanstalk": "^3.1"` package first. You 
can include this in any `composer.json` file you want.
