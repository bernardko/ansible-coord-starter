# A Starter Ansible Project for Django Deployment using the ansible-coord Collection.

The purpose of this repository is demonstrate how to use the [ansible-coord](https://github.com/bernardko/ansible-coord) collection to store the multi-environment configuration for multiple django projects utilizing the same playbooks for deployment.

Ansible is powerful and provide a multitude of ways to configure your project. This repository aims to focus and streamline this into a more simplified workflow providing clean starter project with tools to fully deploy django and celery workers which meant for further customization and building on top of.

## Features

- Configure and deploy Django projects using pipenv, supervisor and uvicorn.
- Use any python version with pyenv.
- Easy configuration of Celery workers.
- PostgreSQL, user and database configuration.
- RabbitMQ vhost and user configuration.
- Configure memcache instances.
- Custom nginx configuration.
- Vagrant for testing your playbooks.

The basic structure of how to the configuration is stored looks like the following:

```
coord-starter
|
├── inventory1          <-- inventory for project 1
|   ├── files           <-- additional custom files 
|   └── vagrant         <-- inventory folder
|       ├── group_vars
|       |   └── all.yml <-- general configuration
|       └── inventory   <-- host specific configuration
|   ├── vagrant.vault   <-- secrets vault for login credentials
|   ├── production
|   ├── production.vault
|   ├── staging
|   └── staging.vault
|
├── inventory2         <-- inventory for project 2
|   ├── files
|   └── vagrant
|       ├── group_vars
|       |   └── all.yml
|       └── inventory
|   ├── vagrant.vault
|   ├── production
|   ├── production.vault
|   ├── staging
|   └── staging.vault
|   
├── play                <-- simplified run script
...
```

Multiple inventory folders are stored at the root of the project and multiple environment configuration is stored inside.

An example of the intial command to setup services and deploy the project is the following:

```bash
play test-inventory local setup
```
And iterative deployments will use the following command to reduce work and speed up the deployment:

```bash
play test-inventory local deploy
```
The commands are idempotent so feel free to use whichever is more suitable.

The simple script `play` is used to collect together parameters which runs the `ansible-playbook` command with all the relevent parameters and just easier and faster to type on command line.

## Why Multiple Projects?

This is the starter ansible playbook I use when I need to automate my deployments. This allows me keep relevant projects together, deploy them in a unified workflow and allowing me to customize how they are deployed and interact with each other. Also it is a flexible structure allowing for growth when needed. Of course, this project layout is clean enough that it also works for single projects as well.

## Getting Started

There is a `test-inventory` which is already configured to deploy [test-django-project repository](https://github.com/bernardko/test-django-project) which is a blank django project generated with my [django-project-template repository](https://github.com/bernardko/django-project-template) to make it very easy for anyone to try this out via:

- Vagrant (recommended)
- Deploy to localhost

First let's make sure you have ansible installed.

```bash
pip install ansible
```

Setup the basics for this playbook:

```bash
git clone https://github.com/bernardko/coord-starter.git
cd coord-starter
ansible-galaxy install -r requirements.yml
```

### Test this out using Vagrant

This is the quickest way to see this ansible playbook in action and only requires [Vagrant to be installed](https://www.vagrantup.com/docs/installation) on your system. Just run:

```bash
vagrant up
```

### Test this out by deploying to localhost

You will need to the following edit the following in the `coord-starter/test-inventory/local.vault` file and enter a user account with sudo access:

```yaml
ansible_user: username
ansible_become_pass: mypassword
```

You'll also need to make sure you can `SSH` into this account via localhost:
```bash
ssh username@localhost
```

If this works, then you are ready to go, run the following get deploy:

```bash
./play test-inventory local setup
```

## Customize for your own project

The best way to get started using this is to copy `test-inventory` and start customizing the settings to your own project. Make sure you explore the `siplay/playbooks` to how the playbooks and roles are structured. Also check out the `play` script to see how commands are run.  

Remember to encrypt the secrets vault:
```bash
ansible-vault encrypt test-inventory vagrant.vault
```

## play Script Commands

Start and stop celery and uvicorn processes
```bash
./play test-inventory local processes start
./play test-inventory local processes stop
./play test-inventory local processes restart
```

### Clean up commands
Configuration files for nginx, celery, uvicorn can be easily removed by adding `remove: yes` attribute to the configuration. When you run the `deploy` command again the tasks will cleanup those files for you.

There is also a playbook `bernardko.coord.delete` which deletes all configuration and data from all hosts listed under `deleteservers` group in your inventory. Great for migrating servers and cleaning them up.