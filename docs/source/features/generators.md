# Generators

Code Generators are a nice and easy way to speed up development by creating boiler-plate code based on your inputs. You 
may already be familiar with the Laravel code generators (e.g., `php artisan make:controller`). 

HiveApi code generator works the same way. However, they are far more powerful and can generate an entire Container with 
fully working CRUD operations, including routes, requests, controller, Actions, Repositories, Models, Migrations, 
documentation, ... and and and.

## Available Code Generator Commands

To see the list of code generators type `php artisan hive:generate`.

```
  hive:generate:container        Create a Container for HiveApi from scratch
  hive:generate:action           Create a Action file for a Container
  hive:generate:configuration    Create a Configuration file for a Container
  hive:generate:controller       Create a controller for a container
  hive:generate:exception        Create a new Exception class
  hive:generate:job              Create a new Job class
  hive:generate:mail             Create a new Mail class
  hive:generate:migration        Create an "empty" migration file for a Container
  hive:generate:model            Create a new Model class
  hive:generate:notification     Create a new Notification class
  hive:generate:repository       Create a new Repository class
  hive:generate:request          Create a new Request class
  hive:generate:route            Create a new Route class
  hive:generate:seeder           Create a new Seeder class
  hive:generate:serviceprovider  Create a ServiceProvider for a Container
  hive:generate:subaction        Create a new SubAction class
  hive:generate:task             Create a Task file for a Container
  hive:generate:transformer      Create a new Transformer class for a given Model
```

To get more info about each command, add `--help` to the command. Example: `php artisan hive:generate:route --help`. 
The help page shows all options, which can be directly passed to the command.

If you do not provide respective information via the command line options, a wizard will be displayed to guide you through
the process of creating specific components.

For example, you can directly call `php artisan hive:generate:controller --file=UserController` to directly specify the 
class to be generated. The wizard, however, will ask you for the `--container` as well.

Note that **all** generators automatically inherit the options `--container` and `--file` (these are documented as well 
in the help page). Furthermore, a generator may have specific options as well (e.g., the `--ui` (user-interface)
to generate something for).

## Custom Code Stubs (aka. Customizing the Generator)

If you don't like the automatically generated code (or would like to adapt it to your specific needs) you can do this 
quite easily.

The existing `Generators` allow to read `custom stubs` from the `app/Ship/Generators/CustomStubs` folder. The name of 
file needs to be the same as in `vendor/hiveapi/core/src/Generator/Stubs`.

Say, if you like to change the `config.stub`, simply copy the file to `app/Ship/Generators/CustomStubs/config.stub` and 
start adapting it to your needs. 

If you run the respective command (e.g., in this case `php artisan hive:generate:configuration`) 
this would read your specific `config.stub` file instead the pre-defined one!
