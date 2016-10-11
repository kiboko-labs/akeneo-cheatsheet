Akeneo Cheat Sheet
==================

## Console command shortcuts

See further chapters to know how to install those command aliases on 
[Prod](#shortcuts-on-your-production-environment) or [Dev](#shortcuts-on-your-development-environment)

| Command     | Description                                             |
| ----------- | ------------------------------------------------------- |
| `ak-assets` | Regenerate assets for the whole application             |
| `ak-routes` | Regenerate js routes for the whole application          |
| `ak-cache`  | Regenerate caches                                       |
| `ak-trans`  | Regenerate translations files                           |
| `ak-batch`  | Run batch jobs                                          |
| `ak-jobs`   | List batch jobs                                         |

### Shortcuts on your production environment

On your production environments, insert those lines in your `~/.bash_profile` file :

```bash
alias ak-assets='rm -rf app/cache/prod/* web/bundles/* web/css/* web/js/* && app/console pim:install:ass --env=prod && app/console assets:install --env=prod && app/console fos:js-routing:dump --target="web/js/routes.js" --env=prod'
alias ak-routes='rm -rf web/js/routes.js && app/console fos:js-routing:dump --target="web/js/routes.js" --env=prod'
alias ak-cache='rm -rf app/cache/prod/* && app/console cache:clear --env=prod'
alias ak-trans='app/console translation:update --env=prod && app/console oro:translation:dump --env=prod'
alias ak-batch='app/console akeneo:batch:job --env=prod'
alias ak-jobs='app/console akeneo:batch:list-jobs --env=prod'
```

Then run `source ~/.bash_profile`

### Shortcuts on your development environment

On your development environments, insert those lines in your `~/.bash_profile` file :

```bash
alias ak-assets='rm -rf app/cache/dev/* web/bundles/* web/css/* web/js/* && app/console pim:install:ass --env=dev && app/console assets:install --env=dev && app/console fos:js-routing:dump --target="web/js/routes.js" --env=dev'
alias ak-routes='rm -rf web/js/routes.js && app/console fos:js-routing:dump --target="web/js/routes.js" --env=dev'
alias ak-cache='rm -rf app/cache/dev/* && app/console cache:clear --env=dev'
alias ak-trans='app/console translation:update --env=dev && app/console oro:translation:dump --env=dev'
alias ak-batch='app/console akeneo:batch:job --env=dev'
alias ak-jobs='app/console akeneo:batch:list-jobs --env=dev'
```

Then run `source ~/.bash_profile`

## Having fatal errors after upgrading Akeneo or one of your dependencies

If you are experiencing such errors like :

```
request.CRITICAL: Uncaught PHP Exception ReflectionException: "Property Pim\Bundle\CatalogBundle\Entity\Channel::$color does not exist" at /var/www/vendor/doctrine/common/lib/Doctrine/Common/Persistence/Mapping/RuntimeReflectionService.php line 82 {"exception":"[object] (ReflectionException(code: 0): Property Pim\\Bundle\\CatalogBundle\\Entity\\Channel::$color does not exist at /var/www/vendor/doctrine/common/lib/Doctrine/Common/Persistence/Mapping/RuntimeReflectionService.php:82)"} []
```

It is probably an APCu cache issue. To fix it, restart your Apache and PHP-FPM services :

`sudo service apache2 restart`

`sudo service php5-fpm restart`

## Fix cache file permissions

On some platforms, you may have some file permission issues in the cache
files when you have a web user different from your shell user.
 
To fix this run those commands, replacing `www-data:www-data` by the correct
user and group on your server if needed : 

```bash
chown -R www-data:www-data app/cache/*
sudo chmod -R ug-x app/cache/*
sudo chmod -R a-wx app/cache/*
sudo chmod -R ug+Xrw app/cache/*
sudo chmod -R a+Xr app/cache/*
```
