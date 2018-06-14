# Configuration Files

Configs are files that hold configurations for the specific container. For more details about them check the official 
Laravel [documentation](https://laravel.com/docs/5.6/configuration).

In each container, there are two types of config files:
- the container specific config file that contains the container specific configurations.
- the container third-party packages config files (i.e., a config file that belongs to a third-party package, required 
by the composer file of the container).

## Principles

- Your custom config files and the third-party packages config files, should be placed in the container. If they are 
too generic then it can be placed on the Ship layer.
- Container can have as many config files as they need.

## Rules

- When publishing a third-party package config file you **SHOULD** move it manually to its respective container or to 
the `Ship` Config folder.
- Framework config files (provided by Laravel) lives at the default config directory in the root of the project.
- You **SHOULD NOT** add any config file to the `config` directory.
- The container specific config file, **MUST** have the same name of the container in lower letters and post-fixed 
with `-container`, to prevent conflicts between third-party packages and container specific packages.

## Folder Structure
```shell
app
    Containers
        {container-name}
            Configs
                {container-name}-container.php
                package-config-file1.php
                ...
    Ship
        Configs
            hashids.php
            hive.php
            ...
config
    app.php
    ...
```

## Code Samples

```php
<?php

return [

    /*
    |--------------------------------------------------------------------------
    | Basic Configuration
    |--------------------------------------------------------------------------
    */
    'custom-value' => 'foo',
    'enabled' => true,

    // some other config params here...
```

You can access the respective configuration key like this:

```php
$value = Config::get('{container-name}-container.custom-value');     // returns 'foo'
$value = config('{container-name}-container.custom-value');          // same, but using a function and not the Facade

$defaultValue = Config::get('{container-name}-container.unknown.key', 'defaultvalue');   // returns 'defaultvalue' as the key is not set!
```
