---
author:
  name: Linode Community
  email: docs@linode.com
description: 'Odoo is an open-source suite of over 4,500 business applications. Odoo allows administrators to install, configure and customize any application to satisfy their needs. This guide covers how to install and configure Odoo using Git source so it will be easy to upgrade and maintain.'
keywords: 'Odoo,Odoo ERP,CMS,Ubuntu,CRM,OpenERP'
license: '[CC BY-ND 3.0](http://creativecommons.org/licenses/by-nd/3.0/us/)'
published: 'Tuesday, November 3rd, 2015'
modified: 'Tuesday, November 3rd, 2015'
modified_by:
  name: Linode
title: 'Install Odoo 9 ERP on Ubuntu 14.04'
contributor:
  name: Damaso Sanoja
  link: https://github.com/damasosanoja
external_resources:
 - '[Odoo User Documentation](https://doc.odoo.com/book/)'
---

*This is a Linode Community guide. Write for us and earn $250 per published guide.*
<hr>

[Odoo](https://www.odoo.com/) (formerly known as OpenERP) is an open-source suite of business applications that include: Customer Relationship Management, Sales Pipeline, Project Management, Manufacturing, Invoicing, Accounting, eCommerce, and Inventory, just to name a few. The Odoo team has created 31 main applications, from which community members have developed more than 4,500 others that address a wide range of business solutions.

The flexibility inherent to any Odoo deployment allows the administrator to install any combination of modules and configure/customize them at will to satisfy needs of businesses ranging from a small shop to an enterprise-level corporation.

<<<<<<< HEAD
This guide covers how to install and configure Odoo using Git source in a mere 35 minutes, so the application will be easy to upgrade, maintain and customize. 
=======
This guide covers how to install and configure Odoo in just 35 minutes using Git source so it will be easy to upgrade, maintain and customize. 
>>>>>>> dd14a6be4cf7f41e3ff513ac7fc5d415173bcf20

## Before You Begin

1.  Complete the [Getting Started](/docs/getting-started) guide.

2.  Follow the [Securing Your Server](/docs/security/securing-your-server/) guide to create a standard user account, strengthen SSH access and remove unnecessary network services. The Securing Your Server guide uses `sudo` wherever possible. Do **not** follow the *Configuring a Firewall* section in the Securing Your Server guide. This Odoo guide has instructions specifically for an Odoo production server.

3.  Log in to your Linode via SSH and check for updates using `apt-get` package manager:

		sudo apt-get update && sudo apt-get upgrade

##Open Corresponding Firewall Ports

In this case we're using Odoo's default port 8069, but this could be any port you specify later in the configuration file:

	sudo ufw allow ssh
	sudo ufw allow 8069/tcp
	sudo ufw enable

##Install Database and Server Dependencies

Now, you're going to install the PostgreSQL database and other necessary server libraries using `apt-get`:

	sudo apt-get install subversion git bzr bzrtools python-pip postgresql postgresql-server-dev-9.3 python-all-dev python-dev python-setuptools libxml2-dev libxslt1-dev libevent-dev libsasl2-dev libldap2-dev pkg-config libtiff5-dev libjpeg8-dev libjpeg-dev zlib1g-dev libfreetype6-dev liblcms2-dev liblcms2-utils libwebp-dev tcl8.6-dev tk8.6-dev python-tk libyaml-dev fontconfig

###Create Odoo User and Log Directory

1.  Create the Odoo system user:

		sudo adduser --system --home=/opt/odoo --group odoo

2.  Create the log directory:

		sudo mkdir /var/log/odoo

{: .note}
>
>When running multiple Odoo versions on the same Linode, you may want to assign different users and directories for each instance.

###Install Odoo Server Files from Source

1.  Change to the Odoo directory, in your case:

		cd /opt/odoo/

2.  Clone the Odoo files on your server:

		sudo git clone https://www.github.com/odoo/odoo --depth 1 --branch 9.0 --single-branch .

{: .note}
>
>Using Git brings greater flexibility because any time a new upgrade is available you only need to pull that branch; you can even install a different upgrade alongside the production version by changing the destination directory and the  `--branch X.x` flag. Before doing any operation, remember to perform a full backup of your database and custom files.

###Create PostgreSQL User 

1.  Switch to `postgres` user:

		sudo su - postgres

2.  But if you're deploying a Production server, you may want to set a strong password for the database user, so:

		createuser odoo -U postgres -dRSP

3.  You'll be prompted for a password. **Save it**, you'll need it shortly.

	{: .note}
	>
	>Within a testing or development environment you could create a user with no password using `createuser odoo -U postgres -dRS`.

4.  Press **CTRL+D** to exit from `postgres` user session.

{: .note}
>
>If you want to run multiple Odoo instances on the same Linode, remember to check pg_hba.conf and change it according to your needs.

##Specific Dependencies for Odoo Applications 

Using `pip` instead of `apt-get` will guarantee that your installation has the correct versions needed. We'll also desist from using Ubuntu's packaged versions of [Wkhtmltopdf](http://wkhtmltopdf.org/) and [node-less](http://lesscss.org/).

###Install Python Dependencies

Install Python libraries using the following commands:

	sudo pip install -r /opt/odoo/doc/requirements.txt
	sudo pip install -r /opt/odoo/requirements.txt


###Install Less CSS via nodejs and npm

1.  Download `nodejs` installation script from nodesource:

		wget -qO- https://deb.nodesource.com/setup | sudo bash -

2.  Now that your repository list is updated, install `nodejs` using `apt-get`:

		sudo apt-get install nodejs

3.  Next, install a newer version of Less via `npm`:

		sudo npm install -g less less-plugin-clean-css

###Install Updated Wkhtmltopdf Version

1.  Switch to a temporary directory of your choice:  

		cd /tmp

2.  Download the recommended version of wkhtmltopdf for Odoo server, currently **0.12.1**:

		sudo wget http://download.gna.org/wkhtmltopdf/0.12/0.12.1/wkhtmltox-0.12.1_linux-trusty-amd64.deb

3.  Install the package using `dpkg`:

		sudo dpkg -i wkhtmltox-0.12.1_linux-trusty-amd64.deb

4.  For a properly functioning Wkhtmltopdf, we'll need to copy the binaries to an adequate location:

		sudo cp /usr/local/bin/wkhtmltopdf /usr/bin
		sudo cp /usr/local/bin/wkhtmltoimage /usr/bin

##Odoo Server Configuration

1.  Copy the included configuration file to a more convenient location, changing its name to `odoo-server.conf`:

		sudo cp /opt/odoo/debian/openerp-server.conf /etc/odoo-server.conf

2.  Next, we need to modify the configuration file. The finished file should look similar to this, depending on your deployment needs:

	{: .file}
	/etc/odoo-server.conf
	:   ~~~ conf
		[options]
		admin_passwd = admin
		db_host = False 
		db_port = False
		db_user = odoo
		db_password = <PostgreSQL_user_password>
		addons_path = /opt/odoo/addons
		logfile = /var/log/odoo/odoo-server.log
		xmlrpc_port = 8069
    	~~~

	*  `admin_passwd = admin` This is the password that allows database operations.
	*  `db_host = False` Unless you plan to connect to a different database server address, leave this line untouched.
	*  `db_port = False` Odoo uses PostgreSQL default port 5432; change only if necessary.
	*  `db_user = odoo` Database user; in this case, we used the default name.
	*  `db_password =` The previously created PostgreSQL user password.
	*  `addons_path =` You need to modify this line to read: `addons_path = /opt/odoo/addons`. Add `</path/to/custom/modules>` if needed.
	*  You need to include the path to logfiles by adding a new line: `logfile = /var/log/odoo/odoo-server.log`
	*  Optionally, we could include a new line specifying the Odoo Frontend port used for connection: `xmlrpc_port = 8069`. Note: This only makes sense if you're planning to run multiple Odoo instances (or versions) on the same server. For normal installation you can skip this line and Odoo will connect by default to port 8069.

###Odoo Boot Script

Your next step is to create a boot script, `odoo-server`, to gain control over Odoo behavior and use it at server startup and shutdown.

{: .file}
/etc/init.d/odoo-server
:   ~~~ shell
	#!/bin/sh
	### BEGIN INIT INFO
	# Provides: odoo-server
	# Required-Start: $remote_fs $syslog
	# Required-Stop: $remote_fs $syslog
	# Should-Start: $network
	# Should-Stop: $network
	# Default-Start: 2 3 4 5
	# Default-Stop: 0 1 6
	# Short-Description: Odoo ERP
	# Description: Odoo is a complete ERP business solution.
	### END INIT INFO

	PATH=/bin:/sbin:/usr/bin
	# Change the Odoo source files location according your needs.
	DAEMON=/opt/odoo/openerp-server
	# Use the name convention of your choice 
	NAME=odoo-server
	DESC=odoo-server

	# Specify the user name (Default: odoo).
	USER=odoo

	# Specify an alternate config file (Default: /etc/odoo-server.conf).
	CONFIGFILE="/etc/odoo-server.conf"

	# pidfile
	PIDFILE=/var/run/$NAME.pid

	# Additional options that are passed to the Daemon.
	DAEMON_OPTS="-c $CONFIGFILE"

	[ -x $DAEMON ] || exit 0
	[ -f $CONFIGFILE ] || exit 0

	checkpid() {
	[ -f $PIDFILE ] || return 1
	pid=`cat $PIDFILE`
	[ -d /proc/$pid ] && return 0
	return 1
	}

	case "${1}" in
	start)
	echo -n "Starting ${DESC}: "

	start-stop-daemon --start --quiet --pidfile ${PIDFILE} \
	--chuid ${USER} --background --make-pidfile \
	--exec ${DAEMON} -- ${DAEMON_OPTS}

	echo "${NAME}."
	;;

	stop)
	echo -n "Stopping ${DESC}: "

	start-stop-daemon --stop --quiet --pidfile ${PIDFILE} \
	--oknodo

	echo "${NAME}."
	;;

	restart|force-reload)
	echo -n "Restarting ${DESC}: "

	start-stop-daemon --stop --quiet --pidfile ${PIDFILE} \
	--oknodo

	sleep 1

	start-stop-daemon --start --quiet --pidfile ${PIDFILE} \
	--chuid ${USER} --background --make-pidfile \
	--exec ${DAEMON} -- ${DAEMON_OPTS}

	echo "${NAME}."
	;;

	*)
	N=/etc/init.d/${NAME}
	echo "Usage: ${NAME} {start|stop|restart|force-reload}" >&2
	exit 1
	;;
	esac

	exit 0
    ~~~

