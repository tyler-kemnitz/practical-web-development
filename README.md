## Practical Web Development | [Udemy](https://www.udemy.com/course/practical-web-development-with-docker-django-nginx-redis/)

This project creates a web application built using Docker, Django, Nginx, Redis and Gunicorn.

## Starting MYSQL Container ###

```
$ docker run -d --name app-db --cpus 0.5 --memory 512m -e MYSQL_USER=tyler -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=web_app_db --net app-net --platform linux/x86_64 mysql:5.7 
```

### Apple Silicon Note  
Received error when running the above command without the `--platform` arg and instead specifying `mysql:5.7`: 

``` 
docker: no matching manifest for linux/arm64/v8 in the manifest list entries. when running command w/ simple mysql:5.7
```

Specifying the platform as shown in the command above absolves this error. Before running, the following command may be needed:

``` 
$ docker pull --platform linux/x86_64 mysql 
```

## Starting Web App Container (Django)

To create the container and run the web app:

```
$ docker run -itd --name web-app-1 -v $PWD:/code --cpus 0.5 --memory 512m -p 8080:8000 --workdir /code --net app-net -e DOCKER_CONTAINER_ID=1 python:3.8
```

### Configuring DB via Django
1. Run docker interactive shell for container: ` $ docker exec -it web-app-1 bash `
2. Install Python dependencies: ` pip install -r requirements.txt `
3. Generate migrations for the models listed in `app_events/models.py`: ` python manage.py makemigrations app_events `
4. Apply migrations: ` python manage.py migrate `

### Managing SQL DB within Docker Container
1. Run docker interactive shell for container: ` $ docker exec -it app-db bash `
2. Open mySQL: ` mysql -u root -p root `
3. Confirm that `web_app_db` has been created: ` show databases; `
4. Use the web app's database: ` use web_app_db `
    1. Running `show tables; `, we should see the tables that were created as a result of migration commands run above. Note especially the `app_events_event` table, generated based on the Django model in `app_events/`.
   
### Populating the events table
From Python container:

` # python manage.py populate_events `

- ###TODO :: Custom Django Scripts
   - Look into the `django.core.management.base.BaseCommand` class that's used in [`populate_events.py`](app_events/management/commands/populate_events.py). This allows me to run the above command from the base directory
    
### Running Locally (Section 3, Step 7)
Once the containers are running, start the web server from the `web-app-1` container's interactive shell:

` python manage.py runserver 0.0.0.0:8000`

When executing the command to start the web container, the port was mapped using ` -p 8080:8000 `.

To access the Django application locally, go to `http://0.0.0.0:8080`, which will display a web page listing the Event records.
