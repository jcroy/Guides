To set up a Django application with PostgreSQL and Nginx on a Linux server, follow these steps:

1. Update and upgrade your system, then install the necessary packages:

    ```bash
    sudo apt update && sudo apt upgrade -y
    sudo apt install python3-pip python3-dev python3-venv libpq-dev -y
    ```

2. Create a directory for your Django app and navigate into it:

    ```bash
    mkdir ~/mydjangoapp && cd ~/mydjangoapp
    ```

    Note: I prefer to put Django in `/var/www/dir/` as it's easier to share with the nginx user.

3. Create a Python virtual environment and activate it:

    ```bash
    python3 -m venv myenv
    source myenv/bin/activate
    ```

4. Install Django and Gunicorn:

    ```bash
    pip install django gunicorn
    ```

5. Start a new Django project if you are not cloning an exisiting site:

    ```bash
    django-admin startproject myproject .
    ```
    Also make sure to collect the static files using  manage.py for nginx. 
 
6. Install PostgreSQL and create your database:

    ```bash
    sudo apt install postgresql postgresql-contrib -y
    sudo -u postgres psql
    ```

    Within the PostgreSQL prompt, enter:

    ```sql
    CREATE DATABASE mydb;
    CREATE USER myuser WITH PASSWORD 'password';
    ALTER ROLE myuser SET client_encoding TO 'utf8';
    ALTER ROLE myuser SET default_transaction_isolation TO 'read committed';
    ALTER ROLE myuser SET timezone TO 'UTC';
    GRANT ALL PRIVILEGES ON DATABASE mydb TO myuser;
    \q
    ```

7. Configure your Django project to use the PostgreSQL database. In `myproject/settings.py`, set the `DATABASES` configuration:

    ```python
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.postgresql',
            'NAME': 'mydb',
            'USER': 'myuser',
            'PASSWORD': 'password',
            'HOST': 'localhost',
            'PORT': '',
        }
    }
    ```

    Note: Recommend setting this as an environment variable to avoid leaking data if using version control.

8. Migrate the database and run the Django development server:

    ```bash
    python manage.py migrate
    python manage.py runserver 0.0.0.0:8000
    ```

9. Install Nginx and configure it to serve your Django app:

    ```bash
    sudo apt install nginx -y
    ```

    Create a configuration file in `/etc/nginx/sites-available/myproject` with the following content:

    ```nginx
    server {
        listen 80;
        server_name your_domain.com;

        location = /favicon.ico { access_log off; log_not_found off; }
        location /static/ {
            root /path/to/your/static/files;
        }

        location / {
            include proxy_params;
            proxy_pass http://unix:/path/to/your/mydjangoapp/myproject.sock;
        }
    }
    ```

    Create a symlink to enable the site:

    ```bash
    sudo ln -s /etc/nginx/sites-available/myproject /etc/nginx/sites-enabled
    ```

    Note: If you want to take this a bit further, create a local git repo and put your configs there and symlink files from there. Then you can version control easily.

10. Test the Nginx configuration and restart Nginx:

    ```bash
    sudo nginx -t
    sudo systemctl restart nginx
    ```

11. Create a systemd service file for Gunicorn to ensure it starts on boot and stays running:

    Create `/etc/systemd/system/gunicorn.service` with the following content:

    ```ini
    [Unit]
    Description=gunicorn daemon
    After=network.target

    [Service]
    User=ubuntu
    Group=www-data
    WorkingDirectory=/path/to/your/mydjangoapp
    ExecStart=/path/to/your/mydjangoapp/myenv/bin/gunicorn --access-logfile - --workers 3 --bind unix:/path/to/your/mydjangoapp/myproject.sock myproject.wsgi:application

    [Install]
    WantedBy=multi-user.target
    ```

12. Start and enable the Gunicorn service:

    ```bash
    sudo systemctl start gunicorn
    sudo systemctl enable gunicorn
    ```
Thats the rough outline of steps to take. It should be up and running.

Todo  
* Add documentation for ssl via certbot. 