###Odoo Files Ownership and Permissions

1.  Change the `odoo-server` file permissions and ownership so only root can write to it, while odoo user will only be able to read and execute it:

		sudo chmod 755 /etc/init.d/odoo-server
		sudo chown root: /etc/init.d/odoo-server

2.  Since odoo user will run the application, change its ownership accordingly:

		sudo chown -R odoo: /opt/odoo/

3.  You should set odoo user as the owner of log directory, as well:

		sudo chown odoo:root /var/log/odoo

4.  Finally,  you should protect the server configuration file by changing its ownership and permissions so no other non-root user can access it:

		sudo chown odoo: /etc/odoo-server.conf
		sudo chmod 640 /etc/odoo-server.conf

##Testing the Server

1.  It's time to check that everything is working as expected. Let's start the Odoo server:

		sudo /etc/init.d/odoo-server start

2.  Let's take a look at log file to verify no errors occurred:

		cat /var/log/odoo/odoo-server.log

3.  Now, we can check if the server stops properly, too:

		sudo /etc/init.d/odoo-server stop

4.  Enter the same command again:

		cat /var/log/odoo/odoo-server.log

##Running Boot Script at Server Startup and Shutdown

1.  If the Odoo server log doesn't indicate any problem, we can continue and make the boot script start and stop with the server:

		sudo update-rc.d odoo-server defaults

2.  It's a good idea to restart your Linode to see if everything is working properly:

		sudo shutdown -r now

3.  Once your Linode has been restarted, verify the log file one more time:

		cat /var/log/odoo/odoo-server.log

##Testing Odoo Frontend

1.  Open a new browser window and enter in the address bar:

		http://<your_domain_or_IP_address>:8069

2.  A screen similar to this will appear:

[![Odoo Db creation](/docs/assets/odoo_db_creation.png)](/docs/assets/odoo_db_creation.png)

3.  Congratulations! Now, you can create your first database and start using Odoo! 

[![Odoo applications](/docs/assets/odoo_applications.png)](/docs/assets/odoo_applications.png)
