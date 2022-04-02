# NextCloud
How To Install and Configure Nextcloud on Ubuntu 20.04

Prerequisites
In order to complete the steps in this guide, you will need the following:

A non-root sudo-enabled user and firewall configured on your server: You can create a user with sudo privileges and set up a basic firewall by following the Initial Server Setup with Ubuntu 20.04.
(Optional) A domain name pointed to your server: We will be securing connections to the Nextcloud installation with TLS/SSL. Nextcloud can set up and manage a free, trusted SSL certificate from Let’s Encrypt if your server has a domain name. If not, Nextcloud can set up a self-signed SSL certificate that can encrypt connections, but won’t be trusted by default in web browsers. If you are using DigitalOcean, you can read our DNS documentation to learn how to add domains to your account and manage DNS records, if you intend to use Let’s Encrypt.


**Step 1 – Installing Nextcloud**
We will be installing Nextcloud using the Snap packaging system. This packaging system, available on Ubuntu 20.04 by default, allows organizations to ship software, along with all associated dependencies and configuration, in a self-contained unit with automatic updates. This means that instead of installing and configuring a web and database server and then configuring the Nextcloud app to run on it, we can install the snap package which handles the underlying systems automatically.

To download the Nextcloud snap package and install it on the system, type:

`sudo snap install nextcloud`

The Nextcloud package will be downloaded and installed on your server. You can confirm that the installation process was successful by listing the changes associated with the snap:

`snap changes nextcloud`

he status and summary indicate that the installation was completed without any problems.

Getting Additional Information About the Nextcloud Snap
If you’d like some more information about the Nextcloud snap, there are a few commands that can be helpful.

The snap info command can show you the description, the Nextcloud management commands available, as well as the installed version and the snap channel being tracked:

`snap info nextcloud`

Snaps can define connections they support, which consist of a slot and plug that, when hooked together, gives the snap access to certain capabilities or levels of access. For instance, snaps that need to act as a network client must have the network connection. To see what snap connections this snap defines, type:

`snap connections nextcloud`

To learn about all of the specific services and apps that this snap provides, you can take a look at the snap definition file by typing:

`cat /snap/nextcloud/current/meta/snap.yaml`

This will allow you to see the individual components included within the snap, if you need help with debugging.

**Step 2 – Configuring an Administrative Account**
There are a few different ways you can configure the Nextcloud snap. In this guide, rather than creating an administrative user through the web interface, we will create one on the command line in order to avoid a small window where the administrator registration page would be accessible to anyone visiting your server’s IP address or domain name.

To configure Nextcloud with a new administrator account, use the nextcloud.manual-install command. You must pass in a username and a password as arguments:

`sudo nextcloud.manual-install sammy password`

The following message indicates that Nextcloud has been configured correctly:

> Output
> Nextcloud was successfully installed
> Now that Nextcloud is installed, we need to adjust the trusted domains so that Nextcloud will respond to requests using the server’s domain name or IP address.
> 
{.is-info}

**Step 3 – Adjusting the Trusted Domains**
When installing from the command line, Nextcloud restricts the host names that the instance will respond to. By default, the service only responds to requests made to the “localhost” hostname. We will be accessing Nextcloud through the server’s domain name or IP address, so we’ll need to adjust this setting to accept these type of requests.

You can view the current settings by querying the value of the trusted_domains array:

`sudo nextcloud.occ config:system:get trusted_domains`

> Output
> localhost
> Currently, only localhost is present as the first value in the array. We can add an entry for our server’s domain name or IP address by typing:
{.is-info}

`
sudo nextcloud.occ config:system:set trusted_domains 1 --value=example.com`
 
> Output
> System config value trusted_domains => 1 set to string example.com
{.is-info}


If we query the trusted domains again, we will see that we now have two entries:

`sudo nextcloud.occ config:system:get trusted_domains`

> Output
> localhost
> example.com
{.is-info}


If you need to add another way of accessing the Nextcloud instance, you can add additional domains or addresses by rerunning the config:system:set command with an incremented index number (the “1” in the first command) and adjusting the --value.

**Step 4 – Securing the Nextcloud Web Interface with SSL**
Before we begin using Nextcloud, we need to secure the web interface.

