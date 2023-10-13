# Magento 2 Docker Compose

A ready to use Magento 2 local environment.

Features:

- Just 1 configuration file is required in your existing project
- PHP 7.4 + XDebug
- Nginx + SSL
- MariaDB
- cron:run executed constantly
- Redis Cache
- Elasticsearch
- Mailhog

## Before Using

You need to clone this repository only once, and you can use it with multiple
magento projects. _Different Magento versions may need different branch beeing active. You may clone this repo multiple times to make it easier to manage._

1. Clone the repository
2. Run `bin/create-self-signed-ssl`
3. `mkdir -p ~/.config/composer` - in case you don't have global composer set-up

## Installing for existing Magento 2.4+ project

Add .env file in your project with following content. Adjust to needs.

```
# A relative path to the cloned magento2-docker-compose/docker-compose.yml file
COMPOSE_FILE=../path/to/magento2-docker-compose/docker-compose.yml

# Unique name for project, to avoid name clash when using with multiple projects
COMPOSE_PROJECT_NAME=unique-name

# Path to magento source files, should stay as it is unless you have magento installed in some folder.
MAGENTO_PATH=${PWD}
```

Start with `docker-compose up` and then jump to "Configuring Magento" section

## Installing for new Magento 2.4+ project

If you don't have Magento project yet, you'll need to follow few additional
steps. Run commands in the folder you added .env file

1. Create new project folder and `.env` file inside as in above section
2. `mkdir -p lib pub setup update vendor var/composer_home`
3. `docker-compose up`
4. `docker compose exec php composer config --global http-basic.repo.magento.com <PUBLIC KEY> <PRIVATE KEY>`
5. `docker-compose exec php composer create-project --repository-url=https://repo.magento.com/ magento/project-community-edition magento-installation`
6. `docker-compose exec php bash -c 'shopt -s dotglob && cp -R magento-installation/* ./ && rm -R magento-installation/'`
7. Run the `setup:install` command from "Configuring Magento" section
8. Add an admin user with this command  
```
docker-compose exec php bin/magento setup:install \
    --admin-firstname=admin \
    --admin-lastname=admin \
    --admin-email=admin@example.org \
    --admin-user=admin \
    --admin-password=password123
```
8. Open https://localhost for store and https://localhost/admin for admin page

## Configuring Magento

Following parameters are required for a correct magento install.
_Please note that there are few other required parameters not included here._

```
docker-compose exec php bin/magento setup:install \
    --use-secure 1 \
    --use-secure-admin 1 \
    --base-url https://localhost/ \
    --base-url-secure https://localhost/ \
    --db-host db \
    --db-name magento \
    --db-user magento \
    --db-password magento2 \
    --search-engine elasticsearch7 \
    --elasticsearch-host elasticsearch
```

## Common commands reference

- `docker-compose up` - start environment
- `docker-compose stop` - stop environment
- `docker-compose exec php composer` - run composer
- `docker-compose exec php bin/magento` - run bin/magento
- `docker-compose exec php bash` - open bash prompt in php container (allows you to run `bin/magento`, `composer` without docker-compose prefix)
- `docker-compose exec db mysqldump -umagento -pmagento2 magento > database.sql` - make database dump
- `docker-compose exec db mysql -umagento -pmagento2 -e "DROP DATABASE magento; CREATE DATABASE magento;"` - clean database
- `docker-compose exec -T db mysql -umagento -pmagento2 magento < database.sql` - import database.sql file
- `docker-compose exec cache redis-cli "FLUSHALL"` - delete all redis data

## Opinionated defaults

This repository comes with few default Magento configuration changes. You can see all of them in `default.env` file.

## Extra `.env` configuration

You can use following `.env` variables to amend behaviour of this environment:

- `MAGE_RUN_TYPE` - Sets Magento run type for multi-store set-ups
- `PHP_MEMORY_LIMIT` - Changes PHP's memory_limit (default: 2G)
- `PHP_DATE_TIMEZONE` - Chnges PHP's date.timezone
- `XDEBUG_MODE` - Allows to change XDebug mode
- `XDEBUG_CONFIG` - XDebug configuration
- `MARIADB_EXPOSED_PORT` - Changes default exposed MariaDB port (default: 3306)

## Sample Data Setup (when magento2 repository is cloned)

When you cloned the magento/magento2 repository from github the sample data setup
is slightly different than when magento2 is setup as composer project. More details here: 
https://devdocs.magento.com/guides/v2.4/install-gde/install/sample-data-after-clone.html

With this docker-compose it's important that you clone the sample-data repository
within the magento2 project as docker container is limited to project folder only.

Once you clone the repository run the command mentioned in article, but inside
php container

`docker-compose exec php php -f magento2-sample-data/dev/tools/build-sample-data.php -- --ce-source ./`

## E-mails

All emails from PHP containers can be seen in Mailhog at http://localhost:8025
