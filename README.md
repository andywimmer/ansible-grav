# Ansible Grav

I'm learning **[Ansible](https://www.ansible.com/)** and decided to make a playbook which installs the popular **[Grav](https://getgrav.org/)** flat-file CMS, which is [open-source](https://github.com/getgrav/grav). The installation and configuration are based on my production setup that hosts my personal blog.

This playbook installs NGINX, PHP7 and Grav. It also installs [required](https://learn.getgrav.org/basics/requirements#php-requirements) and recommended PHP modules and tweaks php.ini and [NGINX configs](https://learn.getgrav.org/webservers-hosting/local/nginx) based on Grav recommendations. These configs come from what appear in the Grav documentation.

The latest versions of **Ubuntu**, **Debian** and **Fedora** are currently supported. Read the [QA](#qa) section for specific server scenarios.

Note that Ubuntu ships with Python3 only, so targets appearing in the hosts file must look like:  

`server-ip ansible_python_interpreter=/usr/bin/python3`

This is the only way Ansible will be able to connect to a freshly spun-up Ubuntu target. Debian 8 targets threw an error at that parameter during QA, however all other distros were fine with using python3.

# Usage

### Install Ansible on your host or 'control' machine

*   [macOS](http://docs.ansible.com/ansible/intro_installation.html#latest-releases-via-pip):
    1.  `$ sudo easy_install pip`
    2.  `$ sudo pip install ansible`
*   [via Apt for Ubuntu](http://docs.ansible.com/ansible/intro_installation.html#latest-releases-via-apt-ubuntu)
    1.  `$ sudo apt install ansible`
    *   or  
    1.  `$ sudo apt install software-properties-common`
    2.  `$ sudo apt-add-repository ppa:ansible/ansible`
    3.  `$ sudo apt update`
    4.  `$ sudo apt install ansible`
*   [FreeBSD](http://docs.ansible.com/ansible/intro_installation.html#latest-releases-via-pkg-freebsd)
    *   `$ sudo pkg install ansible`
*   [via Apt for Debian](http://docs.ansible.com/ansible/intro_installation.html#latest-releases-via-apt-debian)
*   [more](http://docs.ansible.com/ansible/intro_installation.html#installing-the-control-machine)

### Clone or download this repo

`$ cd /your/dev/folder`

`$ git clone https://github.com/andywimmer/ansible-grav.git && cd ansible-grav`

### Modify hosts file

Changing `server-ip` to your target IP:

```
[prod]
server-ip ansible_python_interpreter=/usr/bin/python3
```

### Specify your remote user and private key file in ansible.cfg

If you are using a specific user (e.g. 'ubuntu' for AWS) or SSH key for your VPS, specify them here:

```
[defaults]
inventory = hosts
remote_user = root
private_key_file = /path/to/.ssh/key_rsa
```

### Test Ansible target connection

`$ cd /path/to/ansible-grav`

`$ ansible -m ping all`

Should result in:

```
XX.XX.XX.XX | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

### Run playbook

1.   `$ ansible-playbook provisions.yml`

2.   You will be prompted to enter your FQDN such as `domain.com` or `subdomain.example.xyz`

3.   Installation will proceed

# Post-installation

1.   Assuming you have configured your DNS to point to your target machine, use your browser to navigate to your FQDN - otherwise use your target's IP address.

2.   The Grav Admin will prompt you to create an admin user. Fill out the form and continue to the dashboard.

3.   Navigate to Configuration > System > Caching section and set Cache driver to APCu.

4.   Navigate to Plugins and click Update All Plugins.

5.   Use your browser to navigate to your FQDN or IP address to view the default Grav welcome page.

6.   Get [Grav'n](https://learn.getgrav.org/)!

# Grav notes

*   Fully explore the Configuration tabs and hover over the option title to display a bit more info on that setting.

*   Enabling [Markdown extra](https://michelf.ca/projects/php-markdown/extra/) in Configuration > System > Markdown section is recommended.

*   Caching is critical and Grav flushes caches upon system changes, so you will probably never want to disable caching - however the setting is there in Configuration > System > Caching.

*   If you aren't developing a custom theme for Grav or otherwise modifying the CSS or JS, you should enable those two pipelines under the Configuration > System > Assets section.

  *   The Configuration > Site tab is where you may specify a site title, author name & email address, custom taxonomies, etc.

*   The Configuration > Info tab serves as a phpinfo();

# Caveats

Recall the first three words of this readme.

This playbook does not create any users or lock down your sshd_config by disabling root login or password authentication - all of which are recommended if using for production.

**This playbook deletes /var/www/html** prior to installing Grav. For a fresh VPS you're probably fine with that - but you know, warning anyway.

This playbook does not install Grav with SSL enabled in the NGINX site. Read the  [documentation](https://learn.getgrav.org/webservers-hosting/local/nginx#using-ssl-with-an-existing-certificate) for more information on enabling SSL with an origin certificate.

`become: true` is peppered throughout this playbook to support AWS and the default 'ubuntu' user. I haven't looked, but if there's a better way to do this on a global level that would be great.

# QA

This playbook has been tested under the following scenarios:

### Distros

*   Ubuntu Server 16.04, 16.04.2, 16.10 & 17.04
*   Debian 8, 8.8 & 9.0
*   Fedora 25

### Hosts

*   [AWS](https://aws.amazon.com/) EC2
*   [DigitalOcean](https://www.digitalocean.com/) Compute droplets
*   [Vultr](https://www.vultr.com/) VC2
