Role Name
=========

An Ansible Role that installs the latest version of Netbox 
(DCIM from DigitalOcean folks)

Requirements
------------

Netbox require a postgresql database, this role will not setup this DB
so you should use an other role to setup and conf posgtres (in my case  
I use the geerlingguy postgres role)

Role Variables
--------------


Available variables are listed below, along with default values (see `defaults/main.yml`):

	netbox_app_dir: "/opt/netbox"
	
the default destination for netbox install


	netbox_user: "netbox"
	
Default unix user running Netbox through gunicorn


	netbox_admin: "admin"

Netbox admin user name (web admin)


	netbox_gunicorn_bind_addr: "0.0.0.0"
	netbox_gunicorn_bind_port: "8001"
	
The IP address and ports on which gunicorn should be listening.

	netbox_gunicorn_workers: "3"
	
Maximum number of gunicorn workers we allow

	netbox_httpd_user_group: "apache"
	
group of the frontend webserver you want to use (for the static files)


	netbox_login_required: False

Setting this to True will permit only authenticated users to access any 
part of NetBox. By default, anonymous users are permitted to access most 
data in NetBox

	netbox_smtp_server: "localhost"
	
Host name or IP address of the email server netbox will use


	netbox_from_email: "{{ netbox_user }}@{{ ansible_fqdn }}"
	
Sender address for emails sent by NetBox



the following variables are not in the defaults/main.yml and must be 
set in your playbook

	netbox_allowed_hosts: "'{{ ansible_fqdn }}'"
	
This is a list of the valid hostnames by which this server can be reached.
You must specify at least one name or IP address.


    netbox_db_name: "netbox"
    netbox_db_user:  "netbox"
    netbox_db_password:""
    
This parameter holds the database configuration details. You must define 
the username and password used when you configured PostgreSQL.

    netbox_secret_key: ""

Netbox secret key, generate a random secret key of at least 50 
alphanumeric characters. This key must be unique to this installation 
and must not be shared outside the local system.
  
  
Define the next variable if you need to authenticate your user against 
an ldap directory

    netbox_ldap_srv_uri: "ldap://ldap.example.net"
    netbox_ldap_base_dn: "dc=example,dc=be"
    netbox_ldap_bind_dn: "cn=netbox,dc=example,dc=be"
    netbox_ldap_user_search: "ou=People,dc=example,dc=be"

Dependencies
------------

A list of other roles hosted on Galaxy should go here, plus any details 
in regards to parameters that may need to be set for other roles, or 
variables that are used from other roles.

Example Playbook
----------------

Example using geerlingguy apache and postgresql to configure netbox

---
- hosts: all
  become: true
  vars_files:
    - vars/infra-tools.yml
  vars:
    netbox_allowed_hosts: "'{{ ansible_fqdn }}'"
    netbox_db_name: "netbox"
    netbox_db_user: "netbox"
    netbox_db_password: "ahwi@wohw5baNiet0iewae$Sh"
    netbox_secret_key: "epha0hee5Ciake7EiYaech5ree1Woh2oung6thiephaeZahGoh"
    netbox_from_email: "netbox.noreply@example.net"
    postgresql_databases:
      - name: netbox
        state: present
    postgresql_users:
      - name: "{{ netbox_db_user }}"
        password: "{{ netbox_db_password }}"
        db: "{{ netbox_db_name }}"
        role_attr_flags: NOSUPERUSER

    apache_vhosts_version: 2.4
         

    apache_vhosts:
      - servername: "{{ansible_fqdn}}"
        documentroot: "/var/www/html"
        extra_parameters: |
          Alias /static /opt/netbox/netbox/static
          ProxyPreserveHost On
          <Location /static>
            ProxyPass !
          </Location>
          ProxyPass / http://127.0.0.1:8001/
          ProxyPassReverse / http://127.0.0.1:8001/

  roles:
    - { role: geerlingguy.repo-epel }
    - { role: geerlingguy.postgresql }
    - { role: geerlingguy.apache }
    - { role: erkax.netbox }


License
-------

BSD

