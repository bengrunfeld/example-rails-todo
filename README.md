# Ruby on Rails Quick Start Guide

This guide will walk you through deploying a Ruby on Rails stack to Amazon EC2 using OpDemand.

## Prerequisites

* An [OpDemand account](http://www.opdemand.com/) that is [linked to your GitHub account](http://www.opdemand.com/docs/about-github-integration/)
* An [OpDemand environment](http://www.opdemand.com/how-it-works/) that contains valid [AWS credentials](http://www.opdemand.com/docs/adding-aws-creds/)

## Setup your workstation

* Install [Ruby](http://www.ruby-lang.org/en/downloads/) if you haven't already (we recommend Ruby 1.9.3)
* Install [RubyGems](http://rubygems.org/pages/download) to get the `gem` command on your workstation
* Install [Foreman](http://ddollar.github.com/foreman/) with `gem install foreman`

## Clone your Application

If you want to use an existing application, no problem.  You can also fork OpDemand's sample application located at <https://github.com/opdemand/example-rails-todo>.  After forking the project, clone it to your local workstation using the SSH-style URL:

	$ git clone git@github.com:mygithubuser/example-rails-todo.git
    $ cd example-rails-todo

## Prepare your Application

To use a Rails application with OpDemand, you will need to conform to 3 basic requirements:

 1. Use [Bundler](http://gembundler.com/) to manage dependencies
 2. Use [Foreman](http://ddollar.github.com/foreman/) to manage processes
 3. Use [Environment Variables](https://help.ubuntu.com/community/EnvironmentVariables) to manage configuration inside your application

If you're deploying the example application, it already conforms to these requirements.

#### 1. Use Bundler to manage dependencies

Every time you deploy, OpDemand will run a `bundle --deployment` on all application instances to ensure dependencies are up to date.  Bundler requires that you explicitly declare your dependencies using a [Gemfile](http://gembundler.com/v1.3/gemfile.html).  Here is a [basic example](https://github.com/opdemand/example-rails-todo/blob/master/Gemfile).

Install your dependencies on your local workstation using `bundle install`:

    $ bundle install
    Fetching gem metadata from http://rubygems.org/..........
    Fetching gem metadata from http://rubygems.org/..
    Resolving dependencies...
    Using rake (10.0.4) 
    Using i18n (0.6.1) 
    Using multi_json (1.7.2) 
    Using activesupport (3.2.13) 
    ...
    Using unicorn (4.6.2)
    Your bundle is complete!
    Use `bundle show [gemname]` to see where a bundled gem is installed.

If your dependencies require any system packages, you can install those later by specifying a list of custom packages in the Instance configuration or by customizing the deploy script to install your own packages.

#### 2. Use Foreman to manage processes

OpDemand uses [Foreman](http://ddollar.github.com/foreman/) to manage the processes that serve up your application.  Foreman relies on a `Procfile` that lives in the root of your repository.  This is where you define the command(s) used to run your application.  Here is an example `Procfile`:

    web: bundle exec unicorn_rails -p $PORT

This tells OpDemand to run web application workers using the command `unicorn_rails -p $PORT` wrapped in a `bundle exec` (highly recommended).  You can test this locally by running `foreman start`.

    $ foreman start
    14:21:05 web.1  | started with pid 84614
    14:21:06 web.1  | I, [2013-04-06T14:21:06.563234 #84614]  INFO -- : listening on addr=0.0.0.0:5000 fd=11
    14:21:06 web.1  | I, [2013-04-06T14:21:06.563375 #84614]  INFO -- : worker=0 spawning...
    14:21:06 web.1  | I, [2013-04-06T14:21:06.565020 #84614]  INFO -- : master process ready
    14:21:06 web.1  | I, [2013-04-06T14:21:06.565858 #84634]  INFO -- : worker=0 spawned pid=84634
    14:21:06 web.1  | I, [2013-04-06T14:21:06.566155 #84634]  INFO -- : Refreshing Gem list
    14:21:08 web.1  | I, [2013-04-06T14:21:08.173211 #84634]  INFO -- : worker=0 ready

You should now be able to access your application locally at <http://localhost:5000>.

#### 3. Use Environment Variables to manage configuration

OpDemand uses environment variables to manage your application's configuration.  For example, your application listener must use the value of the `PORT` environment variable. The same is true for external services like databases, caches and queues.  Here is an example of how to connect to a Memcached service using environment variables:

    config.cache_store = :mem_cache_store, ENV["MEMCACHED_HOST"] || "localhost"

## Add a Rails Stack to your Environment

We now have an application that is ready for deployment, along with an [OpDemand environment](http://www.opdemand.com/how-it-works/) that includes [AWS credentials](http://www.opdemand.com/docs/adding-aws-creds/).  Let's add a basic Rails stack to host our example application:

* Click the **Add/Discover Services** button
* Select the **Rails** stack and press **Save**

A typical Rails application stack includes:

* An **EC2 Load Balancer** used to route traffic to your EC2 instances
* An **EC2 Instance** used to host the application behind [Nginx](http://wiki.nginx.org/Main)
* An **RDS Instance** used as the external MySQL database for Rails
* An **RDS Security Group** used as a virtual firewall for your RDS instance
* An **EC2 Security Group** used as a virtual firewall inside EC2
* An **EC2 Key Pair** used for deployment automation

## Deploy the Environment

To deploy this application stack, press the green deploy button on the environment toolbar.

![Deploy your environment](http://www.opdemand.com/wp-content/uploads/2013/03/Screen-Shot-2013-03-27-at-1.04.35-PM.png)

### Specify Required Configuration

OpDemand provides reasonable defaults, but you'll want to review a few configuration values:

* Check the default *Regions*, *Zones* and *Instance Types*
* Add your public key to *SSH Authorized Keys* so you can SSH into Instances
* Make sure the *Repository URL* and *Repository Revision* are correct for your application
* If your app is in a private GitHub repository, click **Create Deploy Key** to have OpDemand install a secure deploy key using the GitHub API
* Set the *Database Username*, *Database Password* and *Database Name* for RDS

Once you've reviewed and modified the required configuration, press **Save & Continue** to initiate your first deploy.

### Wait until Active

OpDemand will now orchestrate the deployment of your application stack to your cloud providers.  Once the environment has an **Active** status, your application should be good to go.

This can take a while depending on the cloud provider, service types, instance sizes and the build/deploy scripts (are you compiling something?).  While you wait, grab some coffee and:

* Watch the Key Pairs and Security Groups build, deploy and become **Active**
* Watch the RDS Instance build, deploy and become **Active** (this can take up to 10 min)
* Watch the Instances build, deploy and become **Active** (this takes a few minutes too, check out the real-time log feedback)
* Watch the Load Balancers build, deploy and become **Active**

### Troubleshooting

It's not uncommon to experience errors or warnings during deploys.  If you get stuck on an error you can click **Report This** to [open a ticket](https://desk.opdemand.com/) with the OpDemand help desk.

* For *Cloud Provider Errors*, check the service's primary configuration fields
* For *SSH Key Warnings*, make sure Deployment configuration sections contain valid SSH private keys
* For *SSH Return Code Warnings*, SSH into the instance and make sure the Build & Deploy scripts execute successfully
* For *Other Warnings*, try re-deploying to bring the service back to active status

###### SSH Access

Click the **SSH** button on the toolbar to SSH into Instances.  If you didn't add your SSH key initially, you can always modify SSH keys later, save the new configuration and **Deploy** again to update the Instance.

![SSH into your Instance](http://www.opdemand.com/wp-content/uploads/2013/03/Screen-Shot-2013-03-27-at-1.10.19-PM.png)

###### Database Access

To get a shell on the RDS database, SSH into one of the application Instances and use the following:

    $ source .opdemand                             # read in env vars for MYSQL
    $ mysql -h $MYSQL_HOST -u $MYSQL_USERNAME -p   # requires "mysql-client" package
    Password:                                      # enter password manually

Alternatively you can open the RDS firewall to the world and use a MySQL client on your local workstation.

## Access your Application

Once your application is active, you can access its [published URLs](http://www.opdemand.com/how-it-works/monitor/) on the Environment's **Monitor** tab.  If you're looking at a service that publishes something, you can jump to the published URL in the upper-right corner of the service:

![Access your application](http://www.opdemand.com/wp-content/uploads/2013/03/Screen-Shot-2013-03-27-at-2.43.09-PM.png)

For the example application you should see: *Powered by OpDemand*

## Update your Application

As you make changes to your application or deployment code:

1. **Push** the code to GitHub
2. **Deploy** the environment

OpDemand will use the latest environment configuration to update cloud services, SSH into instances, pull down source code from GitHub, install dependencies, re-package your application and restart services where necessary.

If you want to integrate OpDemand into your command-line workflow, `opdemand deploy` can also be used to trigger deploys.  See [Using the OpDemand Command-Line Interface](http://www.opdemand.com/docs/) more details.

## Additional Resources

* [OpDemand Documentation](http://www.opdemand.com/docs/)
* [OpDemand - How It Works](https://www.opdemand.com/how-it-works/)