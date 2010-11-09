nagios-plugin-xserve_raid Nagios plugin for checking Apple Xserve RAIDs
======================================================================


Features
--------

This plugin monitors many conditions on Apple Xserve RAID units.

It reports critical states for:

* power off
* enclosure buzzer
* fibre channel link state
* drive SMART errors
* any array status except online
* broken array RAID member slots

It reports warnings states for:

* array rebuild in progress (including percentage completion)
* offline array slots
* fan (blower)
* power
* temperature
* UPS

It reports unknown states for:

* controller starting up

In all cases it provides information on the configuration.


Requirements
------------

* Python >= 2.4, < 3.0


Basic Installation
------------------

1. Copy `check_xserve_raid` to your Nagios plugins directory.

2. If you are using Python 2.5 or earlier then copy `plistlib.py` to somewhere
that `check_xserve_raid` will find it, e.g. the Nagios plugins directory.
Python 2.6 and later include `plistlib.py` as part of the standard library.


Usage
-----

Syntax:
	check_xserve_raid -H <address> -c <controller> -u <username> -p <password-hash>

`<address>` is the IP address or hostname of the Xserve RAID network interface.

`<controller>` is either `top` or `bottom`.

`<username>` is the administration/monitoring username.

`<password-hash>` is the hashed administration/monitoring password.

The default username and password hash corresponds to the default
administration username and password of `guest` and `public`.  If you are not
using the default username or password then the easiest way to find these
values is by sniffing the network traffic between a computer running the Xserve
RAID Admin software and the Xserve RAID unit.  The software makes unencrypted
HTTP requests to the Xserve RAID unit, the values are in the `ACP-User` and
`ACP-Password:` headers.

If `check_xserve_raid` is running correctly, you should see output like:

	$ check_xserve_raid -H xraid1-top -c top -p xxxxxxxx
	XSERVE RAID OK: array 1: 2.73TB RAID5, online
	$ echo $?
	0


Example Nagios configuration
----------------------------

In my configuration I use the default monitoring username (`guest`) with a
non-default password.  Therefore, I define a `check_xserve_raid` command in
`commands.cfg`:

	define command {
		command_name	check_xserve_raid
		command_line	$USER1$/check_xserve_raid -H $HOSTADDRESS$ -c $ARG1$ -p $ARG2$
	}

I define two Nagios hosts, one for each network interface unit in the physical
Xserve RAID unit in `hosts.cfg`:

	# Xserve RAID 1, top controller network interface
	define host {
		host_name	xraid1-top
		# ...
	}

	# Xserve RAID 1, bottom controller network interface
	define host {
		host_name	xraid1-bottom
		# ...
	}

Then, for each physical Xserve RAID unit you will want check both controllers
from both network interfaces, for a total of four checks per physical unit.  I
define these four checks in `services.cfg`:

	define service {
		host_name                      xraid1-top
		check_command                  check_xserve_raid!top!xxxxxxxx
		# ...
	}

	define service {
		host_name                      xraid1-top
		check_command                  check_xserve_raid!bottom!xxxxxxxx
		# ...
	}

	define service {
		host_name                      xraid1-bottom
		check_command                  check_xserve_raid!top!xxxxxxxx
		# ...
	}

	define service {
		host_name                      xraid1-bottom
		check_command                  check_xserve_raid!bottom!xxxxxxxx
		# ...
	}


Bugs
----

There is little error handling.


License
-------

nagios-plugin-xserve_raid Nagios plugin for checking Apple Xserve RAIDs

Copyright &copy; 2010 Tom Payne

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with this program.  If not, see <http://www.gnu.org/licenses/>.
