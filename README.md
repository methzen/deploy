 NB for certbot installation phase see here https://certbot.eff.org/instructions?ws=nginx&os=ubuntufocal
# How To Deploy A Next.js Website To A DigitalOcean Server

Are you looking for a way to deploy your Next.js website or application?

In this article, we'll walk you through how to deploy a production-ready [Next.js](https://nextjs.org/) website to a [DigitalOcean](https://m.do.co/c/ce20017d8588) server.

Through that process, you'll create a new VPS ([Virtual Private Server](https://en.wikipedia.org/wiki/Virtual_private_server)), configure your server to host [Node.js](https://nodejs.org/) applications, configure your domain name, use a [Nginx](https://www.nginx.com/) web server as a reverse-proxy in front of your Next.js application, and setup HTTPS/SSL encryption so your website is served with `https://`.

Before you begin this guide you'll need to have Node.js and NPM installed on your local development machine. We created an [installation guide](https://coderrocketfuel.com/article/guide-to-installing-nodejs-on-windows-linux-or-macos) if you need help getting those installed on your machine.

The assumption is that you already have a Next.js application built and ready to deploy. If that's not the case, check out our [article](https://coderrocketfuel.com/article/how-to-create-a-website-with-next-js-and-react) on creating a Next.js website.

Once you've got those two things covered, let's jump right in!

#### Table Of Contents

