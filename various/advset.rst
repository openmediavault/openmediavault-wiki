Advanced Configuration
======================

|omv| is not a replacement for webmin, where you can configure all options in
the web interface. Options are already preconfigured to make it easier for the
average user to install and start using the NAS server.

As mentioned before in the :doc:`FAQ </FAQ>` |omv| takes full control of some
services, making difficult to intervene configuration files. Changes manually
added to configuration files will eventually rewritten at some stage by the
system. The list of common files not to intervene is listed :doc:`here </various/conffiles>`.

To overcome this there are some options available to modify some of the default
|omv| configuration options and values, like the use of environmental variables.

Environmental Variables
-----------------------

The web does not provide access to ALL the configuration aspects of a complex
system like |omv|. However, the system allows to change some advanced settings
through the use of environment variables. To set or change these variables,
login to you |omv| system using SSH and edit the file:

:file:`/etc/default/openmediavault`

Put the variable you want to change at the end of the file with the new value.
Ensure the value is declared with double quotes.

For example we are going to change the default sftp server for SSH service.

:code:`OMV_SSHD_SUBSYSTEM_SFTP=“/usr/lib/openssh/sftp-server”`

Make your changes, save, restart engined service openmediavault-engined restart
and run omv-mkconf ssh, finally reload the SSH service via |webui| or systemd.

The mkconf folder
-----------------

The other advanced configuration is the use of :code:`mkconf/{service}.d/`
folder type.

The mkconf scripts bundled in |omv| core and plugins are in :code:`/usr/share/openmediavault/mkconf/{service}.d/` folder. Those scripts are executed by run-parts in alphabetical order when a change is done in the webUI so the configuration is piped to the corresponding .conf file on their respective service.

These scripts are useful for placing extra configurations for services in a way
that server will not overwrite the files every time you change something in the
|webui|.


Examples using mkconf folders
-----------------------------

Network interfaces file
	The file ``/etc/network/interfaces`` will be (re-)generated by |omv| on
	demand. Thus custom changes that are done by the user will get lost. To
	prevent this, the config generation supports custom scripts to add
	additional configuration to the '/etc/network/interfaces' file when |omv|
	is generating it. Using this new feature it is no problem to add bridge or
	VLAN configurations.
	To do that a script must be located at /usr/share/openmediavault/mkconf/interfaces.d/.
	and has to be executable. The script should look like the following::

		#!/bin/sh
		#
		# This file is part of OpenMediaVault.
		#
		# @license   http://www.gnu.org/licenses/gpl.html GPL Version 3
		# @author    Volker Theile <volker.theile@openmediavault.org>
		# @copyright Copyright (c) 2009-2015 Volker Theile
		#
		# OpenMediaVault is free software: you can redistribute it and/or modify
		# it under the terms of the GNU General Public License as published by
		# the Free Software Foundation, either version 3 of the License, or
		# any later version.
		#
		# OpenMediaVault is distributed in the hope that it will be useful,
		# but WITHOUT ANY WARRANTY; without even the implied warranty of
		# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
		# GNU General Public License for more details.
		#
		# You should have received a copy of the GNU General Public License
		# along with OpenMediaVault. If not, see <http://www.gnu.org/licenses/>.

		set -e

		. /etc/default/openmediavault
		. /usr/share/openmediavault/scripts/helper-functions

		OMV_INTERFACES_CONFIG=${OMV_INTERFACES_CONFIG:-"/etc/network/interfaces"}

		cat <<EOF >> ${OMV_INTERFACES_CONFIG}
		# The loopback network interface
		auto lo
		iface lo inet loopback
		iface lo inet6 loopback
		EOF

Reference: http://forums.openmediavault.org/index.php/Thread/7355-Customize-etc-network-interfaces-the-OMV-way-1-11/

Samba
	Another example was this script published in the forum as a `guide <http://forums.openmediavault.org/index.php/Thread/11607-Samba-access-based-share-enum-workaround-for-workgroups-Hide-shares-that-users-d/>`_
	for samba. The intention of the user was to hide samba shares (not
	browsable) to users who did not have privileges to login into that shared
	folder. So basically the script will read the valid users list and will
	attempt to create as many files as valid users, appending the username
	variable to the end.