# Deploy A Node.js Application To DigitalOcean With HTTPS

In this guide, we'll deploy a [Node.js](https://nodejs.org/en/) application on a [DigitalOcean](https://m.do.co/c/ce20017d8588) server in the cloud with SSL/HTTPS encryption and a custom domain.

Furthermore, [Nginx](https://www.nginx.com/) will be used as a [reverse proxy](https://en.wikipedia.org/wiki/Reverse_proxy) and [Let's Encrypt](https://letsencrypt.org/) and [Certbot](https://certbot.eff.org/) will be used to configure SSL/HTTPS encryption for your application.

When the Node.js application is running on the server, we'll use [PM2](https://github.com/Unitech/pm2) to keep the application running forever without any downtime.

Let's get started!

### Table of Contents

1.  [Create & Configure A New DigitalOcean Server](https://coderrocketfuel.com/article/deploy-a-nodejs-application-to-digital-ocean-with-https#configure-a-new-digitalocean-server)
2.  [Configure Your Domain Name](https://coderrocketfuel.com/article/deploy-a-nodejs-application-to-digital-ocean-with-https#configure-your-domain-name)
3.  [Install & Configure Nginx](https://coderrocketfuel.com/article/deploy-a-nodejs-application-to-digital-ocean-with-https#install-and-configure-nginx)
4.  [Configure SSL/HTTPS Using Let's Encrypt & Certbot](https://coderrocketfuel.com/article/deploy-a-nodejs-application-to-digital-ocean-with-https#configure-ssl-https)
5.  [Configure The Node.js Application](https://coderrocketfuel.com/article/deploy-a-nodejs-application-to-digital-ocean-with-https#configure-the-node-js-application)
6.  [Setup Nginx As A Reverse Proxy](https://coderrocketfuel.com/article/deploy-a-nodejs-application-to-digital-ocean-with-https#setup-nginx-as-a-reverse-proxy)

## Step 1 - Create & Configure A New DigitalOcean Server

Before we can do anything, we need to create and configure a VPS ([Virtual Private Server](https://en.wikipedia.org/wiki/Virtual_private_server)) in the cloud to host your Node.js application. There are a lot of companies that provide this service, but we'll use [DigitalOcean](https://m.do.co/c/ce20017d8588).

You can use any other VPS service provider you wish, but some of the steps in this tutorial will be slightly different for you.

To start, you need to create an account on DigitalOcean or log in to your existing account.

For a **FREE $200 CREDIT FOR 60 DAYS**, use this link to sign up: [https://m.do.co/c/ce20017d8588](https://m.do.co/c/ce20017d8588).

They will ask you for a credit card, but you can cancel anytime before the 60 days end and not be charged.

### Create A New Droplet On DigitalOcean

After logging in or successfully signing up for a new account, open the **Create** drop-down menu and click the **Droplets** link.

This will take you to the **Create Droplets** page where you'll be given some configuration options before creating a new server.

In the first section, select the Ubuntu operating system for your server.

Ubuntu 18.04 is the operating system used in this guide. If you use a different version of Ubuntu, some of the commands may differ from what's laid out in this guide.

Then, choose the $5 per month standard plan, which will give your application plenty of computing power to start with. You can easily upgrade in the future if needed.

![DigitalOcean Create New Droplet Page - Select Plan](https://assets.coderrocketfuel.com/coding-blog-create-digitalocean-droplet-choose-plan.png)

Next, they let you choose the datacenter region for your server. This is the physical location for your server and, therefore, you should choose the one closest to the people visiting your website.

![DigitalOcean Create New Droplet Page - Choose Datacenter Region](https://assets.coderrocketfuel.com/coding-blog-create-digitalocean-droplet-datacenter-region.png)

In the **Authentication** section, make sure the **Password** option is selected and enter a strong root password for your server. This is the password we'll use to initially SSH into your server.

![Create DigitalOcean Droplet - Root Password](https://assets.coderrocketfuel.com/coding-blog-create-digitalocean-droplet-root-password.png)

Also, you can choose a hostname for your server. This will give your server a name to remember it by.

![Create DigitalOcean Droplet - Hostname](https://assets.coderrocketfuel.com/coding-blog-create-digitalocean-droplet-hostname.png)

When you're done selecting options, hit the **Create Droplet** button to kick off the creation of your new server.

![Create DigitalOcean Droplet - Submit Button](https://assets.coderrocketfuel.com/coding-blog-create-digitalocean-droplet-submit-button.png)

When the Droplet is fully up and running, the control panel will display the server's IP address.

![Create DigitalOcean Droplet - Create Droplet Success](https://assets.coderrocketfuel.com/coding-blog-create-digitalocean-droplet-success.png)

Your server is now up and running!

In the next step, we'll complete the initial configuration process for the server. This will include logging into the server, setting up SSH access to the server, and creating a basic firewall.

### Access The Server Using Root

The first step is to gain access to the server using your root login.

To do this, you'll need both the IP address of the server and the private key (password) for the `root` user's account you created in the last step.

To log into your server, open a terminal (`Ctrl`+`Alt`+`T` for Linux) on your local machine.

Once you have a terminal open, use the following command to SSH in as the root user (replace `server_ip_address` with your server's public IP address):

Command

```bash
ssh root@server_ip_address

```

Accept the warning about host authenticity, if it appears, and provide the root password you created.

The `root` user in a Linux environment has very broad privileges and, for that reason, you are discouraged from using it on a regular basis. This is because very destructive changes (even by accident) can be made while using it.

Therefore, in the next step, we are going to create an alternative account with limited scope that will be used for daily work.

### Create A New User

Logged in as `root`, we can create a new user account that will be used to log in from this point forward. You can create a new user with the following command (substitute `bob` with your username):

Command

```bash
adduser bob

```

You'll be asked some questions starting with the password. Choose a strong password and fill in any of the optional information after that. You can just hit `ENTER` repeatedly to skip the rest of the questions.

### Give Your New User Root Privileges

You now have a new user account with regular account privileges. But you might occasionally need to do administrative tasks that require root privileges. So, instead of logging out of your normal user and logging back in as the `root` account, we can give the normal account the ability to run root privileged commands when you need to by adding `sudo` before each command.

To do this, we need to add your new user to the `sudo` group on the machine.

As `root`, run the following command to add your user to the `sudo` group (substitute `bob` with your username):

Command

```bash
usermod -aG sudo bob

```

Now your user can run commands with `root` privileges!

The next server set up steps will help increase the security of your server. They are optional but **highly recommended**.

### Add Public Key Authentication

By setting up public-key authentication for the new user, it will increase your server's security by requiring a private SSH key to login instead of anyone being able to access the server via password.

Let's get pubic key authentication configured.

#### Generate A Key Pair

If you don't already have an SSH key pair, which consists of a public and private key, you need to generate one. If you already have a key that you want to use, skip to the [Copy the Public Key](https://coderrocketfuel.com/article/deploy-a-nodejs-application-to-digital-ocean-with-https#copy-public-key) step.

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

Next, you'll be prompted to enter a password to secure the newly created key. You can either create a password or leave it blank. This generates a private key, `id_rsa`, and a public key, `id_rsa.pub`, in the `.ssh` directory of your home directory.

#### Copy The Public Key

Now that you have the SSH key pair on your local machine, you need to copy our public key to the server.

Use one of the two options below to do this.

**Option 1: SSH-Copy-Id**

If your local machine has the `ssh-copy-id` script installed, you can use it to install your public key to any user that you have login credentials for. If not, use [Option 2](https://coderrocketfuel.com/article/deploy-a-nodejs-application-to-digital-ocean-with-https#option-2-install-the-key-manually) to install the key manually.

Still on your **local machine**, type the following command (replace `bob` and `server_ip_address` with your username and server public IP address:

Command

```bash
ssh-copy-id bob@server_ip_address

```

You'll be asked for the user's password.

This is the password for the user you created on your server. Not the root password.

After successfully authenticating, your public key will be added to the server user's `.ssh/authorized_keys` file. The corresponding private key can now be used to log into the server.

You can skip the next section on installing the key manually and jump to the [Disable Password Authentication](https://coderrocketfuel.com/article/deploy-a-nodejs-application-to-digital-ocean-with-https#disable-password-authentication) section.

**Option 2: Install The Key Manually**

Assuming you generated an SSH key pair using the previous step, use the following command at the terminal of your **local machine** to print your public key (`id_rsa.pub`):

Command

```bash
cat ~/.ssh/id_rsa.pub

```

This should print your public SSH key, which should look something like the following:

Output

```bash
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDBGTO0tsVejssuaYR5R3Y/i73SppJAhme1dH7W2c47d4gOqB4izP0+fRLfvbz/tnXFz4iOP/H6eCV05hqUhF+KYRxt9Y8tVMrpDZR2l75o6+xSbUOMu6xN+uVF0T9XzKcxmzTmnV7Na5up3QM3DoSRYX/EP3utr2+zAqpJIfKPLdA74w7g56oYWI9blpnpzxkEd3edVJOivUkpZ4JoenWManvIaSdMTJXMy3MtlQhva+j9CgguyVbUkdzK9KKEuah+pFZvaugtebsU+bllPTB0nlXGIJk98Ie9ZtxuY3nCKneB+KjKiXrAvXUPCI9mWkYS/1rggpFmu3HbXBnWSUdf localuser@machine.local

```

Select the public key, and copy it to your clipboard.

To enable the use of the SSH key to authenticate as the new remote user, you must add the public key to a special file in the user's home directory.

**On your DigitalOcean server** and as the `root` user, enter the following command to temporarily switch to the new user (substitute `bob` with your username):

Command

```bash
su - bob

```

Now you will be in your new user's home directory.

Create a new directory called `.ssh` and restrict its permissions with the following commands:

Command

```bash
mkdir ~/.ssh && chmod 700 ~/.ssh

```

Now open a file in `.ssh` called `authorized_keys` with a text editor. We'll use `nano` to edit the file:

Command

```bash
nano ~/.ssh/authorized_keys

```

Now insert your public key (which should be in your clipboard) by pasting it into the editor.

Hit `CTRL-X` to exit the file, then `Y` to save the changes that you made, then `ENTER` to confirm the file name.

Now restrict the permissions of the `authorized_keys` file with this command:

Command

```bash
chmod 600 ~/.ssh/authorized_keys

```

Type this command once to return to the `root` user:

Command

```bash
exit

```

Now your public key is installed, and you can use SSH keys to log in as your user.

### Disable Password Authentication

This step will configure your server to only allow SSH access to users that have the SSH key you just created on your local machine.

Only people who possess the private key that pairs with the public key that was installed will get into the server. This increases your server's security by disabling password-only authentication.

Only follow this step if you installed a public key in the last step. Otherwise, you'll lock yourself out of the server.

To disable password authentication, follow the proceeding steps.

As the `root` user or new `sudo` user on your **DigitalOcean server**, open the SSH daemon configuration file using the following command:

Command

```bash
sudo nano /etc/ssh/sshd_config

```

Find the line that says `PasswordAuthentication` and change its value to `no`. It should look like this after the change was made:

/etc/ssh/sshd_config

```txt
PasswordAuthentication no

```

Save and close the file using the method: `CTRL-X`, then `Y`, then `ENTER`.

To reload the SSH daemon and put the changes live, execute the following command:

Command

```bash
sudo systemctl reload sshd

```

Password authentication is now disabled. Now your server can only be accessed with SSH key authentication.

### Test Log In Using SSH Key

Let's test logging in using the SSH key.

On your **local machine**, SSH into your server using the new account that we created. Use the following command to do so (substitute your username and IP address):

Command

```bash
ssh bob@server_ip_address

```

Once authentication is provided to the server, you will be logged in as your new user.

Now your server is only accessible by someone who has the SSH keys you generated. Which, in this case, are only located on your local machine. This greatly improves the security of your server.

### Basic Firewall Set Up

Another security improvement we can make to the server is to add a basic [firewall](https://en.wikipedia.org/wiki/Firewall_(computing)).

Ubuntu servers can use the `UFW` firewall to ensure only connections to certain services are allowed. It's a simple process to set up a basic firewall and will improve your server's security.

On your **DigitalOcean server**, you can see which applications `UFW` currently allows by typing:

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

Press `y` and then `ENTER` to proceed. You can see that SSH connections are still allowed by typing:

Command

```bash
sudo ufw status

```

That was the last step in the initial setup for the server.

## Step 2 - Configure Your Domain Name

In this section, we'll configure a domain name that you want to use for your Node.js application.

To set up a domain, you'll need to do two things:

1.  Purchase a domain name from a domain name registrar.
2.  Setup DNS ([Domain Name System](https://en.wikipedia.org/wiki/Domain_Name_System)) records for your domain by using a DNS hosting service.

DigitalOcean is not a domain name registrar, which means you can't purchase a domain name from them. But, they do provide a DNS hosting service that makes it easy to configure a domain name with their servers.

Before proceeding to the next step, make sure you have purchased a domain name from a service like [GoDaddy](https://www.godaddy.com), [namecheap](https://www.namecheap.com) (my personal favorite), [HostGator](https://www.hostgator.com), [name.com](https://www.name.com), or another registrar.

### Configure DNS

Using DigitalOcean, let's configure DNS for your domain.

Back on the DigitalOcean website, open the **Create** drop-down menu and click the **Domains/DNS** link.

In the **Add a Domain** section, enter your domain (this is usually the base only: `example.com` and not `www.example.com`) and click the **Add Domain** button.

![DigitalOcean Create New Domain](https://assets.coderrocketfuel.com/coding-blog-digital-ocean-create-new-domain.png)

After you click the **Add Domain** button, you will be taken to the **Create new record** page.

You now need to add NS records for the domain on DigitalOcean servers. You'll only be adding `A` records, which maps an `IPv4` address to a domain name. This will determine where to direct any requests for your domain name.

Therefore, we need to create two `A` records for your website.

For the first one, enter `@` in the `HOSTNAME` field and select the server you want to point the domain name to:

![DigitalOcean Add DNS Record Example Dot Com](https://assets.coderrocketfuel.com/coding-blog-add-dns-record-example-dot-com.png)

For the second one, enter `www` in the `HOSTNAME` field and select the same server:

![DigitalOcean Add DNS Record WWW Dot Example Dot Com](https://assets.coderrocketfuel.com/coding-blog-add-dns-record-www-example-dot-com.png)

Make sure the `A` records are pointed to the correct server droplet.

Awesome, we can move on to the next step.

### Configure Your Domain Registrar To Direct The Domain To DigitalOcean

To use the DigitalOcean DNS, you'll need to update the nameservers used by your domain registrar to point at DigitalOcean's nameservers instead.

As an example, we'll walk you through the steps for doing this for [namecheap](https://www.namecheap.com). But, these steps can be easily replicated for whatever other service you used (GoDaddy, HostGator, etc.).

First, sign in to your [namecheap](https://www.namecheap.com) account and click **Domain List** in the left-hand column. You will be presented with a dashboard listing all of your domains.

Click the **Manage** button of the domain you'd like to update.

![Namecheap Domain List](https://assets.coderrocketfuel.com/namecheap-domain-list.png)

In the **Nameservers** section of the resulting screen, select **Custom DNS** from the dropdown menu and enter the following nameservers:

-   `ns1.digitalocean.com`
-   `ns2.digitalocean.com`
-   `ns3.digitalocean.com`

![Namecheap NS Entries](https://assets.coderrocketfuel.com/namecheap-ns-entries.png)

Click the green checkmark to apply your changes.

It may take some time for the name server changes to propagate after you've saved them.

During this time, the domain registrar communicates the changes you've made with your ISP (Internet Service Provider). In turn, your ISP caches the new nameservers to ensure quick site connections. This process usually takes about 30 minutes but could take up to a few hours depending on your registrar and your ISP's communication methods.

To set up your domain on other registrars, there are more guides [here](https://www.digitalocean.com/community/tutorials/how-to-point-to-digitalocean-nameservers-from-common-domain-registrars).

You should now have a domain pointing at your newly created DigitalOcean server.

## Step 3 - Install & Configure Nginx

Now that your domain is pointing to your server, it's time to install [Nginx](https://www.nginx.com) and configure your server to host web content.

Nginx is one of the most popular web servers and helps host some of the largest and highest-traffic sites out there. It is more resource-friendly than Apache in most cases and can be used as a web server or a reverse proxy.

Let's get Nginx configured on your server.

### Install Nginx

Nginx is available in Ubuntu's default repositories, so installation is pretty straightforward.

On your **DigitalOcean server**, run the following commands to update your local `apt package` index so we have access to the most recent package lists:

Command

```bash
sudo apt-get update && sudo apt-get install nginx

```

The `apt-get` command will install Nginx along with any other required dependencies.

When those commands finish, Nginx will be available for you to use on the server.

### Adjust The Firewall

Before we can test Nginx, we need to reconfigure our firewall software to allow access to the service. Nginx registers itself as a service with `ufw`, our firewall, upon installation. This makes it rather easy to allow Nginx access.

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

-   **Nginx Full**: Opens both port `80` (normal, unencrypted web traffic) and port `443` (`TLS`/`SSL` encrypted traffic)
-   **Nginx Http**: Opens only port `80` (normal, unencrypted web traffic)
-   **Nginx Https**: Opens only port `443` (`TLS`/`SSL` encrypted traffic)

It is recommended that you enable the most restrictive profile that will still allow the traffic you've configured. Since we haven't configured SSL for our server yet, in this guide, we will only need to allow traffic on port `80`.

When we configure SSL/HTTPS encryption later on, we'll change these settings.

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

### Test Your Web Server

The Nginx web server should already be up and running.

You can check with the systemd init system to make sure the service is running by typing:

Command

```bash
systemctl status nginx

```

This should output the following:

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

You can access the default Nginx landing page to confirm that the software is running properly. You can access this through your server's domain name or IP address.

When you have your server's IP address or domain, enter it into your browser's address bar:

Server URL

```bash
http://server_domain_or_IP

```

You should see the default Nginx landing page, which should look something like this:

![Default NGINX Page HTML](https://assets.coderrocketfuel.com/default_nginx.png)

You now have a web server running!

In the next step, we will configure `SSL` certificates for your domain. This will allow us to run the application over `https://`.

## Step 4 - Configure SSL/HTTPS Using Let's Encrypt & Certbot

[Let's Encrypt](https://letsencrypt.org) is a Certificate Authority (CA) that provides an easy way to obtain and install free `SSL` certificates, thereby enabling encrypted HTTPS on web servers. It simplifies the process by providing a software client, [Certbot](https://certbot.eff.org), that attempts to automate most (if not all) of the required steps.

Currently, the entire process of obtaining and installing a certificate is fully automated on both Apache and Nginx.

We'll use Certbot to obtain a free `SSL` certificate for Nginx and set up your certificate to renew automatically.

### Install Certbot

The first step is to install the Certbot software on your server.

First, add the repository to your server:

Command

```bash
sudo add-apt-repository ppa:certbot/certbot

```

Press `ENTER` to accept.

Then, update the package list to pick up the new Certbot repository information:

Command

```bash
sudo apt-get update

```

When that finishes, install Certbot's Nginx package using the `apt` package manager:

Command

```bash
sudo apt install python-certbot-nginx

```

Certbot is now ready to use!

### Update Nginx Configuration

Certbot can automatically configure `SSL` for Nginx, but it needs to be able to find the correct server block in your server's Nginx configuration settings. It does this by looking for a `server_name` directive that matches the domain you're requesting a certificate for.

Open the default configuration file with `nano` or your favorite text editor:

Command

```bash
sudo nano /etc/nginx/sites-available/default

```

Find the existing `server_name` line and replace the underscore with your domain name:

/etc/nginx/sites-available/default

```txt
. . .

server_name example.com www.example.com;

. . .

```

Save the file and exit the editor.

Then, verify the syntax of your configuration edits with:

Command

```bash
sudo nginx -t

```

If you get any errors, reopen the file and check for typos, then test it again.

Once your configuration's syntax is correct, reload Nginx to load the new configuration:

Command

```bash
sudo systemctl reload nginx

```

Certbot will now be able to find the correct server block and update it.

Next, we will update your firewall to allow HTTPS traffic.

### Allow HTTPS Access In Your Firewall

Previously, we configured the `ufw` firewall on your server to allow HTTP traffic. To additionally let in HTTPS traffic, we need to allow the `Nginx Full` profile and then delete the redundant `Nginx HTTP` allowance.

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

We're now ready to run Certbot and fetch the `SSL` certificates.

### Get The SSL Certificate From Certbot

Certbot provides a variety of ways to obtain `SSL` certificates, through various plugins. The Nginx plugin will take care of reconfiguring Nginx and reloading the configuration settings whenever it's necessary.

To get SSL certificates for your `example.com` and `www.example.com` URLs, use this command (make sure to use your URLs):

Command

```bash
sudo certbot --nginx -d example.com -d www.example.com

```

This runs Certbot with the `--nginx` plugin, using `-d` to specify the names we'd like the certificate to be valid for.

If this is your first time running Certbot, you'll be prompted to enter an email address and agree to the terms of service. After doing so, certbot will communicate with the Let's Encrypt server, then run a challenge to verify that you control the domain you're requesting a certificate for.

If that's successful, `certbot` will ask how you'd like to configure your HTTPS settings.

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

Select your choice then hit `ENTER`. The configuration will be updated, and Nginx will reload to pick up the new settings

The redirect option (Option 2) is the suggested choice. This will ensure that your website is always served over `https://`.

Your site is now being served over HTTPS!

Enter your domain into your browser's address bar and check it out:

URL

```bash
https://your_domain

```

Your certificates are now downloaded, installed, and loaded. And notice that your website is now being served over HTTPS.

### Verify Certbot Auto-Renew

Let's Encrypt's certificates are only valid for 90 days. This is to encourage users to automate their certificate renewal process.

The Certbot package we installed takes care of this for us by running `certbot renew` twice a day via a `systemd` timer. On non-systemd distributions, this functionality is provided by a script placed in `/etc/cron.d`. This task runs twice a day and will renew any certificate that's within thirty days of expiration.

To test the renewal process, you can do a dry run with Certbot:

Command

```bash
sudo certbot renew --dry-run

```

If you see no errors, you're all set.

When necessary, Certbot will renew your certificates and reload Nginx to pick up the changes. If the automated renewal process ever fails, Let’s Encrypt will send a message to the email you specified, warning you when your certificate is about to expire.

## Step 5 - Configure The Node.js Application

You should now have a server running with Nginx and a domain with HTTPS/SSL encryption.

Now you are ready to install Node.js and configure your application.

### Install Node.js

We will install the latest LTS release of Node.js, using the [NodeSource](https://github.com/nodesource/distributions) package archives.

First, you need to install the NodeSource PPA to get access to its contents. We'll use `curl` to retrieve the installation script for the Node.js `16.x` archives.

First, make sure you have navigated to your home directory:

Command

```bash
cd ~

```

Then, you can retrieve the installation script:

Command

```bash
curl -sL https://deb.nodesource.com/setup_16.x -o nodesource_setup.sh

```

If you want to use a different version of Node.js, replace `16.x` with the version of Node.js you want to use.

And run the script using `sudo`:

Command

```bash
sudo bash nodesource_setup.sh

```

The `nodesource_setup.sh` script won't be needed again, so let's delete it:

Command

```bash
sudo rm nodesource_setup.sh

```

The PPA has now been added to your configuration and your local package cache will be automatically updated.

You can now install the Node.js package using `apt-get`:

Command

```bash
sudo apt-get install nodejs

```

The `nodejs` package contains the `nodejs` binary as well as `npm`, so you won't need to install NPM separately. But for some `npm` packages to work, you will need to install the `build-essential` package:

Command

```bash
sudo apt-get install build-essential

```

Node.js is now installed and ready to use. Let's get a Node.js application up and running!

### Create An Application

We'll use a simple application to get you started, which you can replace with your own application later on. For now, we'll create an application that simply returns `Hello from your app!` to any HTTP requests.

If you already have an application ready to go, you can skip the next section where we create a sample application.

#### Application Code

Navigate to your home directory:

Command

```bash
cd ~

```

Then, create and open an `app.js` file using `nano`:

Command

```bash
nano app.js

```

And add the following code to the `app.js` file:

app.js

```js
const http = require('http');

const server = http.createServer((req, res) => {
	res.writeHead(200, {'Content-Type': 'text/plain'});
	res.end('Hello from your app!\n');
});

server.listen(8080);

console.log('Server running at http://localhost:8080/');

```

When ran, the `app.js` application will simply listen on the `localhost` address and port `8080`, and return `Hello from your app!` with a `200` HTTP success code on each request.

Save the file and exit the editor (`CTRL-X`+`Y`+`ENTER`).

#### Test Application

To test your application, run the `app.js` file with the following command:

Command

```bash
nodejs app.js

```

You should get this output:

Output

```bash
Server running at http://localhost:8080/

```

To test the HTTP response for your application, open another terminal window on your server, and connect to `localhost` using `curl`:

Command

```bash
curl http://localhost:8080

```

You should the following message in the output:

Output

```bash
Hello from your app!

```

If you don't see the right output, make sure your application is running and configured to listen on the `8080` port.

Once you have confirmed the application is working, kill the application with `CTRL`+`C`.

### Install & Configure PM2

You are now ready to install and configure [PM2](https://github.com/Unitech/pm2), which is a process manager for Node.js applications. It will allow you to keep your Node.js application alive forever, reload it without downtime and help facilitate common system admin tasks.

#### Install PM2

Using `npm`, you can install PM2 on your server with the following command:

Command

```bash
sudo npm install -g pm2

```

The `-g` option tells `npm` to install the module globally. It will now be available across your server's system.

#### Manage Node.js Application With PM2

Let's get your application running using PM2. It's easy and simple to use.

First, use the `pm2 start` command to start running your `app.js` application in the background:

Command

```bash
pm2 start app.js

```

This also adds your application to PM2's process list, which is outputted every time an application is started. Here is what the output should look like:

Output

```bash
[PM2] Spawning PM2 daemon
[PM2] PM2 Successfully daemonized
[PM2] Starting app.js in fork_mode (1 instance)
[PM2] Done.
┌──────────┬────┬──────┬──────┬────────┬─────────┬────────┬─────────────┬──────────┐
│ App name │ id │ mode │ pid  │ status │ restart │ uptime │ memory      │ watching │
├──────────┼────┼──────┼──────┼────────┼─────────┼────────┼─────────────┼──────────┤
│ app      │ 0  │ fork │ 3524 │ online │ 0       │ 0s     │ 21.566 MB   │ disabled │
└──────────┴────┴──────┴──────┴────────┴─────────┴────────┴─────────────┴──────────┘
Use `pm2 show <id|name>` to get more details about an app

```

PM2 automatically adds an **App Name** and a PM2 **id** to your application, along with other information such as the **PID** of the process, its current state, and memory usage.

PM2 applications will be restarted automatically if the application crashes or is killed. But additional steps need to be taken for applications to start on system startup (reboot or boot). PM2 provides an easy way to do this with its `startup` command.

Run it with the following command:

Command

```bash
pm2 startup systemd

```

In the resulting output on the last line, there will be a command that you must run with superuser privileges:

Output

```bash
[PM2] Init System found: systemd
[PM2] You have to run this command as root. Execute the following command:
sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u bob --hp /home/bob

```

Copy and paste the command that was generated (same as above but with your username instead of `bob`) to have PM2 always start when your server is booted.

Your command will look similar to this:

Command

```bash
sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u bob --hp /home/bob

```

This will create a systemd **unit** that will run `pm2` for your user on boot. This `pm2` instance, in turn, will run `app.js`. To check the status of the new systemd unit, use the following command:

Command

```bash
systemctl status pm2-bob

```

For more commands and information on PM2, check out the [PM2 documentation page](https://pm2.keymetrics.io/docs/usage/pm2-doc-single-page/#cheatsheet) on their website.

## Step 6 - Setup Nginx As A Reverse Proxy

Now that your application is running and listening on `localhost`, you need to make it so people from the outside world can access it. To achieve this, we will use Nginx as a [reverse proxy](https://en.wikipedia.org/wiki/Reverse_proxy).

First, you need to update the `/etc/nginx/sites-available/default` configuration file.

Open the file with this command:

Command

```bash
sudo nano /etc/nginx/sites-available/default

```

Within the server block, find the `location /` section and replace the contents of the block with the following configuration:

/etc/nginx/sites-available/default

```nginx
. . .

location / {
    proxy_pass http://localhost:8080;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
}

. . .

```

The new configuration you just added tells the Nginx server to respond to requests at its root. Assuming the server is available at `example.com`, accessing `https://example.com` via a web browser will send the request to `app.js` on port `8080` at `localhost`.

Save and exit the file when you are finished making the changes.

Test to make sure your Nginx configuration file is clear of any errors:

Command

```bash
sudo nginx -t

```

If no errors were found, restart Nginx:

Command

```bash
sudo systemctl restart nginx

```

Assuming that your Node.js application is running, and your application and Nginx configurations are correct, you should now be able to access your application via the Nginx reverse proxy.

Try it out by viewing the `www.example.com` web page in your browser.

You should see the `Hello from your app!` message displayed on the page. This means your Node.js application is up and running!

That was the last step for this guide.