-   [Create & Configure A DigitalOcean Server](https://coderrocketfuel.com/article/how-to-deploy-a-next-js-website-to-a-digital-ocean-server#create-and-configure-a-digitalocean-server)
-   [Configure A Domain Name](https://coderrocketfuel.com/article/how-to-deploy-a-next-js-website-to-a-digital-ocean-server#configure-a-domain-name)
-   [Install & Configure Nginx](https://coderrocketfuel.com/article/how-to-deploy-a-next-js-website-to-a-digital-ocean-server#install-and-configure-nginx)
-   [Install Node.js](https://coderrocketfuel.com/article/how-to-deploy-a-next-js-website-to-a-digital-ocean-server#install-node-js)
-   [Deploy The Next.js Application](https://coderrocketfuel.com/article/how-to-deploy-a-next-js-website-to-a-digital-ocean-server#deploy-the-next-js-application)
-   [Configure Nginx As A Reverse Proxy](https://coderrocketfuel.com/article/how-to-deploy-a-next-js-website-to-a-digital-ocean-server#configure-nginx-as-a-reverse-proxy)
-   [Configure HTTPS/SSL Encryption](https://coderrocketfuel.com/article/how-to-deploy-a-next-js-website-to-a-digital-ocean-server#configure-https-ssl-encryption)

## Step 1 - Create & Configure A DigitalOcean Server

Before anything else, we need to create and configure the server in the cloud using [DigitalOcean](https://m.do.co/c/ce20017d8588).

To start, you'll need to create an account on [DigitalOcean](https://m.do.co/c/ce20017d8588) or log in to your existing account.

If you're going to sign up for a new account and want a free $200 credit for 60 days, use this referral link to signup: [https://m.do.co/c/ce20017d8588](https://m.do.co/c/ce20017d8588).

They will ask you for a credit card, but you can cancel anytime before the 60 day period ends without being charged.

### Create A New Droplet On DigitalOcean

After logging in or successfully signing up for a new account, open the **Create** drop-down menu and click the **Droplets** link.

This will take you to the **Create Droplets** page where you'll be given some configuration options before creating a new server.

![DigitalOcean Create Droplet Page](https://assets.coderrocketfuel.com/coding-blog-create-digitalocean-droplet-page.png)

In the first section, select the Ubuntu operating system for your server.

![DigitalOcean Create Droplet Page - Choose Operating System](https://assets.coderrocketfuel.com/coding-blog-create-digitalocean-droplet-operating-system.png)

Then, choose the $5 per month standard plan, which will give your application plenty of computing power to start with. You can easily upgrade in the future if needed.

![DigitalOcean Create Droplet Page - Choose Pricing Plan](https://assets.coderrocketfuel.com/coding-blog-create-digitalocean-droplet-choose-plan.png)

Next, they allow you to choose the data center region for your server. This is the physical location for your server and, therefore, you should choose the one closest to the people visiting your website.

![DigitalOcean Create Droplet Page - Choose Data Center Region](https://assets.coderrocketfuel.com/coding-blog-create-digitalocean-droplet-datacenter-region.png)

In the **Authentication** section, make sure the **Password** option is selected and enter a strong root password for your server. This is the password we'll use to initially SSH into your server.

![DigitalOcean Create Droplet Page - Authenctication Section](https://assets.coderrocketfuel.com/coding-blog-create-digitalocean-droplet-root-password.png)

Make sure you save this password because you'll need it to gain initial access to your server.

Also, you can choose a hostname for your server. This will give your server a name to remember it by.

![DigitalOcean Create Droplet Page - Choose a Name For the Server](https://assets.coderrocketfuel.com/coding-blog-create-digitalocean-droplet-hostname.png)

When you're done selecting options, hit the **Create Droplet** button to kick off the creation of your new server.

![DigitalOcean Create Droplet Page - Create the Droplet Button](https://assets.coderrocketfuel.com/coding-blog-create-digitalocean-droplet-submit-button.png)

When the Droplet is fully up and running, the control panel will display the server's IP address.

![DigitalOcean Droplet Dashboard Page](https://assets.coderrocketfuel.com/coding-blog-create-digitalocean-droplet-success.png)

Your server is now up and running!

In the next step, we'll complete the initial configuration process for the server. This will include logging into the server, setting up SSH access to the server, and creating a basic firewall.

### Configure Your New Server Droplet

Right now, your server is a blank machine humming along on a server rack in your geographic region of choice.

In this section, we'll walk through how to gain remote access to the server and set up some basic configuration to help keep it secure before we deploy and run your Next.js application on it.

Via the terminal, we'll be switching back and forth from the server we create and your local machine. So, pay close attention to what environment you're in before running commands throughout this article.

Let's get started!

#### Access The Server Using Root

The first step is to gain access to the server using your root login.

To do this, you'll need both the IP address of the server and the private key (password) for the root user's account you created in the last section.

To log into your server, open a terminal on your local machine and use the following command to SSH in as the root user (insert your server's IP address in place of `server_ip_address`):

Command

```bash
ssh root@server_ip_address

```

If a warning about host authenticity appears, accept it.

Then, provide the root password you created.

After you've successfully logged in, you should be remotely inside your DigitalOcean server.

The root user in a Linux environment has very broad privileges and, for that reason, you're discouraged from using it regularly. This is because very destructive changes (even by accident) can be made while using it.

Therefore, in the next step, we're going to create an alternative account with limited scope that will be used for daily tasks.

#### Create A New User

Logged in as root, we can create a new user account that will be used to log in from this point forward.

You can create a new user with the following command (insert a username you want in place of `bob`):

Command

```bash
adduser bob

```

You'll be asked some questions starting with the password. Make sure you choose a strong password.

The rest of the question prompts are optional. You can just hit `ENTER` repeatedly to skip them if you wish.

You now have a new user account with regular account privileges. But you might occasionally need to do administrative tasks that require root privileges. So, instead of logging out of your normal user and logging back in as the root account, we can give the normal account the ability to run root privileged commands when you need to by adding `sudo` before each command.

To do this, we need to add your new user to the `sudo` group on the machine.

As root, run the following command to add your user to the `sudo` group (replace `bob` with your username):

Command

```bash
usermod -aG sudo bob

```

Now your user can run commands with root privileges!

The next initial server setup steps will help increase the security of your server. **They are optional but highly recommended**.

#### Add Public Key Authentication

By setting up public-key authentication for the new user, it will increase your server's security by requiring a private SSH key to login in instead of anyone being able to access the server via password.

Let's get pubic key authentication configured.

##### Generate A Key Pair On Your Local Machine

If you don't already have an SSH key pair on your **local machine**, which consists of a public and private key, you need to generate one. If you already have a key that you want to use, skip to the [Copy the Public Key](https://coderrocketfuel.com/article/how-to-deploy-a-next-js-website-to-a-digital-ocean-server#copy-the-public-key) step below.

To generate a new key pair, enter the following command at the terminal of your **local machine**:

Command

```bash
ssh-keygen

```

You'll receive an output similar to the following:

Output

```bash
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/yourusername/.ssh/id_rsa):

```

Press `ENTER` to accept the file name and path.

Next, you'll be prompted to enter a password to secure the newly created key. You can either create a password or leave it blank. This generates a private key, `id_rsa`, and a public key, `id_rsa.pub`, in the `/.ssh` directory inside your home directory.

##### Copy The Public Key

Now that you have the SSH key pair configured on your local machine, you need to copy your public key to the server.

Use one of the two options below to do this.

###### Option 1: Use SSH-Copy-Id

If your local machine has the `ssh-copy-id` script installed, you can use it to install your public key for any user that you have login credentials for. If your machine doesn't have the script, use the second option to install the key manually instead.

Still on your **local machine**, type the following command (replace your username and IP address for the values below):

Command

```bash
ssh-copy-id bob@server_ip_address

```

You'll be asked for the user's password.

This is the password for the user you created on your server. Not the root password.

After successfully authenticating, your public key will be added to the server user's `.ssh/authorized_keys` file. The corresponding private key can now be used to log into the server.

###### Option 2: Install The Key Manually

Assuming you generated an SSH key pair using the previous step, use the following command at the terminal of your **local machine** to print and display your public key (`id_rsa.pub`):

Command

```bash
cat ~/.ssh/id_rsa.pub

```

This should print your public SSH key, which should look something like the following:

Output

```bash
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDBGTO0tsVejssuaYR5R3Y/i73SppJAhme1dH7W2c47d4gOqB4izP0+fRLfvbz/tnXFz4iOP/H6eCV05hqUhF+KYRxt9Y8tVMrpDZR2l75o6+xSbUOMu6xN+uVF0T9XzKcxmzTmnV7Na5up3QM3DoSRYX/EP3utr2+zAqpJIfKPLdA74w7g56oYWI9blpnpzxkEd3edVJOivUkpZ4JoenWManvIaSdMTJXMy3MtlQhva+j9CgguyVbUkdzK9KKEuah+pFZvaugtebsU+bllPTB0nlXGIJk98Ie9ZtxuY3nCKneB+KjKiXrAvXUPCI9mWkYS/1rggpFmu3HbXBnWSUdf localuser@machine.local

```

Select the public key in the terminal output and copy it to your clipboard.

To enable the use of the SSH key to authenticate as the new remote user, you must add the public key to a special file in the user's home directory.

Back on your **DigitalOcean server**, as the root user, enter the following command to temporarily switch to the new user (substitute your username in place of `bob`):

Command

```bash
su - bob

```

Now you'll be in your new user's home directory.

Create a new directory called `/.ssh`:

Command

```bash
mkdir ~/.ssh

```

And restrict its permissions with this command:

Command

```bash
chmod 700 ~/.ssh

```

Now open a file in `.ssh` called `authorized_keys` with a text editor. We will use `nano` to edit the file:

Command

```bash
nano ~/.ssh/authorized_keys

```

Now insert the public key you generated and copied by pasting it into the `nano` editor.

Hit `CTRL`+`X` to exit the file, then `Y` to save the changes that you made, then `ENTER` to confirm the file name.

Now restrict the permissions of the `authorized_keys` file with this command:

Command

```bash
chmod 600 ~/.ssh/authorized_keys

```

Type this command once to return to the root user:

Command

```bash
exit

```

Now your public key is installed, and you can use SSH keys to log in as your user.

#### Disable Password Authentication

This step will configure your DigitalOcean server to only allow logins using the SSH key you just created. Therefore, only people who possess the private key that pairs with the public key that was installed will get into the server. This increases your server's security by disabling password-only authentication.

Only follow this step if you installed a public key in the last step. Otherwise, you'll lock yourself out of the server.

To disable password authentication, follow the proceeding steps.

As the root user or new sudo user on your **DigitalOcean server**, open the SSH daemon configuration file using the following command:

Command

```bash
sudo nano /etc/ssh/sshd_config

```

Find the line that says `PasswordAuthentication` (using arrow keyboard keys) and change its value from `yes` to `no`. It should look like this after the change was made:

/etc/ssh/sshd_config

```text
PasswordAuthentication no

```

Save and close the file using this method: `CTRL-X`, then `Y`, then `ENTER`.

To reload the SSH daemon and put the changes live, execute the following command:

Command

```bash
sudo systemctl reload sshd

```

Password authentication is now disabled. Now your server can only be accessed with SSH key authentication.

Let's test logging in using the SSH key.

On your **local machine**, log in to your server using the new account that we created. Use the following command to do so (substitute your username and IP address):

Command

```bash
ssh bob@server_ip_address

```

Once authentication is provided to the server, you'll be logged in as your new user.

Now your server is only accessible by someone who has the SSH keys you generated. Which, in this case, are only located on your local machine. This greatly improves the security of your server.

#### Add a Basic Firewall

Another security improvement we can make to the server is to add a basic [firewall](https://en.wikipedia.org/wiki/Firewall_(computing)).

Ubuntu servers can use the `UFW` firewall to ensure only connections to certain services are allowed. It's a simple process to set up a basic firewall and will further improve your server's security.

You can see which applications `UFW` currently allows by typing:

Command

```bash
sudo ufw app list

```

This should output the following:

Output

```bash
Available applications
  OpenSSH

```

We need to make sure the firewall allows SSH connections so that we can log back in next time. To allow these types of connections, type the following command:

Command

```bash
sudo ufw allow OpenSSH

```

And then enable the firewall:

Command

```bash
sudo ufw enable

```

Press `y` and then `ENTER` to proceed.

You can see that SSH connections are still allowed by typing:

Command

```bash
sudo ufw status

```

The `OpenSSH` directive should be listed.

That was the last step in the initial setup for our server. Our server is now configured and ready to go!

In the next section, we'll configure your domain name.

## Step 2 - Configure A Domain Name

In this section, we'll configure a domain name that you want to use for your Next.js website.

To set up a domain, you'll need to do two things:

1.  Purchase a domain name from a domain name registrar.
2.  Setup DNS (Domain Name System) records for your domain by using a DNS hosting service.

DigitalOcean is not a domain name registrar, which means you can't purchase a domain name from them. But, they do provide a DNS hosting service that makes it easy to configure a domain name with their servers.

Before proceeding, make sure you have purchased a domain name from a service like [GoDaddy](https://www.godaddy.com/), [namecheap](https://www.namecheap.com/), [HostGator](https://www.hostgator.com/), [name.com](https://www.name.com/), or another registrar.

Let's get started!

### Configure DigitalOcean DNS For Your Domain

Using DigitalOcean, let's configure DNS for your domain.

Back on the [DigitalOcean](https://m.do.co/c/ce20017d8588) website, open the **Create** drop-down menu and click the **Domains/DNS** link.

In the **Add A Domain** section, enter your domain (this is usually the base only: `example.com` and not `www.example.com`) and click the **Add Domain** button.

![DigitalOcean Add a Domain Form](https://assets.coderrocketfuel.com/coding-blog-digital-ocean-create-new-domain.png)

Once you've hit the **Add Domain** button, you'll be taken to the **Create new record** page.

You now need to add NS records for the domain on DigitalOcean's systems. You'll only be adding `A` records, which maps an IPv4 address to a domain name. This will determine where DigitalOcean should direct any requests for your domain name to.

We want your website to be accessible at two different base URLs:

1.  `example.com`
2.  `www.example.com`

Therefore, we need to create two `A` records for your domain.

For the first one, enter `@` in the `HOSTNAME` field and select the server you want to point the domain name to:

![DigitalOcean Add a Domain Form - Add a record](https://assets.coderrocketfuel.com/coding-blog-add-dns-record-example-dot-com.png)

For the second one, enter `www` in the `HOSTNAME` field and select the same server:

![DigitalOcean Add a Domain Form - Add Second A Record](https://assets.coderrocketfuel.com/coding-blog-add-dns-record-www-example-dot-com.png)

Awesome, we can move onto the next step.

### Configure Your Domain Registrar To Direct Domains To DigitalOcean

To use the DigitalOcean DNS, you'll need to update the nameservers used by your domain registrar to point to DigitalOcean's nameservers instead.

As an example, we'll walk you through the steps for doing this for [namecheap](https://www.namecheap.com/). But, these steps can be easily replicated for whatever other service you used (GoDaddy, HostGator, etc.).

First, sign in to your [namecheap](https://www.namecheap.com/) account and click **Domain List** in the left-hand column. You will be presented with a dashboard listing all of your domains.

Click the **Manage** button of the domain you'd like to update.

![Namecheap Domain List Page](https://assets.coderrocketfuel.com/namecheap-domain-list.png)

In the **Nameservers** section of the resulting screen, select Custom DNS from the dropdown menu and enter the following nameservers:

-   `ns1.digitalocean.com`
-   `ns2.digitalocean.com`
-   `ns3.digitalocean.com`

![Namecheap Domain Custom DNS Nameserver Configuration](https://assets.coderrocketfuel.com/namecheap-ns-entries.png)

Click the green checkmark to apply your changes.

It may take some time for the name server changes to propagate after you've saved them. During this time, the domain registrar communicates the changes you've made with your ISP (Internet Service Provider). In turn, your ISP caches the new nameservers to ensure quick site connections. This process usually takes about 30 minutes but could take up to a few hours depending on your registrar and your ISP's communication methods.

You should now have a domain pointing at your newly created DigitalOcean server.

Throughout the rest of this guide, your domain will be referenced as `example.com`. So, whenever you see that domain going forward, make sure you replace it with the domain you just finished configuring.

In the next section, we'll return to your DigitalOcean server and get Nginx installed and configured.

## Step 3 - Install & Configure Nginx

Now that your domain is pointing to your server, it's time to install Nginx and configure it as a reverse proxy so your Next.js application can be accessible by visitors via the browser.

We'll be using the [Nginx](https://www.nginx.com) web server software to host your applications. Nginx is one of the most popular web servers and helps host some of the largest and highest-traffic sites out there. It's more resource-friendly than Apache in most cases and can be used as either a web server or reverse proxy.

Let's get Nginx configured on your server.

### Install Nginx

First, let's get Nginx installed on your server.

Nginx is available in Ubuntu's default repositories, so installation is pretty straightforward.

On your **DigitalOcean server**, run the following commands to update your local `apt` package index so we have access to the most recent package lists and install the `nginx` software:

Command

```bash
sudo apt-get update && sudo apt-get install nginx

```

The `apt-get` command will install Nginx along with any other required dependencies.

When those commands finish, Nginx will be available for you to use on the server.

### Adjust Firewall Settings

Before we can use Nginx, we need to reconfigure our firewall software to allow access to the service. Nginx registers itself as a service with `ufw`, our firewall, upon installation. This makes it rather easy to allow Nginx access.

We can list the applications configurations that `ufw` knows how to work with by typing:

Command

```bash
sudo ufw app list

```

You should get a listing of the application profiles:

Output

```bash
Available applications:
  Nginx Full
  Nginx HTTP
  Nginx HTTPS
  OpenSSH

```

There are three profiles available for Nginx:

1.  **Nginx Full**: opens both port `80` (normal, unencrypted web traffic) and port `443` (TLS/SSL encrypted traffic)
2.  **Nginx HTTP**: opens only port `80` (normal, unencrypted web traffic)
3.  **Nginx HTTPS**: opens only port `443` (TLS/SSL encrypted traffic)

It's recommended that you enable the most restrictive profile that will still allow the traffic you've configured. Since we haven't configured SSL for our server yet, we'll only need to allow traffic on port `80`. When we set up SSL Encryption later on, we'll change these settings.

You can enable this by typing:

Command

```bash
sudo ufw allow 'Nginx HTTP'

```

You can verify the change with this command:

Command

```bash
sudo ufw status

```

You should see `Nginx HTTP` listed in the output.

### Test Your Nginx Web Server

The Nginx web server should already be up and running.

To make sure the service is running, execute this command:

Command

```bash
systemctl status nginx

```

It should output something that looks like this:

Output

```bash
● nginx.service - A high performance web server and a reverse proxy server
    Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
    Active: active (running) since Mon 2016-04-18 16:14:00 EDT; 4min 2s ago
  Main PID: 12857 (nginx)
    CGroup: /system.slice/nginx.service
      ├─12857 nginx: master process /usr/sbin/nginx -g daemon on; master_process on
      └─12858 nginx: worker process

```

You can access the default Nginx landing page to confirm that the software is running properly. This can be accessed through your server's domain name or IP address in the browser.

When you have your server's IP address or domain, enter it into your browser's address bar: `http://server_domain_or_ip_address`.

You should see the default Nginx landing HTML page, which should look something like this:

![Nginx Default HTML Page](https://assets.coderrocketfuel.com/default_nginx.png)

You now have a Nginx web server running on your server that is accessible to the world!

We'll come back to Nginx later on this section when we create a reverse-proxy for your application.

## Step 4 - Install Node.js

To run your Next.js application, you'll need to have Node.js installed on your DigitalOcean server. Furthermore, you'll need to have Node.js version `14.6.0` or newer for Next.js to work properly.

To make things easy, we'll use the [NodeSource](https://github.com/nodesource/distributions) package archives to install Node.js.

First, you need to install the NodeSource PPA (Personal Package Archive) to get access to its contents. Make sure you're in the home directory on your **DigitalOcean server**:

Command

```bash
cd ~

```

And use `curl` to retrieve the installation script for the Node.js `18.x` archives (replace `18.x` with the version of Node.js you want to use):

Command

```bash
curl -sL https://deb.nodesource.com/setup_18.x -o nodesource_setup.sh

```

And then run the `nodesource_setup.sh` script using `sudo`:

Command

```bash
sudo bash nodesource_setup.sh

```

The PPA has now been added to your configuration and your local package cache should have automatically updated.

The `nodesource_setup.sh` script won't be needed again, so let's delete it:

Command

```bash
sudo rm nodesource_setup.sh

```

You can now install the Node.js package with the following command:

Command

```bash
sudo apt-get install nodejs

```

The `nodejs` package contains the `nodejs` binary as well as NPM, so you won't need to install NPM separately. But for some NPM packages to work, you'll need to install the `build-essential` package:

Command

```bash
sudo apt-get install build-essential

```

When that is done installing, Node.js should be fully installed and ready to use on your server.

You can test this by verifying the version of Node.js on your machine:

Command

```bash
node --version

```

And the version of NPM:

Command

```bash
npm --version

```

In the next sections, we'll use Node.js and NPM to configure and run your Next.js application.

## Step 5 - Deploy The Next.js Application

Now it's time to deploy your Next.js application!

If you don't have one created already, check out our [article](https://coderrocketfuel.com/article/how-to-create-a-website-with-next-js-and-react) on creating a Next.js website before proceeding.

To deploy your Next.js website, we'll need to push the code for your project to [GitHub](https://github.com/) (or [Gitlab](https://gitlab.com/)) and pull it into our DigitalOcean server. Then we'll run the application in production mode and keep it running forever using the [PM2](https://www.npmjs.com/package/pm2) NPM package.

Let's get to it!

### Push Application Code To GitHub/Gitlab

Let's push the project code for your Next.js application to GitHub.

Gitlab is another option for source code management and the proceeding steps will also work for it.

We need to first create a repository on GitHub that you can upload the code to.

First, go to [https://github.com](https://github.com/).

![GitHub Homepage Screenshot](https://assets.coderrocketfuel.com/github-homepage.png)

If you don't have a GitHub account, you'll need to create one.

Once you're logged in or signed up, click the **Start a Project** button or navigate to the **Create a New Repository** page.

![GitHub Welcome Page](https://assets.coderrocketfuel.com/github-start-a-project.png)

After you press the **Start a Project** button, you'll be redirected to another page where you can name your repository, give it a description and more.

![GitHub Create a New Repository Page](https://assets.coderrocketfuel.com/github-create-new-repo-form.png)

When you're done filling out the form, press the **Create Repository** button.

You should now have an empty repository created on GitHub!

If you don't plan on sharing your code with others, we suggest making your repository private. You can do so in the **Settings** section of your GitHub repository.

Now we can add some code to our repository.

On your **local machine**, navigate to the root directory of your project.

Command

```bash
cd your-project

```

And run the `git init` command to initiate the project folder as a Git repository:

Command

```bash
git init

```

Next, we need to set the remote origin for our repository. This gives Git a URL for where our code is stored remotely for this repository. When you push changes later on, this is the URL that code will be uploaded to.

Command

```bash
git remote add origin https://github.com/your_github_username/your_repository_name.git

```

Before we push the code to GitHub (or Gitlab), we need to add a `.gitignore` file to our project. This will instruct Git to not commit certain files or directories of a given name or type.

In the root of your project directory, create a `.gitignore` file:

Command

```bash
touch .gitignore

```

Then, add this code to the file via your editor:

/your-project/.gitignore

```text
# Logs
logs
*.log
npm-debug.log*

# Runtime data
pids
*.pid
*.seed

# Directory for instrumented libs generated by jscoverage/JSCover
lib-cov

# Coverage directory used by tools like istanbul
coverage

# nyc test coverage
.nyc_output

# Grunt intermediate storage (http://gruntjs.com/creating-plugins#storing-task-files)
.grunt

# node-waf configuration
.lock-wscript

# Compiled binary addons (http://nodejs.org/api/addons.html)
build/Release

# Dependency directories
node_modules
jspm_packages

# Optional npm cache directory
.npm

# Optional REPL history
.node_repl_history
.next

```

Now, we can commit and push our project code without causing problems like committing our entire `node_modules` file.

Tell Git to stage all the files in the project folder with this command:

Command

```bash
git add .

```

And create a new commit:

Command

```bash
git commit -m "Initial commit."

```

To push the commit you just created to your GitHub remote repository, use the `push` command as follows:

Command

```bash
git push origin master

```

Once you've successfully entered your GitHub account credentials and the push process is finished, head over to GitHub and view your repository. You should see the code has been pushed and is now visible.

Your application is now ready to pulled onto the server.

### Configure & Run Your Application On The Server

We're now ready to get your Next.js application up and running on your DigitalOcean server.

First, we'll pull in the code we just pushed to GitHub and then we'll get the application running.

On your **DigitalOcean server**, make sure you are located in the home directory:

Command

```bash
cd ~

```

Next, pull in the code from GitHub by cloning the repository we just created (make sure you use your specific GitHub link):

Command

```bash
git clone https://github.com/your_github_username/your_repository_name.git website

```

If you made your repository private, it'll ask for your GitHub username and password credentials before it initiates the cloning process.

The `website` portion of the command will ensure the name of the project folder created is `/website`.

When the clone is complete, `cd` into the new `/website` directory:

Command

```bash
cd website

```

And use the `npm install` command to install all the NPM dependencies for your project:

Command

```bash
npm install

```

When that's done, test the application by running the `npm run dev` command:

Command

```bash
npm run dev

```

You shouldn't encounter any errors and the terminal should output the same message as it did when we were in development mode on your local machine:

Output

```bash
ready - started server on http://localhost:3000

```

Stop the application with the `CTRL`+`C` keyboard shortcut.

Now that we know everything is working, we need to use the `npm run build` function from the `package.json` file to create a production version of the Next.js application.

Still in the root of the `/website` project directory, execute this command:

Command

```bash
npm run build

```

This command may take a minute to complete.

When it's complete, a new `/.next` folder will be created in your project directory. The production version of your website will be ran using the code in that folder.

When we run the application in production mode, we'll use the `npm start` command (instead of the `npm run dev`). So, let's start the application using that command:

Command

```bash
npm start

```

If all went well, you should get this output in your terminal:

Output

```bash
Ready on http://localhost:3000

```

Awesome! Your frontend website Next.js application is up and running on the server!

But how do we keep the application running forever, even if errors occur or in the rare scenario the entire machine needs to be restarted? In the next section, we'll introduce the PM2 NPM package to help us with that.

### Install & Use PM2 To Run The Application

You're now ready to install and configure [PM2](https://www.npmjs.com/package/pm2), which is a process manager for Node.js applications. It will allow you to keep your Node.js application alive forever, reload it without any downtime and help facilitate common system administrative tasks.

Install the PM2 NPM package globally on your **DigitalOcean machine** with this command:

Command

```bash
sudo npm install -g pm2

```

The `-g` option tells NPM to install the package globally. It'll now be available across your server's entire system.

Now we want to run the Next.js website application using PM2.

First, make sure you've navigated into the `/website` project directory:

Command

```bash
cd website

```

Then, use this command to run the application with PM2:

Command

```bash
pm2 start --name=website npm -- start

```

This will start your Next.js application and then display a table with its name, process ID, status, CPU usage, etc.

PM2 automatically adds an `App Name` and PM2 `id` to your application, along with other information such as the `PID` of the process, its current state, and memory usage.

The specific command we used gives the application the name of `website` within PM2. This will help us easily identify all the applications running within PM2 as we add more.

PM2 applications will be restarted automatically if the application crashes or is killed. But additional steps need to be taken for applications to start on system startup (reboot or boot). PM2 provides an easy way to do this with its `startup` command.

Run it with the following command:

Command

```bash
pm2 startup systemd

```

On the last line of the resulting output, there will be a command that you must run with superuser privileges:

Output

```bash
[PM2] Init System found: systemd
[PM2] You have to run this command as root. Execute the following command:
sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u bob --hp /home/bob

```

Copy and paste the command that was generated (same as above but with your username instead of bob) to have PM2 always start when your server is booted.

Your command will look similar to this:

Command

```bash
sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u bob --hp /home/bob

```

This will create a `systemd` unit that will run PM2 for your user on boot. This PM2 instance, in turn, will run all the applications configured to run using PM2. To check the status of the new `systemd` unit, use the following command:

Command

```bash
systemctl status pm2-bob

```

Your Next.js website application is now up and running on port `3000` and will run forever, or until you tell PM2 to stop.

In the future, you'll most likely need to push new code updates. To do so, you can follow these steps:

1.  Push code changes to GitHub/Gitlab.
2.  SSH into your DigitalOcean server (i.e. `ssh bob@example.com` command via a terminal window).
3.  Remove the current `/website` directory with `sudo rm -r website` (PM2 will continue to run a cached version of the application, so website downtime won't be an issue).
4.  Clone your GitHub repository with `git clone https://github.com/your_github_username/your_repository_name.git website`.
5.  Install the project's NPM dependencies with `npm install`.
6.  Run the Next.js build process with `npm run build`.
7.  Restart the PM2 process with `pm2 restart website`.

In the next section, we'll setup a Nginx reverse-proxy in front of your Next.js application to make it accessible to the world.

## Step 6 - Configure Nginx As A Reverse Proxy

Now that your application is running and listening on `localhost:3000`, we need to make it so people from the outside world can access it. To achieve this, we will use Nginx as a [reverse-proxy](https://en.wikipedia.org/wiki/Reverse_proxy).

On Debian systems (i.e. Ubuntu), Nginx server configuration files are stored in `/etc/nginx/sites-available` directory, which are enabled through symbolic links to the `/etc/nginx/sites-enabled` directory.

Therefore, we need to create a configuration file in the `/etc/nginx/sites-available` directory that will instruct Nginx how to serve our application and at what URLs to serve it to.

First, navigate to the `/etc/nginx/sites-available` directory:

Command

```bash
cd /etc/nginx/sites-available

```

Then, create a new configuration file for your website (you'll need to use `sudo` for any changes you make in these configuration files):

Command

```bash
sudo touch example.com

```

To keep things organized, we suggest naming each Nginx configuration file after the URL they're created for.

Open that file for editing using `nano`:

Command

```bash
sudo nano example.com

```

And add this code to the file:

/etc/nginx/sites-available/example.com

```nginx
server {
        listen 80;
        listen [::]:80;

        root /var/www/html;
        index index.html index.htm index.nginx-debian.html;

        server_name _;

        location / {
                try_files $uri $uri/ =404;
        }
}

```

These are the contents of the [default Nginx configuration file](https://coderrocketfuel.com/article/default-nginx-configuration-file-inside-sites-available-default), but with all the commented out lines removed for brevity. Also, the `default_server` designation for the listen directives (`listed 80;` and `listen [::]:80;`) are removed. Only one server block can have the `default_server` option enabled, which is already occupied by the `/sites-available/default` configuration file.

The first thing we need to modify in this file is the domain URL and any other aliases to match requests for. To do this, replace the `server_name _;` line with the following:

/etc/nginx/sites-available/example.com

```nginx
server_name example.com www.example.com;

```

Make sure you replace `example.com` and `www.example.com` with your URLs.

This configuration file is also telling Nginx to serve the content of the `/var/www/html` directory, which is a default file generated by Nginx. To change that to instead serve our Next.js application running on `localhost:3000`, update the `location` section to this:

/etc/nginx/sites-available/example.com

```nginx
location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
}

```

With those settings, Nginx will serve the application running at `http://localhost:3000`, which we just finished configuring in the last section.

When you're done, the entire file should look similar to this:

/etc/nginx/sites-available/example.com

```nginx
server {
        listen 80;
        listen [::]:80;

        root /var/www/html;
        index index.html index.htm index.nginx-debian.html;

        server_name example.com www.example.com;

        location / {
                proxy_pass http://localhost:3000;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection 'upgrade';
                proxy_set_header Host $host;
                proxy_cache_bypass $http_upgrade;
        }
}

```

Save and exit the file when you're finished making changes.

Now that we have the server block file created, we need to enable it. We can do this by creating symbolic links from these files to the `/sites-enabled` directory, which Nginx reads from during startup.

We can create the symbolic link with this command:

Command

```bash
sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/

```

Make sure you replace the `example.com` portion of the command with the name of your configuration file.

Your configuration file is now symbolically linked to the `/sites-enabled` directory.

To avoid a hash bucket memory problem in the future when we add additional applications, we need to adjust a single line in the `/etc/nginx/nginx.conf` file.

Open the file using `nano`:

Command

```bash
sudo nano /etc/nginx/nginx.conf

```

In that file, find a commented out line that reads `# server_names_hash_bucket_size 64;` and remove the comment `#` symbol to uncomment the line:

/etc/nginx/nginx.conf

```nginx
http {
    . . .

    server_names_hash_bucket_size 64;

    . . .
}

```

Save and close the file when you're done.

Next, use the `nginx -t` command to test that there are no syntax errors any of the Nginx files you just edited or created:

Command

```bash
sudo nginx -t

```

If no problems were found, restart Nginx to enable the changes you made:

Command

```bash
sudo systemctl restart nginx

```

Nginx should now be serving the Next.js application at your domain (`www.example.com`).

Open a browser and navigate to your domain. You should see your website!

In the next section, we'll set up SSL encryption so your website can be served over `https://`.

## Step 7 - Configure HTTPS/SSL Encryption

Next, we'll use both the [Let's Encrypt](https://letsencrypt.org/) service and the [Certbot](https://certbot.eff.org/) software to obtain and install free SSL certificates for the `example.com` and `www.example.com` URLs.

Let's Encrypt is a Certificate Authority (CA) that provides an easy way to obtain and install free SSL certificates, thereby enabling encrypted HTTPS on web servers. It simplifies the process by providing a software client, Certbot, that attempts to automate most (if not all) of the required steps.

Let's get it done!

### Install Certbot

First, we need to install the Certbot software on your server.

You can add the repository to your machine with this command:

Command

```bash
sudo add-apt-repository ppa:certbot/certbot

```

You'll need to press `ENTER` to accept.

Then, update the package list on your machine to pick up the Certbot repository's package information:

Command

```bash
sudo apt-get update

```

And then install the Certbot package:

Command

```bash
sudo apt install python-certbot-nginx

```

The Certbot software is now ready to use on your server.

### Update Firewall To Allow HTTPS

Previously, we configured the `ufw` firewall on your server to allow HTTP traffic. To additionally let in HTTPS traffic, we need to allow the `Nginx Full` profile and then delete the redundant Nginx HTTP allowance.

Here is the command to allow `Nginx Full`:

Command

```bash
sudo ufw allow 'Nginx Full'

```

And here is the command to delete the redundant `Nginx HTTP` profile:

Command

```bash
sudo ufw delete allow 'Nginx HTTP'

```

We're now ready to run Certbot and configure SSL certificates for your website.

### Obtain SSL Certificates

Certbot provides a variety of ways to obtain SSL certificates, through various plugins. The Nginx plugin will take care of reconfiguring Nginx and reloading the Nginx configuration files whenever necessary.

To get SSL certificates for your `example.com` and `www.example.com` URLs, use this command (make sure to use your URLs):

Command

```bash
sudo certbot --nginx -d example.com -d www.example.com

```

This command runs `certbot` with the `--nginx` plugin and also uses the `-d` directive to specify the names we want certificates to be valid for.

Since this is the first time you've run Certbot on the machine, you'll be prompted to enter an email address and asked to agree to the terms of service.

After doing so, Certbot will communicate with the Let's Encrypt server, then run a challenge to verify that you control the domain you’re requesting a certificate for.

If that’s successful, Certbot will ask how you’d like to configure your HTTPS settings.

Output

```bash
Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
-------------------------------------------------------------------------------
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
-------------------------------------------------------------------------------
Select the appropriate number [1-2] then [enter] (press 'c' to cancel):

```

The redirect option (Option 2) is the suggested choice. Select your choice then hit `ENTER`. The configuration will be updated, and Nginx will reload to pick up the new settings.

Your certificates are now downloaded, installed, and loaded. Go to your browser window and try reloading the website using `https://` instead. The site should now be served using `https://`.

The last thing we need to do is test the certificate renewal process.

Let's Encrypt’s certificates are only valid for ninety days. This is to encourage users to automate their certificate renewal process.

The `certbot` package we installed takes care of this for us by running `certbot renew` twice a day via a `systemd` timer. On non-systemd distributions, this functionality is provided by a script placed in `/etc/cron.d`. This task runs twice a day and will renew any certificate that’s within thirty days of expiration.

To test the renewal process, you can do a dry run with `certbot`:

Command

```bash
sudo certbot renew --dry-run

```

If you don't see any errors, you're good to go!

When needed, Certbot will renew your certificates and reload Nginx to pick up the changes. If the automated renewal process fails, Let's Encrypt will send a message to the email you specified, warning you when your certificate is about to expire.

Your Next.js website is now fully deployed!

Thanks for reading and happy coding!
