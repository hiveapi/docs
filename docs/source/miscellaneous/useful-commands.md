# Useful Commands

HiveApi already ships with many useful commands to help you speed up the development process. You can see list of all 
commands, by typing `php artisan hive` and look for the `HiveApi` (`hive`) section.

## Available Commands (Excerpt)

- `php artisan hive:list:actions` Show all Actions within the application
- `php artisan hive:list:tasks` Show all Tasks within the application
- `php artisan hive:seed:test` Seeds your custom testing data from `app/Ship/Seeders/SeedTestingData.php`.
- `php artisan hive:seed:deploy` Seeds your custom deployment data from `app/Ship/Seeders/SeedDeploymentData.php`.
- `php artisan hive:generate:xxx` Generate a specific component for the framework (e.g., `Action`, `Task`, ...). For 
more details on the [Code Generator](./../features/code-generator.html).

## List All Actions / Tasks Command

It's useful to be able to see all the implemented use cases in your application. To do so type
`php artisan hive:list:actions`. You can also pass `--withfilename` flag to see all Actions with the files names.  
`hive:list:actions --withfilename`.

The same command works for Tasks as well.

<a name="list-container-dependencies-command"></a>
## List Container Dependencies Command

Sometimes it is required to show dependencies between containers (e.g., how they are _interlinked_ amongst each others). 
HiveApi provides a command to list all dependencies for one specific container. The command does take the `Hive::call()` 
and `$this->call()` (with `use X`) into account.

If you want to get the dependencies for one container, you can call 

```bash
php artisan hive:list:dependencies app/Containers/X
```

where `X` is the name of the requested container (e.g., `User`).
