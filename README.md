# fiwarelab-horizon-theme

This theme of Cloud Portal is based on Ubuntu theme for Openstack-dashboard. Version of Openstack dashboard should be Rocky
and Ubuntu Version should be 18.04 -- There are no big changes from this version to the previous one, representing Queens.

## Installation
After installing Ubuntu 18.04, we only need to run

    sudo apt update && sudo apt install -y software-properties-common
    sudo add-apt-repository cloud-archive:rocky
    apt update && apt dist-upgrade -y
    sudo apt install openstack-dashboard git


## System configurations
### Changes on system
I'd add in file **/etc/hosts** a couple of lines in order to  acceletate searchs in DNSs. So, I would add:

     10.0.3.145   keystone
     10.0.3.145   cloud.lab.fiware.org

### Apache configurations
I need to do som configurations in Apache so it can properly work. First, I need to set up on file **/etc/apache2/conf-available/openstack-dashboard.conf **-- The end version should be something like this:

    WSGIScriptAlias /horizon /usr/share/openstack-dashboard/openstack_dashboard/wsgi.py process-group=horizon
    WSGIScriptAlias /        /usr/share/openstack-dashboard/openstack_dashboard/wsgi.py process-group=horizon
    WSGIDaemonProcess horizon user=horizon group=horizon processes=3 threads=10 display-name=%{GROUP}
    WSGIProcessGroup horizon
    WSGIApplicationGroup %{GLOBAL}
    WSGIScriptReloading  On
    WSGIPassAuthorization On
    
    Alias /static /var/lib/openstack-dashboard/static/
    Alias /horizon/static /var/lib/openstack-dashboard/static/
    
    <Directory /usr/share/openstack-dashboard/openstack_dashboard>
        Require all granted
    </Directory>
    
    <Directory /var/lib/openstack-dashboard/static>
        Require all granted
    </Directory>
    
 Another file I should modify is the Apache configuration of the VirtualHosts. The file should be in /etc/apache2/:
          
          # Set some name to the ServerName
               ServerName cloud3.lab.fiware.org
            .....
           # Comment the next line    
           # DocumentRoot /var/www/html

And last configuration needed is being sure the there is configuration for **javascript-common.conf**:

     a2enconf javascript-common.conf

## Local Settings for Openstack dashboard configuration
We need to tweak the fie **/etc/openstack-dashboard/local_settings.py** as follows:

Uncomment API VERSIONS (This isn't likely to be needed):

    OPENSTACK_API_VERSIONS = {
        "data-processing": 1.1,
        "identity": 3,
        "image": 2,
        "volume": 2,
        "compute": 2,
    }

Configuraion of Keystone:

    OPENSTACK_HOST = "keystone"
    OPENSTACK_KEYSTONE_URL = "http://%s:4730/v3" % OPENSTACK_HOST
    OPENSTACK_KEYSTONE_DEFAULT_ROLE = "member"

Set up Session Engine

    SESSION_ENGINE='django.contrib.sessions.backends.cache'

And we reconfigure the plugins we want (in this case, our fiwarelab-horizon-theme):
 
    # Comment next line
    # DEFAULT_THEME = 'ubuntu'
    
    
    AVAILABLE_THEMES = [
       ( 'fiwarelab-horizon-theme', 'FiwareLab', 'themes/fiwarelab-horizon-theme'),
    ]
    
    
And setup WEBROOT, so we can access the cloud portal using */* instead of */horizon/* :

    WEBROOT='/'
    
## Installing the theme
To install the theme we have to clone the repo to **/usr/share/openstack-dashboard/openstack_dashboard/themes** -- The next steps are done using the root account:

        git clone https://github.com/jicarretero/fiwarelab-horizon-theme.git /usr/share/openstack-dashboard/openstack_dashboard/themes/fiwarelab-horizon-theme
    
Collect static info:
    
        python /usr/share/openstack-dashboard/manage.py collectstatic

 Compress files
 
        python /usr/share/openstack-dashboard/manage.py compress --force
     
And finally we must restart apache:
     
        systemctl restart apache2