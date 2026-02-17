## Children Center System (WEB)

### Project prerequisites:
- [Docker Desktop](https://www.docker.com/products/docker-desktop) (Containerization platform)
---
### A. Local Environment Setup
After pulling the repository, go to the project's root directory from the terminal and create a copy of the environment variables template file, like so:
```sh
# copy environment variables file template
cp .env.example .env
```
Next, open your newly created .env file, and add any [required system credentials](https://treasury-dev.atlassian.net/wiki/x/AYBMRg_). The database credentials is already preset for the developers on Docker, so it should not need any modification. (Except maybe for the port)

Now, make sure you save your .env file and run this command under the project directory to initialize and start the project:
```sh
# You can use bash or sh command for this.
sh start
```
**Notes:**
- *The initialization may take some time as this will pull images, build the docker files, and install all composer and node packages required to run the project.*
- *Should there be an error that has something to do with conflicting ports, please change the port in the `docker-compose.yml` file. Replace the left hand of the ports value with a usable port. (e.g. `xxxx:9030`)*  
- *Upon running the the script file, it will create a `.started` file in the root directory. So, if you ran into a problem running the script, make sure to delete the `.started` file first.*

After the initialization, the backend application should already be running under `localhost:3030`. The port may vary depending on what port you have set in the `docker-compose.yml` file.

The frontend can be run with:
```sh
docker exec ccs-web-php npm run dev
```

~~It should run by default under `localhost:5130`. (not port 5173, that's internal)~~

Run `localhost:3030`

#### Interacting with the Docker containers
Since you are using docker to run your project, running PHP, NPM commands or any configuration related commands must be done inside the containers.

**Available container services:** 
- `ccs-web-php` (PHP server: `php`, `composer`, `npm`, `supervisor` | default port: `9030` for PHP, `5130` for frontend)
- `ccs-web-nginx` (web server: `nginx` | default port: `3030`)
- `ccs-mysql` (database server: `mysql` | default port: `4356`)

**Option 1: SSH and run commands in a container:**
```sh
# To list the running containers to check the NAME column for the container name:
docker container ls

# To SSH in the container:
docker exec -it <container_name> sh 

e.g.
docker exec -it ccs-web-php sh
composer install
php artisan route:cache

# To execute a command directly:
docker exec <container_name> <your_command_here>

e.g.
docker exec ccs-web-php php artisan tinker
docker exec ccs-web-php cat .env | grep '^DB_'
```
**Option 2: Stop/start the running containers:**
```sh
docker compose stop

# To start the containers again:
docker compose start
```
**Option 3: Uninstall/remove the containers:**
```
docker compose down
```
**Note:**
*You can initialize the containers again by running the `start` script file again.*

**Option 4: Modify configuration files:**
- Supervisor worker: `./scripts/docker/php/supervisor/ccs-web-worker.conf`
- Nginx main config: `./scripts/docker/nginx/nginx.conf`
- Nginx server config: `./scripts/docker/nginx/default.conf`
- Dockerfile: `./scripts/docker/Dockerfile.dev`
- Docker Composer: `./docker-compose.yml`

---

### B. XDebug Debugger Setup (PHP)
#### PHP Configuration
In the scripts folder look for a file under docker php called `php.ini`.

Once opened it should look like so:
```ini
[xdebug]
xdebug.client_host = host.docker.internal
xdebug.mode = debug
xdebug.start_with_request = yes
xdebug.discover_client_host = 0
xdebug.remote_handler = "dbgp"
xdebug.client_port = 9021
xdebug.log_level = 0
```

As shown on the template above, depending on the ports available in your host, the `client_port` has to be changed. In the example, if port `9021` is not being used anywhere in your host, you can leave it as is.

#### Install the PHP Debug Extension in VS Code
To integrate the debugger to your code editor, an extension is necessary. 

In this guide, we are using VS Code but it can also be done on PHPStorm, it's probably using a different extension and a different configuration schema as well to communicate to XDebug.

In general, the configuration file should be doing the same idea.

Now go to the extensions on the side navigation panel, look for `PHP Debug by XDebug`. Once installed, we can move to the next step.

#### Configure launch configuration of XDebug
To place our configuration file, we will need to create a new folder under the name of `.vscode`.

Under the same scripts folder again, look for the vscode folder and copy the `launch.json` file. It should look like so:

```json
{
    "version": "0.2.0",
    "configurations": [
    {
      "name": "Listen for Docker XDebug",
      "type": "php",
      "request": "launch",
      "port": 9021,
      "pathMappings": {
        "/var/www/html": "/path_to_host_project_directory"
      }
    }
  ]
}
```

There are 2 things that need to be changed here:

- The port number needs to be the same as the one set in the `php.ini` file.
- Change the `pathMappings` to the absolute path of your project folder. Only modify the `value` and not the `key` of the key-value pair.

*Note: You can easily get your project directory via `pwd` command in the VS code terminal of this project.*

#### Run the Debugger (Step Debugging)
You're all set. If the application is already running, just hit the debug button (▶ icon) on the sidebar navigation on the left side, then hit another Debug button on the top part to start the debugger.

This should detect the `launch.json` file you have created, run the debugger, and show a debug controller on the top right side of the IDE. Refreshing your app (localhost:3030) on the browser should trigger a breakpoint and pauses the request as the scope is initially set to `Everything`.

On trigger, you should see the `Variables` and the `Call Stack` dropdown being filled in to see what's going on.

Alright, you can now set your `breakpoints` to any PHP class and start debugging!

**Unfamiliar with breakpoint debugging? Here's a [guide](https://dev.to/phpcontrols/debugging-php-with-vscode-and-xdebug-a-step-by-step-guide-4296).**

>
>*"Using a breakpoint debugger will save you more time than spamming hardcoded logs around your logic; it's the old fashion way, but it's the right way to debug."*
>
> **—Gabe Newell**, probably
>

*Note: You might come across a payload error with the Laravel decrypter, make sure to uncheck the `Everything` checkbox under Breakpoints dropdown to only debug the ones you have set breakpoints on.*

XDebug is not only limited to breakpoint debugging, it can also do the following if you're interested:
- Profiling (Performance)
- Function Tracing
- Improved Error Reporting
- Code Coverage Analysis (Unit Testing)


**Additional:**
On top of this, a **[Laravel Debugbar](https://laraveldebugbar.com/)** package is already installed for the project. Check that one out, it's very helpful for the development, especially on backend development. 

---

#### *Cheers!* 🍺
***It's now time to develop new features and fix some bugs!***


*Help us keep this README file up-to-date! Should you find any inconsistencies or things that need to be added, feel free to do so. Thanks!*