# Migration Akeneo avec Transporteo

> Documentation officielle : https://github.com/akeneo/transporteo

Transporteo est une application permettant de migrer un PIM Akeneo 1.7 vers
un PIM Akeneo 2.0. Elle assure la migration des données de la base.

> Des problèmes peuvent survenir pendant la migration des variantes si elle ont
été incorrectement modélisées dans le PIM 1.7. Le PIM 2.3 est beaucoup plus strict
sur la conception des variantes de produits.

Le code custom n'est pas migré.

Les images et autres assets ne pas migrés non plus. S'ils sont sur un système de fichiers distants, il faut simplement configurer le PIM 2.0 pour qu'il puisse y avoir accès. Sinon, il faut les copier manuellement dans le nouveau PIM.

## Prérequis

* Un PIM Akeneo 1.7 (source): il peut être sur le même serveur que la destination ou distant. Dans ce deuxième cas, la destination doit pouvoir s'y connecter via SSH. Le client mysql doit être installé, ainsi que mysqldump (généralement livré dans le même paquet que mysql).
* Un PIM Akeneo 2.0 (destination) : le client mysql doit être installé.

## Utilisation de Transporteo

### Installation

Transporteo doit être installé sur la même machine que la destination.

```bash
$ composer.phar create-project "akeneo/transporteo":"dev-master"
```
### Utilisation
```bash
$ php Transporteo.php akeneo-pim:migrate
```
Le process Transporteo pouvant être long de plusieurs heures, il est vivement recommandé de l'exécuter avec l'option verbose, afin d'avoir des informations plus précises en cas d'erreur critique.
```bash
$ php Transporteo.php akeneo-pim:migrate -vvv
```
Une fois la migration terminée, pensez à faire un backup de la base du PIM de destination avant de poursuivre les montées de version vers 2.3 et supérieures.

### Configuration
Transporteo peut être configuré à l'exécution via un formulaire dans le terminal. Il est cependant conseillé de créer et d'éditer le fichier `transporteo/src/Infrastructure/Common/config/parameters.yml`.

```yaml
parameters:
  default_responses:

    # PIM installation absolute path
    installation_path_source_pim: '/var/www/html'
    installation_path_destination_pim: '/var/www/html'

    # Only for remote source PIM
    ssh_hostname_source_pim: '172.22.0.1'
    ssh_port_source_pim: '2222'
    ssh_user_source_pim: 'root'

    # Base URI (with http://) to request the API
    api_base_uri_source_pim: 'http://172.22.0.1:8081'
    api_base_uri_destination_pim: 'http://172.22.0.1:8082'

    # API authentication
    api_client_id: '1_dj523co06hkcwgk80skoww4k8kkg8g8o0kw0k8848k08ow88w'
    api_secret: '28c8n2108fb480g4ww0wg0ogwoowosok8g4gwggssc0s4og8gs'
    api_user_name: 'jimmy'
    api_user_pwd: 'dkjJ29SZDJ'
```
#### Docker, Containers et Networks
Sur l'exemple précédent, les deux PIMs sont dans des containers et des réseaux Docker différents. Pour déterminer les IPs à utiliser dans les paramètres, il faut connaître l'IP du gateway du réseau sur lequel est connecté le container exécutant Transporteo :
```bash
docker network inspect <network> | grep -i gateway
```
Remplacez `<network>` par le nom de réseau adéquat. Pour afficher la liste des réseaux Docker:
```bash
docker network ls
```

