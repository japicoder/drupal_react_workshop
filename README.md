# Drupal + React Workshop

An interesting approach to have a headless Drupal working as backend, serving content to a React frontend.

It will install 3 containers (better with traefik so these links are available. Do not access them by IP, use the traefik hostnames):

  * Frontend ("React"): TBD
  * Backend ("Webserver"): 1 Ubuntu 16.04 with Apache and PHP 7.0. Linked to port 80.
    * Traefik will enable it here: http://morecare-mediator.localhost/
  * Backend DB: 1 Mysql 5.7 official container. Linked to port 3306.

Those containers will be linked together and to your host machine, avoiding any cross domain problem as long as they are internally configured too. Please install and run traefik first so the ports are correctly redirected and so you can use the hostnames instead of ips (which won't work).

### Requirements

  * docker
  * docker-compose
  * traefik

### Install and start Traefik
   * Ensure that ports 80, 8080, 8100, 35729, 53703 and 3306 are not in use on your local machine. Or simply just stop any apache or ionic you may have around.
   * Install traefik before you proceed. Follow the instructions at https://wiki.cocomore.com/display/AI/Traefik.io (Docker option 2 is recommended and used through this document. Remember the folder where you download that docker-compose.yml file for future usage). **Project requires Traefik 1.2.3 version to work properly.**
   * To start traefik, if you installed it with docker, use `docker-compose start` and `docker-compose stop` from its folder to run/stop it. (Remember that docker-compose depends on the folder where you run it, and both Morecare-Docker and Traefik folders use it, try not to confuse them.)
   * Traefik will now be available/running at http://localhost:8080. It is needed so we can access the containers with proper URLs, as well as redirecting all their ports to avoid cross-domain problems. If at any point, any container seems not accessible, check traefik is running by entering its url or by `docker ps`.

### Build the project
   * Be sure that traefik is running before each time the project needs to run. To install it, start it or if you have doubts, check previous section.
   * Go to the folder where you cloned this repository.
   * Execute `docker-compose build`
   * Execute `docker-compose up -d`. This should keep this console tied to the containers. It also creates them if any of them is missing. In future attempts, you can use `docker-compose start` instead, as it doesn't lock any console. However, if you use 'start' and any container is missing on `docker ps`, you should better return to use `docker-compose up` command again, to force a reload of containers while being able to see logs directly.
   * If everything worked, on traefik you can also check that the containers are running. Alternatively `docker ps` will display them as well.
   * Now, set/check the right permissions to drw folder:
     * First option: enter inside backend container, executing `docker exec -ti drupalreactworkshop_webserver_1 bash` and:
        * Doing a `ls -la` inside /var/www/drw should reveal the user that owns those files. It should be www-data:www-data.
        * If it isn't, make sure inside the backend container the group www-data exists.
        * Then inside the container: `sudo chown -R www-data:www-data /var/www/drw/`
        * Finally, check permissions: `sudo chmod -R 775 /var/www/drw/`
     * Second option: from outside backend container, enter drw folder and:
        * Doing a `ls -la` inside drw folder should reveal the user that owns those files. It should be your user in both sides XXX:XXX.
        * If it isn't, then execute ``sudo chown -R `whoami`:www-data drw/``.
        * Also, check file permissions: `sudo chmod -R 775 drw/`

You will now have some important directories:
   * drw - maps to /var/www/drw on the container.
   * logs stores the different logs needed for the project.

### Drupal installation
   * To access to the webserver docker, use: ``docker exec -u 1000 -it drupalreactworkshop_webserver_1 bash
``
   * Access to the webserver directory: ``cd /var/www/drw``
   * Update all the project dependencies: ``composer update``

### Database
   * DB name: drw
   * DB user: drw
   * DB pass: drwtestpass
   * DB hostname: db
   
### Drupal console
   In case you're running out with problems when executing `drupal --version`, follow this steps:
   * composer update drupal/console --with-dependencies
   * Install as global: https://docs.drupalconsole.com/en/getting/launcher.html
   
