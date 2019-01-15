# Dockerized Jupyter Lab with Selenium
*Author: Douglas Strodtman (SaMo)*

This directory contains everything you need to get Selenium up and running in Google Chrome. This will allow you to quickly create identical environments for automating tasks and scraping Javascript enabled pages, and make it easy to run Selenium on AWS.

If you're running in AWS, you'll want to make sure you install Docker and docker-compose as you initialized. Use the following code in the '**Advanced Details**' within the **Configure Instance Details** tab (Step 3):

```
#!/bin/bash
curl -sSL https://get.docker.com/ | sh
usermod -aG docker ubuntu
curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

Full details on my recommended AWS configurations can be found [here](https://git.generalassemb.ly/DSI-US-6/SM-Flex-6/wiki/Creating-an-AWS-instance).

## Files Contained
Other than this README, each of the files in this directory is necessary to successfully build out your environment. Hopefully the descriptions below will allow you to update these to your specific needs.

### Dockerfile
Dockerfiles allow you to define custom Docker images. This file inherits the CURRENT `jupyter/scipy-notebook` image and then pip installs a number of additional packages through the `requirements.txt` file. Note that currently the versions are not specified for any of these files--if you run into version errors, you may want to update these specifications.

### requirements.txt
This file is essentially just a list of packages to be pip installed. In the current iteration, versions are not specified, and the following packages are appended to those included in the `jupyter/scipy-notebook` image [(base packages here)](https://jupyter-docker-stacks.readthedocs.io/en/latest/using/selecting.html#jupyter-scipy-notebook):
- selenium
- scrapy
- psycopg2

### docker-compose.yml
This file does all the magic. Chrome, Selenium, and Jupyter are each installed in separate Docker containers that are networked to one another. The Jupyter image is custom-built from the included Dockerfile. Note that current versions of the Selenium Docker images are hard-coded (to ensure compatability) and can be updated later. [Here's Selenium's official docker-compose instructions, in case you break this](https://github.com/SeleniumHQ/docker-selenium/wiki/Getting-Started-with-Docker-Compose).

Note that a number of custom options are being specified here:
- The Jupyer container name is set as `jupyter_selenium`.
- The base directory for the Jupyter container is set to the computer's home directory (`~/`).
- Jupyter Lab is enabled (this will open Lab by default and prevent you from opening Jupyter Notebook; you can comment out this option to revert to Jupyter notebooks).
- The Jupyter instance is ported to `8888` (if you're trying to run this alongside another Jupyter instance, you'll need to change this port to avoid conflicts).


## Instructions for Use
To set up all 3 of these containers, you'll simply need to navigate to this directory and type 
```
docker-compose up -d
```
This method will also contain an internal network linking the containers; the IP address of each is mapped to the service name.

Specifically, this is important because the IP address of the Selenium hub container is mapped to the alias `hub`, which allows us to therefore access our remote Selenium instance through the web address `http://hub:4444/wd/hub`.

## Access Jupyter Lab
Run
```
docker exec jupyter_selenium jupyter notebook list
```
to display the token of your currently running lab instance. If running on AWS, you'll want to sub the public IP address of your instance for the localhost.

## Remote Selenium
The following code demonstrates how easy it is to use a remote webdriver. Here we specify that we want to use a headless Chrome browser (a browser without a visual display) for our driver, which is hosted on port `4444` of our `hub` service:
```
from selenium import webdriver
from selenium.webdriver.common.desired_capabilities import DesiredCapabilities

options = webdriver.ChromeOptions()
options.add_argument('headless')
driver = webdriver.Remote(
    command_executor='http://hub:4444/wd/hub',
    desired_capabilities=DesiredCapabilities.CHROME)
```
If this is your first time, I recommend doing a simple test. Here, we'll go to Google and just print the title. **Don't forget to close your driver after completing your task.**
```
driver.get('https://www.google.com')
print(driver.title)
driver.quit()
```

## Stopping
To stop and remove these instances (which will not delete your files or images, only the containers), simply run
```
docker-compose down
```
