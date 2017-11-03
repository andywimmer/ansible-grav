# Ansible Grav

This playbook installs NGINX, PHP7 and Grav. It also installs [required](https://learn.getgrav.org/basics/requirements#php-requirements) and recommended PHP modules and tweaks php.ini and [NGINX configs](https://learn.getgrav.org/webservers-hosting/local/nginx) based on Grav recommendations. These configs come from what appear in the Grav documentation.

Master branch will install Grav Admin 'Vanilla' single page site. If you would like to experiment with the **[Gantry 5](http://gantry.org/)** Helium Skeleton site, you will need to clone that branch instead. Instructions are [below](#clone-repo).

The latest versions of **Amazon Linux**, **CentOS**, **Debian**, **Fedora** and **Ubuntu** are currently supported, though there are a few inconsistencies in the configuration (specific PHP7 version, inclusion of YAML parser, etc). Check the QA section for an up-to-date distro/host compatibility matrix. If you don't care what distro you run I would recommend Ubuntu 16.04/16.04.2 at this time.

*   [Pre-installation](#pre-installation)
*   [Usage](#usage)
*   [Post-installation](#post-installation)
*   [Grav notes](#grav-notes)
*   [Caveats](#caveats)
*   [QA](#qa)

# Pre-installation

### Install Ansible on your host or 'control' machine per the [instructions](http://docs.ansible.com/ansible/intro_installation.html#installing-the-control-machine)

*   macOS: `$ sudo easy_install pip && sudo pip install ansible`

# Usage

### Clone repo

Two branches are available:
*   Master will install **Grav Admin 'Vanilla'** single-page site
*   Gantry will install **Grav [Gantry 5](http://gantry.org/) Helium Skeleton** site

#### Grav Admin 'Vanilla'

1.  `$ cd /your/dev/folder`

2.  `$ git clone https://github.com/andywimmer/ansible-grav.git && cd ansible-grav`

#### Grav Gantry 5 Helium Skeleton

1.  `$ cd /your/dev/folder`

2.  `$ git clone -b gantry https://github.com/andywimmer/ansible-grav.git && cd ansible-grav`

### Modify hosts file

```
[prod]
server-ip domain= ansible_python_interpreter=/usr/bin/python3
```

1.  Change `server-ip` to target IP

2.  Add FQDN after `domain=` such as:

    *   `domain=yourdomain.com`

    *   or

    *   `domain=subdomain.example.xyz`

    *   **do not include the 'www'**

Note that some targets (like Ubuntu) ship with Python3 only, so the hosts file must contain the parameter `ansible_python_interpreter=/usr/bin/python3`

### Modify ansible.cfg file

```
[defaults]
inventory=hosts
stdout_callback=skippy
remote_user=root
private_key_file=/path/to/.ssh/key_rsa
```

1.  Change `remote_user` if necessary

2.  Change `private_key_file` accordingly

### Test Ansible target connection

1.  `$ cd /path/to/ansible-grav`

2.  `$ ansible -m ping all`

Should result in:

```
XX.XX.XX.XX | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

### Run playbook

*   `$ ansible-playbook provisions.yml`

# Post-installation

1.  Navigate to FQDN or IP.

2.  Grav Admin will prompt you to create an admin user. Fill out the form and continue to the dashboard.

3.  The dashboard my display purple bar(s) indicating update(s) to Grav and/or Plugins are available. Ensure Grav update is applied before attempting to update Plugins via Dashboard > Plugins > Update All Plugins.

4.  Navigate to Configuration > System > Caching section and set Cache driver to APCu.

5.  Use your browser to navigate to your FQDN or IP address to view the default Grav welcome page.

6.  Get [Grav'n](https://learn.getgrav.org/)!

# Grav notes

*   Fully explore the Configuration tabs and hover over the option title to display a bit more info on that setting.

*   Enabling [Markdown extra](https://michelf.ca/projects/php-markdown/extra/) in Configuration > System > Markdown section is recommended.

*   Caching is critical and Grav flushes caches upon system changes, so you will probably never want to disable caching - however the setting is there in Configuration > System > Caching.

*   If you aren't developing a custom theme for Grav or otherwise modifying the CSS or JS, you should enable those two pipelines under the Configuration > System > Assets section.

*   The Configuration > Site tab is where you may specify a site title, author name & email address, custom taxonomies, etc.

*   The Configuration > Info tab serves as a phpinfo();

# Caveats

This playbook does not create any users or lock down your sshd_config by disabling root login or password authentication - all of which are recommended if using for production.

**This playbook deletes /var/www/html** prior to installing Grav. For a fresh VPS you're probably fine with that - but you know, warning anyway. Also /var/www/html is the default webroot regardless of the standard distribution defaults. This will likely change.

This playbook does not install Grav with SSL enabled in the NGINX site. Read the  [documentation](https://learn.getgrav.org/webservers-hosting/local/nginx#using-ssl-with-an-existing-certificate) for more information on enabling SSL with an origin certificate.

# QA

I have a deep QA background and test heavily before pushing commits. If you find a bug please file a ticket in the issue tracker and I will investigate 🤓

The following matrix outlines current distro/host compatibility for this playbook. Click ❌s to view the ticket in the issue tracker. Version numbers displayed come from the host's UI.

These are the only hosts I have accounts with currently - they seem like popular ones. If you run this playbook on a different host/distro combination with success _or_ failure, please let me know in the issue tracker, or send me a referral to said host so I can verify.

|                        | AWS | DigitalOcean | Google Cloud | Vultr |
|------------------------|:---:|:------------:|:------------:|:-----:|
| Amazon Linux 2017.03.1 |  ✅ |       ⚫      |      ⚫      |   ⚫  |
| CentOS '7'             |  ⚫ |       ⚫      |      ✅      |  [❌](https://github.com/andywimmer/ansible-grav/issues/4)  |
| CentOS 7.3.1611        |  ⚫ |       ✅      |      ⚫      |   ⚫  |
| Debian '8'             |  ⚫ |       ⚫      |      ✅      |   ✅  |
| Debian 8.8             |  ⚫ |       ✅      |      ⚫      |   ⚫  |
| Debian 9.0             |  ⚫ |       ✅      |      ✅      |   ✅  |
| Fedora 25              |  ⚫ |       ✅      |      ⚫      |  [❌](https://github.com/andywimmer/ansible-grav/issues/2)  |
| Ubuntu 16.04           |  ✅ |       ⚫      |      ✅      |   ✅  |
| Ubuntu 16.04.2         |  ⚫ |       ✅      |      ⚫      |   ⚫  |
| Ubuntu 16.10           |  ⚫ |       ✅      |      ✅      |   ✅  |
| Ubuntu 17.04           |  ⚫ |       ✅      |      ✅      |   ✅  |
