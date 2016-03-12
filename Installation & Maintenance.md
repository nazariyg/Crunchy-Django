# Django Installation & Maintenance

## Installation

    (Nginx uwsgicluster)
    (sudo apt-get update)
    (sudo apt-get install libpq-dev)
    mkdir <appname>
    cd <appname> && touch README.md .gitignore && cd -
    mkdir <appname>/requirements
    cd <appname>/requirements && touch base.txt local.txt staging.txt production.txt && cd -
    cd <appname> && virtualenv <appname>_venv && cd -
    . <appname>/<appname>_venv/bin/activate
    [add to] <appname>/requirements/base.txt
    [[
        Django==<ver>
        uwsgi
        ...
    ]]
    [add to] <appname>/requirements/local.txt
    [[
        -r base.txt

        ...
    ]]
    pip install -r <appname>/requirements/<local, production, staging>.txt
    cd <appname> && django-admin startproject <appname> . && cd -
    mkdir <appname>/<appname>/settings
    cd <appname>/<appname>/settings && touch __init__.py base.py local.py staging.py test.py production.py && cd -
    [refactor settings]
    [edit and add uwsgi directory beside manage.py]
    ([edit manage.py for settings location])
    ([edit wsgi.py for settings location])
    [add environment variables to] <appname>/<appname>_venv/bin/activate
    [[
        export PYTHONPATH=/var/www/<appname>
        export DJANGO_SETTINGS_MODULE=<appname>.settings.<local, production, staging>

        export DJANGO_SECRET_KEY='<key>'
        export DJANGO_DB_PASSWORD='<pass>'
        ...
    ]]
    mkdir <appname>/staticroot
    [add to settings/base.py]
    [[
        STATIC_ROOT = "/var/www/<appname>/staticroot/"
    ]]

## Installation - Simpler

    [inside the app's directory] virtualenv <appname>_venv
    . <appname>_venv/bin/activate
    pip install Django==<ver> uwsgi
    django-admin startproject <appname> .

## More

    [create a database named <appname>]
    ([add a cron job for] python manage.py clearsessions)
    [on new requirements] pip install -r requirements/<local, production, staging>.txt

## Shell

    python manage.py collectstatic
    [initial tables for installed apps] python manage.py migrate
    python manage.py makemigrations <djangoappname>
    (python manage.py sqlmigrate <djangoappname> <#>)
    python manage.py check
    python manage.py migrate
    [most often used for migrations] python manage.py makemigrations && python manage.py migrate
    python manage.py shell
    python manage.py test <subappname>

## Apps

    [(Backup and) sync forward]
    python manage.py startapp <djangoappname>
    [Sync backward]
    [add <djangoappname> to INSTALLED_APPS]

## Administration

    python manage.py createsuperuser
    [in admin.py] admin.site.register(ModelClass)

## Django info

    python -c "import sys; sys.path = sys.path[1:]; import django; print(django.__path__)"
    python -c "import django; print(django.get_version())"

## MySQL #1

    pip3 install --allow-external mysql-connector-python mysql-connector-python

## MySQL #2

    sudo apt-get install python-dev libmysqlclient-dev
    pip3 install mysqlclient
