# Extending Javascript

Sometimes we need to extend the vendor's javascripts to enhance the client behaviour or to get rid of limitations, like hardcoded values that may be problematic.
For example, in `vendor/akeneo/pim-community-dev/src/Pim/Bundle/DataGridBundle/Resources/public/js/datagrid/pagination-input.js`, we can see that the value of `maxRescoreWindow` is hardcoded (this can be an issue since it has a great influence on the display of pagination handles):
```javascript
// vendor/akeneo/pim-community-dev/src/Pim/Bundle/DataGridBundle/Resources/public/js/datagrid/pagination-input.js

/* global define */

/**
 * Datagrid pagination with input field
 *
 * @export  oro/datagrid/pagination-input
 * @class   oro.datagrid.PaginationInput
 * @extends oro.datagrid.Pagination
 */
define(
    [
        'jquery',
        'oro/mediator',
        'underscore',
        'oro/datagrid/pagination',
        'pim/template/datagrid/pagination',
        'jquery.numeric'
    ], function(
        $,
        mediator,
        _,
        Pagination,
        template
    ) {
    'use strict';

    const PaginationInput = Pagination.extend({
        collection: {},

        /** @property */
        template: _.template(template),

        /** @property */
        windowSize: 3,

        /** @property */
        fastForwardHandleConfig: {
            gap: {
                label: '...'
            }
        },

        /** @property */
        maxRescoreWindow: 10000,
```
This value is actually a ElasticSearch index setting, and it would be great to ask ElasticSearch for that value, instead of hardcoding it. This is possible by extending the javascript:

```javascript
// src/AppBundle/Resources/public/js/pagination-input.js
define(
    ['jquery', 'oro/datagrid/pagination-input'],
    function ($, PaginationInput) {

        var paginationSettings = {}
        $.ajax({
            url: '/app/index/settings/pagination',
            async: false
        })
            .done(function(data) {
                paginationSettings = data
            })

        const LargePageInput = PaginationInput.extend({
            maxRescoreWindow: paginationSettings.max_rescore_window
        })

        return LargePageInput
    }
)
```
> This script assumes there is a backend route that asks ElasticSearch for the `max_rescore_window` value and returns it as a JSON object.

This is called a module. The module must now be registered for RequireJS:
```yaml
# src/AppBundle/Resources/config/requirejs.yml
config:
    paths:
        kiboko/datagrid/pagination-input: app/js/pagination-input
```
> `kiboko/datagrid/pagination-input` is an arbitrary identifier we give to the module and `app/js/pagination-input` is the public path (accessible by the web server, usually under `web/bundles`) to the javascript file that should be loaded.

The new module can now be used where needed.

## How to use the new module
We need to override the appropriate form extension to tell it to load the new module. The hardest is maybe to find out which extension is to be overriden. Once found, just copy paste the extension definition in the project resources and change the identifier of the module:
```yaml
# src/AppBundle/Resources/config/form_extensions/product/index.yml
extensions:
    pim-product-index-pagination:
        module: kiboko/datagrid/pagination-input
        parent: pim-product-index-grid-container
        targetZone: toolbar
        config:
            gridName: product-grid
```
> Obviously, the value passed to the `module` key is the arbitrary identifier that was defined during the RequireJS definition (see above).

Now clear the cache:
```bash
rm -rf var/cache/*
```
Reinstall the assets:
```bash
bin/console pim:installer:assets --env=prod
```
Repack:
```bash
yarn run webpack
```
