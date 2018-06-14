# Languages

Languages are not real components, but rather serve as additional resources to a specific container. More specifically, 
these files hold translations.

## Rules

- Language files **CAN** be placed inside the containers. However, the default laravel `resources/lang` languages files 
are still loaded and can be used as well.
- All translations are automatically namespaced as the lowercase name of the container.

## Folder Structure

```shell
app
    Containers
        {container-name}
            Resources
                Languages
                    en
                        messages.php
                        users.php
                    de
                        messages.php
                        users.php
```

## Usage

To get a translation from a specific container, call it like this:

```php
<?php

trans('welcome::messages.headline.title');
```

where `welcome` is the name of the container to search for the localization file, `messages` is the actual file to search for,
and `headline.title` is the localization key to be resolved within this file.

For more info about the localization checkout the [Localization](./../features/localization.html) page.
