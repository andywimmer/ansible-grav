# Ansible Grav

I'm learning **Ansible** and decided to make a playbook which installs the popular **[Grav](https://getgrav.org/)** CMS, which is [open-source](https://github.com/getgrav/grav). The installation and configuration is based on my production setup that hosts my personal blog.

The target of this playbook is **Ubuntu Server** and I haven't tested any other distros yet. Ubuntu ships with Python3 only, so targets appearing in the hosts file must look like:  

`127.0.0.1 ansible_python_interpreter=/usr/bin/python3`

This is the only way Ansible will be able to connect to a freshly spun-up Ubuntu VPS target.

This playbook installs NGINX, PHP7.1 and Grav. It also installs [required](https://learn.getgrav.org/basics/requirements#php-requirements) and recommended PHP modules and tweaks php.ini and [NGINX configs](https://learn.getgrav.org/webservers-hosting/local/nginx) based on Grav recommendations. These configs come from one of my other [repos](https://github.com/andywimmer/grav-nginx-configs) for now, but are basic copies of what appear in the Grav documentation.

## Usage

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

Changing `server-1-ip` to your target IP.

```
[dev]
# server-0-ip

[prod]
server-1-ip ansible_python_interpreter=/usr/bin/python3
# server-2-ip
```

### Specify your private key file in ansible.cfg (optional)

If you are using a specific SSH key for your VPS, uncomment the `private_key_file` line and provide the path.

```
[defaults]
inventory = hosts
remote_user = root
# private_key_file = /path/to/.ssh/key_rsa
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

## Post-installation

1.   Assuming you have configured your DNS to point to your target machine, use your browser to navigate to your FQDN, otherwise use your target's IP address.

2.   The Grav Admin will prompt you to create an admin user. Fill out the form and continue to the dashboard.

3.   Navigate to Configuration > System > Caching section and set Cache driver to APCu.

4.   Navigate to Plugins and click Update All Plugins.

5.   Use your browser to navigate to your FQDN or IP address to view the default Grav welcome page.

6.   Get Grav'n!

## Caveats

This playbook does not create any users or lock down your sshd_config by disabling root login or password authentication - all of which are recommended if using for production.

**This playbook deletes /var/www/html** prior to installing Grav. For a fresh VPS you're probably fine with that - but you know - warning anyway.

This playbook does not install Grav with SSL enabled in the NGINX site. Read the [documentation](https://learn.getgrav.org/webservers-hosting/local/nginx#using-ssl-with-an-existing-certificate) for more information on enabling SSL with an origin certificate.