## Débogage
Le process de Transporteo peut être très long. Si un problème survient et que pour le résoudre vous avez envie de n'exécuter que certaines parties de Transporteo, vous pouvez éditer le fichier `transporteo/src/Infrastructure/Common/config/transporteo_state_machine.yml` et commenter les transitions que vous ne voulez pas voir exécutées. Par exemple, pour passer directement à la migration de variantes:
```workflows:
    transporteo:
        transitions:
            ask_source_pim_location:
                from: 'ready'
                to: 'source_pim_location_guessed'
            local_source_pim_configuration:
                from: 'source_pim_location_guessed'
                to: 'source_pim_configured'
            distant_source_pim_configuration:
                from: 'source_pim_location_guessed'
                to: 'source_pim_configured'
            source_pim_api_configuration:
                from: 'source_pim_configured'
                to: 'source_pim_api_configured'
            source_pim_detection:
                from: 'source_pim_api_configured'
                to: 'source_pim_detected'
            ask_destination_pim_location:
                from: 'source_pim_detected'
                to: 'destination_pim_location_guessed'
            download_destination_pim:
                from: 'destination_pim_location_guessed'
                to: 'destination_pim_downloaded'
            destination_pim_pre_configuration:
                from: 'destination_pim_downloaded'
                to: 'destination_pim_pre_configured'
            destination_pim_configuration:
                from: ['destination_pim_downloaded', 'destination_pim_pre_configured']
                to: 'destination_pim_configured'
            destination_pim_api_configuration:
                from: 'destination_pim_configured'
                to: 'destination_pim_api_configured'
            destination_pim_detection:
                from: 'destination_pim_api_configured'
                to: 'destination_pim_detected'

#            destination_pim_check_requirements:
#                from: 'destination_pim_detected'
#                to: 'destination_pim_requirements_checked'
#            local_destination_pim_system_requirements_installation:
#                from: 'destination_pim_requirements_checked'
#                to: 'destination_pim_system_requirements_installed'
#            destination_pim_file_database_migration:
#                from: 'destination_pim_system_requirements_installed'
#                to: 'destination_pim_file_database_migrated'
#            destination_pim_structure_migration:
#                from: 'destination_pim_file_database_migrated'
#                to: 'destination_pim_structure_migrated'
#            destination_pim_family_migration:
#                from: 'destination_pim_structure_migrated'
#                to: 'destination_pim_family_migrated'
#            destination_pim_system_migration:
#                from: 'destination_pim_family_migrated'
#                to: 'destination_pim_system_migrated'
#            destination_pim_job_migration:
#                from: 'destination_pim_system_migrated'
#                to: 'destination_pim_job_migrated'
#            destination_pim_group_migration:
#                from: 'destination_pim_job_migrated'
#                to: 'destination_pim_group_migrated'
#            destination_pim_extra_data_migration:
#                from: 'destination_pim_group_migrated'
#                to: 'destination_pim_extra_data_migrated'
#            destination_pim_enterprise_edition_data_migration:
#                from: 'destination_pim_extra_data_migrated'
#                to: 'destination_pim_enterprise_edition_data_migrated'
#            destination_pim_reference_data_migration:
#                from: 'destination_pim_enterprise_edition_data_migrated'
#                to: 'destination_pim_reference_data_migrated'
#            destination_pim_product_migration:
#                from: 'destination_pim_reference_data_migrated'
#                to: 'destination_pim_product_migrated'

            destination_pim_product_variation_migration:
                from: 'destination_pim_detected'
                to: 'destination_pim_product_variation_migrated'
            finish_migration:
                from: 'destination_pim_product_variation_migrated'
                to: 'migration_finished'
```
Dans cet exemple, Transporteo n'exécutera que les étapes de configuration puis passera directement à la migration des variations. Notez la valeur de `from` dans la définition de `destination_pim_product_variation_migration`.

> Dans le cas de la migration des variations, il faut penser à vider les tables concernées en base de données:
```mysql
delete from pim_catalog_product_model;
delete from pim_catalog_variant_attribute_set_has_axes;
delete from pim_catalog_variant_attribute_set_has_attributes;
delete from pim_catalog_family_variant_translation;
delete from pim_catalog_family_variant_has_variant_attribute_sets;
delete from pim_catalog_family_variant;
```
## Troubleshooting

### Connexion SSH avec un couple utilisateur-mot-de-passe
Par défaut, Transporteo tente de se connecter aux serveurs SSH avec une clé privée. Si vous préférez vous connecter avec un couple utilisateur-mot-de-passe, modifiez la ligne suivante dans la class `\Akeneo\PimMigration\Infrastructure\Cli\Ssh`.
Commentez la ligne :
```
if (false === ssh2_auth_agent($connection, $username)) {
```
et ajoutez :
```
if (false === ssh2_auth_password($connection, $username, "root")) {
```
> Il ne faut bien sûr pas versionner ces modifications, à moins de travailler sur un fork.
