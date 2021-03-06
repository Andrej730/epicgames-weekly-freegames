# Automatically get the epicgames store weekly free games
I like free games but I don't like repeating the same process over and over again when it can be automated... And that's why I made this!

## What it is
This is a simple python3.7 script making use of [selenium webdriver](https://selenium.dev/) and [chrome driver](https://sites.google.com/a/chromium.org/chromedriver/) to run run chrome in headless mode, navigate to the epicgames store, login into your account and redeem all weekly free games available. All of this inside of a docker container.

## Getting started
Personally I avoid hosting such processes on my machine so I don't have to worry about having it turned ON during the times when it's supposed to run. So I opt for deploying it into AWS ECS and completely forget it exists... until I check my account on the website that is! ;)

To deploy it to AWS ECS start by creating a ECR repo and copying its URL to the makefile so it looks like this:
```
DOCKER_IMAGE_TAG ?= "epicgames:latest"
ECR_URL ?= "111111111111.dkr.ecr.us-west-2.amazonaws.com"
```
And then run:
```
make deployimage
```
Now you can create a new ECS task using your newly uploaded image, manually or through a cloudwatch event (recommended).

But you can also choose to run it locally, to do that start by building the image:
```
docker build -t epicgames .
```
And then run a docker container from the newly built image:
```
docker run -e EMAIL=<EMAIL> -e PASSWORD=<PASSWORD> epicgames
```
Replacing the environment variables `EMAIL` and `PASSWORD` for your epicgames store credentials.

## Optional configuration
Different options can be provided to alter the output or execution of the program. All available options are:

- `TIMEOUT` and `LOGIN_TIMEOUT` if you have a slow internet connection speed and want to make sure it won't affect the result of the script. Defaults to `TIMEOUT = 5` and `LOGIN_TIMEOUT = 10`.

- `LOGLEVEL` can be used to specify the [log level](https://docs.python.org/3.7/library/logging.html#logging-levels) to log. Defaults to `INFO`

- `SLEEPTIME` can be used to set the number of seconds to wait between looping through the procedure again. Defaults to `-1`, which will stop execution after a single iteration.

- `TOTP` is used to log in to an account protected with 2FA. If you have 2FA enabled already, you may have to redo it in order for to obtain your TOTP token. Go [here](https://www.epicgames.com/account/password) to enable 2FA. Click "enable authenticator app." In the section labeled "manual entry key," copy the key. Then use your authenticator app to add scan the QR code. Activate 2FA by completing the form and clicking activate. Once 2FA is enabled, use the key you copied as the value for the `TOTP` parameter.

- `DEBUG` allows the script to be run locally without having to manually edit the chromedriver path (instead will searches in the current dir) and will ignore the headless flag when starting up chrome. Only checks for the presence of this environment variable, its value is not interpreted.

Running a docker container with these:
```
docker run -e TIMEOUT=15 -e LOGIN_TIMEOUT=20 -e LOGLEVEL=DEBUG -e SLEEPTIME=43200 -e EMAIL=<EMAIL> -e PASSWORD=<PASSWORD> epicgames
```

## Docker Compose
```
version: '2'

services:

    egs-freegame:
        build:
            context: ./epicgames-weekly-freegames/
            dockerfile: Dockerfile
        restart: always
        environment:
            - TIMEOUT=10
            - LOGIN_TIMEOUT=15
            - SLEEPTIME=43200
            - LOGLEVEL=DEBUG
            - EMAIL=example@example.com
            - PASSWORD=password123
```
