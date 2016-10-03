Akeneo Cheat Sheet
==================

## Console command shortcuts

See further chapters 

| Command     | Description                                             |
| ----------- | ------------------------------------------------------- |
| `ak-assets` | Regenerate assets for the whole application             |
| `ak-routes` | Regenerate js routes for the whole application          |
| `ak-cache`  | Regenerate caches                                       |

### On your production environment

On your production environments, insert those lines in your `~/.bash_profile` file :

```bash
alias ak-assets='rm -rf app/cache/prod/* web/bundles/* web/css/* web/js/* && app/console pim:install:ass --env=prod && app/console assets:install --env=prod && app/console fos:js-routing:dump --target="web/js/routes.js" --env=prod'
alias ak-routes='rm -rf web/js/routes.js && app/console fos:js-routing:dump --target="web/js/routes.js" --env=prod'
alias ak-cache='rm -rf app/cache/prod/* && app/console cache:clear --env=prod'
```

Then run `source ~/.bash_profile`

### On your development environment

On your development environments, insert those lines in your `~/.bash_profile` file :

```bash
alias ak-assets='rm -rf app/cache/dev/* web/bundles/* web/css/* web/js/* && app/console pim:install:ass --env=dev && app/console assets:install --env=dev && app/console fos:js-routing:dump --target="web/js/routes.js" --env=dev'
alias ak-routes='rm -rf web/js/routes.js && app/console fos:js-routing:dump --target="web/js/routes.js" --env=dev'
alias ak-cache='rm -rf app/cache/dev/* && app/console cache:clear --env=dev'
```

Then run `source ~/.bash_profile`