If you have a domain name associated with your Nextcloud server, the Nextcloud snap can help you obtain and configure a trusted SSL certificate from Let’s Encrypt. If your Nextcloud server does not have a domain name, Nextcloud can configure a self-signed certificate which will encrypt your web traffic but won’t be automatically trusted by your web browser.

With that in mind, follow the section below that matches your scenario.

Option 1: Setting Up SSL with Let’s Encrypt
---

If you have a domain name associated with your Nextcloud server, the best option for securing your web interface is to obtain a Let’s Encrypt SSL certificate.

Start by opening the ports in the firewall that Let’s Encrypt uses to validate domain ownership. This will make your Nextcloud login page publicly accessible, but since we already have an administrator account configured, no one will be able to hijack the installation:

`sudo ufw allow 80,443/tcp`

Next, request a Let’s Encrypt certificate by typing:

`sudo nextcloud.enable-https lets-encrypt`

You will first be asked whether your server meets the conditions necessary to request a certificate from the Let’s Encrypt service:

> Output
> In order for Let's Encrypt to verify that you actually own the
> domain(s) for which you're requesting a certificate, there are a
> number of requirements of which you need to be aware:
> 
> 1. In order to register with the Let's Encrypt ACME server, you must
>    agree to the currently-in-effect Subscriber Agreement located
>    here:
> 
>        https://letsencrypt.org/repository/
> 
>    By continuing to use this tool you agree to these terms. Please
>    cancel now if otherwise.
> 
> 2. You must have the domain name(s) for which you want certificates
>    pointing at the external IP address of this machine.
> 
> 3. Both ports 80 and 443 on the external IP address of this machine
>    must point to this machine (e.g. port forwarding might need to be
>    setup on your router).
> 
> Have you met these requirements? (y/n)
> Type y to continue.
{.is-info}


Next, you will be asked to provide an email address to use for recovery operations:

> Output
> Please enter an email address (for urgent notices or key recovery):
> Enter your email and press Enter to continue.
{.is-info}


Finally, enter the domain name associated with your Nextcloud server:

> Output
> Please enter your domain name(s) (space-separated): example.com
> Your Let’s Encrypt certificate will be requested and, provided everything went well, the internal Apache instance will be restarted to immediately implement SSL:
{.is-info}


> Output
> Attempting to obtain certificates... done
> Restarting apache... done
{.is-info}


You can now skip ahead to the next step to sign into Nextcloud for the first time.

Option 2: Setting Up SSL with a Self-Signed Certificate
---

If your Nextcloud server does not have a domain name, you can still secure the web interface by generating a self-signed SSL certificate. This certificate will allow access to the web interface over an encrypted connection, but will be unable to verify the identity of your server, so your browser will likely display a warning.

To generate a self-signed certificate and configure Nextcloud to use it, type:

`sudo nextcloud.enable-https self-signed`

> Output
> Generating key and self-signed certificate... done
> Restarting apache... done
> 
{.is-info}

The above output indicates that Nextcloud generated and enabled a self-signed certificate.

Now that the interface is secure, open the web ports in the firewall to allow access to the web interface:

`sudo ufw allow 80,443/tcp`

You are now ready to log into Nextcloud for the first time.

**Step 5 – Logging in to the Nextcloud Web Interface**
Now that Nextcloud is configured, visit your server’s domain name or IP address in your web browser:

https://example.com
Note: If you set up a self-signed SSL certificate, your browser may display a warning that the connection is insecure because the server’s certificate is not signed by a recognized certificate authority. This is expected for self-signed certificates, so feel free to click through the warning to proceed to the site.



**Step 6 – Move Data New Location**

With Snap, you need to use the original data dir location. I made it work with the following mount:

> mount  /dev/sdb1/ /var/snap/nextcloud/common/nextcloud/data
> fstab:
> 
> /dev/sdb1 /var/snap/nextcloud/common/nextcloud/data ext4 defaults 0 1 mount -a

**Step 7 – Copy old path to new**
{.is-warning}


> The correct means of doing this is to use the -T (--no-target-directory) option, and recursively copy the folders (without trailing slashes, asterisks, etc.), i.e.:
> 
> cp -rT /etc/skel /home/user


**Step 8 – Firewall Ports Needed**
