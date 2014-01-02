# Overview

This is a build pack provides Zend Server on Cloud Foundry. Current version is 6.1. In the future the web server and Zend Server binaries will be probably be moved to the stack. This will save space for each container and make pushes and restart almost instant. For the time being they are included in the build pack for portability and ease of installation.

## Buildpack components

* Zend Server 6.1 Enterprise edition.
* Zend Server 6.1 configuration files
* PHP 5.4
* Nginx web server
 

# Usage
1. Create a folder on your workstation and "cd" into it.
2. Create an empty file named "zend_server_php_app" (if you don't do this then you'll have to manually specify which buildpack to use for the app). Also make sure your app contains an "index.php" file.
3. Issue the `cf push --buildpack=https://github.com/davidl-zend/zend-server-mysql-buildpack-dev` command (choose to save your manifest when asked to by the cf client). Allocate at least 512M of RAM for your app. 
4. You can optionally bind a mysql service (cleardb/mysql/MariaDB/user-provided) to the app - this will cause Zend Server to operate in cluster mode (experimental). Operating in cluster mode enables: scaling, persistence of settings changed using the gui and persistence of apps deployed using Zend Server's deployment mechanism. Choose to save the manifest when prompted to.
8. Issue the comand below to change the Zend Server GUI password (You can issue in the future in case you forget your password):
`cf set-env <app_name> ZS_ADMIN_PASSWORD <password>`
4. The previous steps should generate a YAML file named "manifest.yml" that looks a like the example below. You can optionally add and push the generated manifest in future applications (in this case cf push will not ask you so many questions).

 ```
 ---
 env:
    ZS_ACCEPT_EULA: 'TRUE'
    ZS_ADMIN_PASSWORD: '<password_for_Zend_Server_GUI_console>'
 applications:
 - name: <app_name>
    instances: 1
    memory: <at least 512M >
    host: <app_name>
    domain: <your_cloud_domain>
    path: .
 ```

5. wait for the app to start.
5. Once the app is started you can access the Zend Server GUI at http://url-to-your-app/ZendServer, For example : http://dave2.vcap.me/ZendServer . If you forgot to perform step 5 then the password for the GUI will be "changeme".
7. If you chose to save the manifest in the previous steps then you can issue the `cf push` to udpate your application code in the future.

#Using an external database service
It is possible to bind an external database to Zend Server app as a "user-provided" service.
Doing so will enable persistence, session clustering and more.
1. Run `cf create-service`.

2. As a service type select "user-provided".
3. Enter a friendly name for the service.
4. Enter service paramaters. the required ones are `hostname, port, password, name`, where "name" is the database Zend Server will use for its internal functions.
5. Enter the paramaters of your external database provider in order.
6. Bind the service to your app- `cf bind-service [service-name] [app-name]`.
7. The service will be auto-detected upon push. Zend Server will create the schema and enable clustering features.


## Known issues
* Code tracing might not work properly in this version.
* Several issues might be encountered if you don't bind mysql providing service to the app (cleardb/mysql/MaraiaDB):
 * You can change settings using the gui and apply them - but they won't survive application pushes and restarts nor will they be propagated to new application instances.
 * Application packages deployed using Zend Server's deployment mechanism (zpk packages) will not be propagated to new app instances.
 * Zend Server will not operate in cluster mode.
* Application generated data is not persistent (this is a limitation of Cloud Foundry) unless saved to a third party storage provider (like S3). 
* Mysql is not used automatically - If you require MySQL then you'll have to setup your own server and configure your app to use it.
* Each container has their own full copy of Zend Server at the moment so droplet size is about 161MB for an empty app (this should not be an issue in public cloud foundry platforms).
* If the application does not contain an index.php file you will most likely encounter a "403 permission denied error".

# Local Installation
You can optionally install the cartridge in yout local cloud foundry environment if you want it to be available as a system build pack. This enables cartridge "auto detection" and adds it to the menu of cartridges presented by the cf utility. If you don't wish to install then skip this stage and go directly to the usage section.

## Requirements
* Working cloud foundry v2 environment with dea_ng and gorouter
* The lucid64 cloud foundry stack should be installed and enabled - check settings in /vagrant/dea_ng/config/dea.yml
* xz compression utility - it's installed automatically by vagrant if you follow the guide below

The buildpack is tested against a system generated by the vagrant installer: http://blog.cloudfoundry.com/2013/06/27/installing-cloud-foundry-on-vagrant/

## Instructions
Clone the git repo into the buildpack directory on your dea_ng node using the command:
`git clone https://github.com/davidl-zend/zend-server-mysql-buildpack-dev`

Alternatively you can clone the repo into a webserver in your environment and specify the buildpack to the cf client. 
f.e  `cf push --buildpack=http://url-to-cloned-repo` or   (you can save this value in the app's manifest.yml).
