---
pagetitle: High Availability Linode Pairs
longtitle: A recipe for creating automatic failover virtual server pairs at Linode.
tags: devops
author: Rob Martin
published: 2011-08-08 20:08
snippet: Something like a recipe for creating database and application servers that failover on each other. This recipe is for Lighttpd and MySQL, but it can be extended easily for other applications. Later, I'll do one for Tryton and PostgreSQL. I use Ubuntu 11.04 64-bit on two Linode virtual private servers.
---

Something like a recipe for creating database and application servers that failover on each other. This recipe is for Lighttpd and MySQL, but it can be extended easily for other applications. Later, I'll do one for Tryton and PostgreSQL. I use Ubuntu 11.04 64-bit on two Linode virtual private servers.

## Initial configuration, from within the Linode[^linode] dashboard

Repeat the following procedure for each member of the pair:

1. Go to the Remote Access tab on the Linode dashboard.

	* Note the public IP. We'll need that later.

	* Add a private IP, and note it. We'll need that later, too.

	* If by any chance you have an IPv6 private address, note that. We'll use it all. If you don't have one, click on "Enable IPv6" and get one. We really are going to use it.

	* Set a Lish password. I use APG[^APG] to generate new passwords, e.g. `apg -n 100`.

	* Add an SSH public key to the Keys box. This is in addition to the Lish password.  Or instead of it, if you prefer.

	* This is also good time to write to Linode support. You'll need to order an additional public IP address, and by default you are unable to purchase an additional IP address until you've requested and justified it specifically with support. You'll also need another *private* IP address, which doesn't cost anything but it takes a support person to do it - there's no way to automatically add more than one private IP address. These addresses will float between the pairs, so it doesn't matter which of the pair get these additional addresses. Make note of these floating addresses.

[^linode]: I didn't realize that Linode has a referral program, so I'm updating this footnote! If you sign up, kindly use this link: [Linode.com][].

[Linode.com]: http://www.linode.com/?r=3056f385845a7a1f538397b0daa697357e884da1 "A link to Linode that includes my referral code, which means I could get paid a little bit."

[^APG]: I built a [pronouncable password generator][ppg] too, in Perl, more than a decade ago. Got lots of reputations points on PerlMonks.org. But [APG][] is cool too.

[ppg]: http://perlmonks.org/index.pl?node_id=101324 "Password generator using a linguistic rule base, by me, long before I started using the version2beta handle."

[apg]: http://www.adel.nursat.kz/apg/ "Automatic Password Generator"

2. Go to the Settings tab in the Linode dashboard.

	* Give the Linode a good name.

	* Name a display group for your pair. Generally we use "location pair", e.g. "newark pair", "dallas pair", etc.

	* Change the email alert thresholds

		CPU Usage: 50% (Default 80% over two hours is too high)

		Disk I/O Rate: 2000 I/O Ops/sec (Default 1000 ops over two hours is too low)

3. Go to the Dashboard tab in the Linode dashboard, and click on "Create a new Disk Image".

	* Create a disk image called "var-www", type "unformatted / raw", with a size that's half your available space (just as a rule of thumb).

	* Create another disk image called "var-mysql", also type "unformatted / raw", with a size that's half your _remaining_ available space (again, a rule of thumb).

4. Also on the Linode dashboard tab, choose to deploy a Linux distribution.

	* I like the latest Ubuntu[^ubuntu]. Right now, that's 11.04 64bit.

	* Set the swap partition size next. Max it out, which on Linode seems to be 1 x RAM.

	* Use everything that's left for the deployment disk size.

	* Set a good root password.

	* Press the Deploy button. That takes you back to the dashboard.

5. The configuration profile name is too pedestrian. "My Ubuntu blah blah blah". I already knew it was mine, after all I made it, didn't I? Click the "Edit" link to the right of the profile.

	* Change the label to something with some teeth. Something like "Ubuntu 11.04-64bit high availability pair".

	* Assign block devices for our DR:BD drives. Put var-mysql on /dev/xvdc, and var-www on /dev/xvdd. Save the changes, and you'll be taken back to the dashboard.

6. Each server needs to be able to assume the IP address from the other. From the "Remote Access" tab for each server, select all of the addresses, both the public and private, for IP Failover.

7. Boot the system and log in, either through SSH using the root password you just assigned, or through Lish.

	This is a good time to repeat the previous six steps on the other member of the pair. Once that is done, continue to the final step in the Linode dashboard.

##Some notes before we get into it

We have a number of implementation specific details, stuff that changes for each cluster. I'd like to refer to things in here in a way that is internally consistent, yet easy to use. So here is an index of variable data:

Server names: `alice` and `gertrude`. If you're running `vi` or `vim` against this document, and wanted to name your systems Fred and Wilma, you should be able to run `:%s/alice/fred/g` followed by `:%s/gertrude/wilma/g`.

IP addresses:

* Public IP on alice: `pub.a.a.a`

* Private IP on alice: `priv.a.a.a`

* IPv6 on alice: `ipv6:pub:a`

* Public IP on gertrude: `pub.g.g.g`

* Private IP on gertrude: `priv.g.g.g`

* IPv6 on gertrude: `ipv6:pub:g`

* Floating public IP: `pub.f.f.f`

* Floating private IP: `priv.f.f.f`

##Initial configuration, operating system

Again, repeat the following steps for each member of the pair:

1. Name the server.

	`echo 'alice' > /etc/hostname`[^toklas]

2. Fix (as in "make permanent") the networking in `/etc/network/interfaces`:

		#vi /etc/network/interfaces

			# The loopback interface
			auto lo
			iface lo inet loopback

			# Configuration for eth0 and aliases

			# This line ensures that the interface will be brought up during boot.
			auto eth0 eth0:0

			# eth0 - This is the main IP address that will be used for most outbound connections.
			# The address, netmask and gateway are all necessary.
			iface eth0 inet static
			 address pub.a.a.a
			 netmask 255.255.255.0
			 gateway gate.a.a.a

			# eth0:0
			# This is a private IP
			iface eth0:0 inet static
			 address priv.a.a.a
			 netmask 255.255.128.0

			# eth0 ipv6
			iface eth0 inet6 static
			 address ipv6:pub:a
			 netmask 64
		 	 gateway fe80::1


3. Deal with some networking stuff that makes our life easier.

		#vi /etc/hosts

			127.0.0.1       localhost.localdomain		localhost
			pub.a.a.a   	alice.version2beta.com		# FQDN is public
			priv.a.a.a	alice				# short name is private
			pub.g.g.g	gertrude.version2beta.com	# FQDN is public
			priv.g.g.g	gertrude			# short name is private
			pub.f.f.f	ag.version2beta.com		# public floating IP
			priv.f.f.f	ag				# private floating IP

			#IPv6 addresses
			ipv6:a		ip6-alice
			ipv6:g		ip6-gertrude

			# The following lines are desirable for IPv6 capable hosts
			::1     ip6-localhost ip6-loopback
			fe00::0 ip6-localnet
			ff00::0 ip6-mcastprefix
			ff02::1 ip6-allnodes
			ff02::2 ip6-allrouters

	SSH Keys

		# ssh-keygen -t rsa

	Copy `/etc/.ssh/id_rsa.pub` into the other server's `/etc/.ssh/authorized_keys`, which you'll need to create. While you're at it, put your key in there too.


4. Restart networking. (If there's a mistake in the previous step, we'll catch it either here or in the next few steps.)

		# /etc/init.d/networking restart

5. Bring the package database, and installed packages, up to date.

		# apt-get update && apt-get upgrade

## Install and configure DR:BD

1. Install the tools

	First, try:

		# apt-get install drbd8-utils
		# drbdadm -V

	At the top of the options, it will give the version of the DRBD module and userland version. If these match, you're golden. If they don't (as is the case currently for me), you need to build userland tools from source.

		# drbdadm -V

		DRBD module version: 8.3.10
		   userland version: 8.3.9
		you should upgrade your drbd tools!

	Building drbd8-utils from source:

		# apt-get remove drbd8-utils
		# apt-get install psmisc build-essential flex git xsltproc
		# wget http://oss.linbit.com/drbd/8.3/drbd-8.3.10.tar.gz
		# tar xvzf drbd-8.3.10.tar.gz
		# cd drbd-8.3.10
		# ./configure
		# make
		# make install

	All good? Try the drbdadm -V again:

		# drbdadm -V

		...
		Version: 8.3.10 (api:88)
		GIT-hash: 5c0b0469666682443d4785d90a2c603378f9017b build by root@alice, 2011-08-06 22:35:11

	Configure DR:BD global settings in `/usr/local/etc/drbd.d/global_common.conf`:

		# vi /usr/local/etc/drbd.d/global_common.conf

			global {
				usage-count yes;
				# minor-count dialog-refresh disable-ip-verification
			}

			common {
				protocol C;

				handlers {
					pri-on-incon-degr "/usr/lib/drbd/notify-pri-on-incon-degr.sh; /usr/lib/drbd/notify-emergency-reboot.sh; echo b > /proc/sysrq-trigger ; reboot -f";
					pri-lost-after-sb "/usr/lib/drbd/notify-pri-lost-after-sb.sh; /usr/lib/drbd/notify-emergency-reboot.sh; echo b > /proc/sysrq-trigger ; reboot -f";
					local-io-error "/usr/lib/drbd/notify-io-error.sh; /usr/lib/drbd/notify-emergency-shutdown.sh; echo o > /proc/sysrq-trigger ; halt -f";
					# fence-peer "/usr/lib/drbd/crm-fence-peer.sh";
					split-brain "/usr/lib/drbd/notify-split-brain.sh root";
					# out-of-sync "/usr/lib/drbd/notify-out-of-sync.sh root";
					# before-resync-target "/usr/lib/drbd/snapshot-resync-target-lvm.sh -p 15 -- -c 16k";
					# after-resync-target /usr/lib/drbd/unsnapshot-resync-target-lvm.sh;
				}

				startup {
					# wfc-timeout degr-wfc-timeout outdated-wfc-timeout wait-after-sb
					wfc-timeout 15;
					degr-wfc-timeout 60;
				}

				disk {
					# on-io-error fencing use-bmbv no-disk-barrier no-disk-flushes
					# no-disk-drain no-md-flushes max-bio-bvecs
				}

				net {
					# sndbuf-size rcvbuf-size timeout connect-int ping-int ping-timeout max-buffers
					# max-epoch-size ko-count allow-two-primaries cram-hmac-alg shared-secret
					# after-sb-0pri after-sb-1pri after-sb-2pri data-integrity-alg no-tcp-cork
					cram-hmac-alg sha1;
				}

				syncer {
					# rate after al-extents use-rle cpu-mask verify-alg csums-alg
					rate 4M;
				}
			}

	Configure DR:BD resource settings for mysql in `/usr/local/etc/drbd.d/mysql.res`:

		# vi /usr/local/etc/drbd.d/mysql.res

			resource mysql {
				net {
					shared-secret "DONTTELL!";
				}
				on alice {
					device /dev/drbd0;
					disk /dev/xvdc;
					address ipv6 [ipv6:a]:7801;
					meta-disk internal;
				}
				on gertrude {
					device /dev/drbd0;
					disk /dev/xvdc;
					address ipv6 [ipv6:g]:7801;
					meta-disk internal;
				}
			}

	And create resource settings for www in `/usr/local/etc/drbd.d/www.res`:

		# vi /usr/local/etc/drbd.d/www.res

			resource www {
				net {
					shared-secret "DON'T TELL!";
				}
				on alice {
					device /dev/drbd1;
					disk /dev/xvdd;
					address ipv6 [ipv6:a]:7802;
					meta-disk internal;
				}
				on gertrude {
					device /dev/drbd1;
					disk /dev/xvdd;
					address ipv6 [ipv6:g]:7802;
					meta-disk internal;
				}
			}

	On this setup, drbd expects configuration files to be in /usr/local/etc/, and won't recognize them in /etc/. So we've put all our new stuff in the right place. Now get rid of the old. BUT, the cluster resource manager IS going to want them in /etc/, so we'll create symbolic links.

		# rm -rd /etc/drbd.*
		# ln -s /usr/local/etc/drbd.conf /etc/drbd.conf
		# ln -s /usr/local/etc/drbd.d /etc/drbd.d

2. Start up DR:BD

	Create metadata for the devices (only need to do this on one host):

		# drbdadm create-md mysql
		# drbdadm create-md www

	Modify the init script for drbd to accomodate a problem where IPv6 might not be ready when drbd starts:

		# vi /etc/init.d/drbd # Add a sleep to the top of the script

			### END INIT INFO
			sleep 5

	On each server, at roughly the same time, start drbd:

		# /etc/init.d/drbd start

	On the server that will be primarily a database server:

		# drbdadm -- --overwrite-data-of-peer primary mysql
		# drbdadm disconnect mysql
		# drbdadm connect mysql
		# drbdadm disconnect www
		# drbdadm connect www

	On the server that will be primarily a web application server:

		# drbdadm -- --overwrite-data-of-peer primary www
		# drbdadm disconnect www
		# drbdadm connect www
		# drbdadm disconnect mysql
		# drbdadm connect mysql

	On either or both servers, check to make sure the primaries are primary, the secondaries are secondary, and the sync is syncing:

		# cat /proc/drbd

		version: 8.3.10 (api:88/proto:86-96)
		built-in
		 0: cs:SyncSource ro:Primary/Secondary ds:UpToDate/Inconsistent C r-----
		    ns:3116484 nr:0 dw:0 dr:3593516 al:0 bm:219 lo:0 pe:2 ua:1 ap:0 ep:1 wo:f oos:1649980
			[============>.......] sync'ed: 65.5% (1608/4652)Mfinish: 0:05:08 speed: 5,328 (5,056) K/sec
		 1: cs:SyncTarget ro:Secondary/Primary ds:Inconsistent/UpToDate C r-----
		    ns:0 nr:2976256 dw:2976256 dr:0 al:0 bm:181 lo:0 pe:0 ua:0 ap:0 ep:1 wo:f oos:7509148
			[====>...............] sync'ed: 28.4% (7332/10236)Mfinish: 0:24:20 speed: 5,136 (5,060) want: 4,096 K/sec

	Format the DR:BD devices, each on the server that is primary for that device. On the database server:

		# apt-get install jfsutils
		# mkfs.jfs /dev/drbd0

	And on the web application server:

		# apt-get install jfsutils
		# mkfs.ext4 /dev/drbd1

	On both servers, create the mount points:

		# mkdir /var/www
		# mkdir /var/mysql

		# vi /etc/fstab // Add two lines

			/dev/drbd/by-res/www    /var/www        ext4    noauto,noatime
			/dev/drbd/by-res/mysql  /var/mysql      jfs     noauto

	Mount the drives, each on it's primary server:

		# mount /dev/drbd/by-res/mysql /var/mysql/
		# mount /dev/drbd/by-res/www /var/www

	And there you have it - RAID 1 drives over a network. Failover comes later, after we install and configure the server software.

## Services: MySQL and Lighttpd, plus a few other little things I like to have

1. Install stuff. Not too much stuff. Just the right stuff.

		# apt-get install bc whois apg s3cmd lsof traceroute exim4 mailutils mutt \
			aspell-doc wamerican spellutils ispell mysql-server mysql-client lighttpd \
			lighttpd-doc lighttpd-mod-magnet lua5.1 php5-cli php5-cgi php5 php5-mcrypt \
			php5-mhash php5-gd php5-mysql php5-imagick php5-curl php5-memcache php5-ps \
			php5-pspell php5-tidy php5-xmlrpc php5-xsl libgd-tools libmagickcore3-extra \
			libmcrypt-dev mcrypt rrdtool

		# dpkg-reconfigure exim4-config

2. Configure MySQL.

	Edit `/etc/mysql/my.cnf`:

		# vi /etc/mysql/my.cnf

			[client]
			port            = 3306
			socket          = /var/run/mysqld/mysqld.sock

			[mysqld_safe]
			socket          = /var/run/mysqld/mysqld.sock
			nice            = 0

			[mysqld]
			user            = mysql
			socket          = /var/run/mysqld/mysqld.sock
			port            = 3306
			basedir         = /usr
			datadir         = /var/mysql
			tmpdir          = /tmp
			skip-external-locking
			bind-address            = priv.f.f.f
			key_buffer              = 16M
			max_allowed_packet      = 16M
			thread_stack            = 192K
			thread_cache_size       = 8
			myisam-recover         = BACKUP
			#max_connections        = 100
			#table_cache            = 64
			#thread_concurrency     = 10
			query_cache_limit       = 1M
			query_cache_size        = 16M
			#general_log_file        = /var/log/mysql/mysql.log
			#general_log             = 1
			log_error                = /var/log/mysql/error.log
			#server-id              = 1
			#log_bin                        = /var/log/mysql/mysql-bin.log
			expire_logs_days        = 10
			max_binlog_size         = 100M
			#binlog_do_db           = include_database_name
			#binlog_ignore_db       = include_database_name
			# ssl-ca=/etc/mysql/cacert.pem
			# ssl-cert=/etc/mysql/server-cert.pem
			# ssl-key=/etc/mysql/server-key.pem

			[mysqldump]
			quick
			quote-names
			max_allowed_packet      = 16M

			[mysql]
			#no-auto-rehash # faster start of mysql but no tab completition

			[isamchk]
			key_buffer              = 16M

			!includedir /etc/mysql/conf.d/

	We changed the default directory for MySQL, so we need to move it's files into the correct directory.

		# stop mysql
		# chown mysql:mysql /var/mysql
		# cp -Rp /var/lib/mysql/* /var/mysql/
		# ifconfig eth0:1 priv.f.f.f netmask 255.255.128.0
		# start mysql

	Sync these settings with the other server (both need the same configuration so that either can run the service off the same data).

		# scp -r /etc/mysql root@gertrude:/etc/

3. Configure Lighttpd. (Substitute other instructions for nginx, apache, etc.)

	Edit `/etc/lighttpd/lighttpd.conf` on the web application server:

		# vi /etc/lighttpd/lighttpd.conf

			server.modules = (
				"mod_access",
				"mod_alias",
				"mod_compress",
			 	"mod_redirect",
			        "mod_rewrite",
			)

			server.document-root        = "/var/www"
			server.upload-dirs          = ( "/var/www/cache/uploads" )
			server.errorlog             = "/var/www/logs/error.log"
			server.pid-file             = "/var/run/lighttpd.pid"
			server.username             = "www-data"
			server.groupname            = "www-data"

			index-file.names            = ( "index.php", "index.html")

			url.access-deny             = ( "~", ".inc" )

			static-file.exclude-extensions = ( ".php", ".pl", ".fcgi" )

			## Use ipv6 if available
			include_shell "/usr/share/lighttpd/use-ipv6.pl"

			dir-listing.encoding        = "utf-8"
			server.dir-listing          = "enable"

			compress.cache-dir          = "/var/www/cache/compress/"
			compress.filetype           = ( "application/x-javascript", "text/css", "text/html", "text/plain" )

			include_shell "/usr/share/lighttpd/create-mime.assign.pl"
			include_shell "/usr/share/lighttpd/include-conf-enabled.pl"

			magnet.attract-physical-path-to = ( "/etc/lighttpd/modx.lua" )

	Enable some more modules:

		# lighttpd-enable-mod fastcgi
		# lighttpd-enable-mod simple-vhost
		# lighttpd-enable-mod accesslog
		# lighttpd-enable-mod magnet
		# lighttpd-enable-mod status

	Edit more config files:

		# vi /etc/lighttpd/conf-enabled/10-accesslog.conf

			server.modules += ( "mod_accesslog" )
			accesslog.filename = "/var/www/logs/access.log"

		# vi /etc/lighttpd/conf-enabled/10-simple-vhost.conf

			server.modules += ( "mod_simple_vhost" )
			simple-vhost.server-root         = "/var/www/servers/"
			simple-vhost.document-root       = "htdocs"
			simple-vhost.default-host        = "{hostname}"

		# vi /etc/lighttpd/conf-enabled/10-fastcgi.conf

			server.modules += ( "mod_fastcgi" )
			fastcgi.server    = ( ".php" =>
			        ((
			                "bin-path" => "/usr/bin/php-cgi",
			                "socket" => "/tmp/php.socket",
			                "max-procs" => 2,
			                "idle-timeout" => 20,
			                "bin-environment" => (
			                        "PHP_FCGI_CHILDREN" => "4",
			                        "PHP_FCGI_MAX_REQUESTS" => "10000"
			                ),
			                "bin-copy-environment" => (
			                        "PATH", "SHELL", "USER"
			                ),
			                "broken-scriptfilename" => "enable"
			        ))
			)

		# vi /etc/lighttpd/modx.lua

			attr = lighty.stat(lighty.env["physical.doc-root"] .. "manager/includes/config.inc.php") -- Appears to be a ModX site
			if (attr) then
			  attr = lighty.stat(lighty.env["physical.path"])
			  if (not attr) then -- Requested resource doesn't exist in the file system
			    path = "/index.php"
			    uri = lighty.env["request.uri"]
			    uri2 = string.gsub(lighty.env["request.uri"], "\?", "\&")
			    -- print("Original request.uri: " .. uri .. " Replaced with: " .. uri2)
			    lighty.env["uri.query"] = "q=" .. string.gsub(uri, "\?", "\&")
			    lighty.env["uri.path"] = path
			    lighty.env["request.uri"] = path .. "?" .. lighty.env["uri.query"]
			    -- print("New request.uri: " .. lighty.env["request.uri"])
			    lighty.env["physical.rel-path"] = path
			    lighty.env["physical.path"] = lighty.env["physical.doc-root"] .. string.sub(lighty.env["physical.rel-path"], 2)
			    -- print("New physical.path: " .. lighty.env["physical.path"])
			  end
			end

		# vi /etc/lighttpd/wp-rewrite.conf

			# Use when a site has a blog
			# Example:
			# $HTTP["host"] =~ "www\.example\.com" {
			#   var.wpdir = "/blog/"
			#   include "wp-rewrite.conf"
			# }

			url.rewrite-once = (
			  "^" + wpdir + "(wp-.+).*/?" => "$0",
			  "^" + wpdir + "(sitemap.xml)" => "$0",
			  "^" + wpdir + "(xmlrpc.php)" => "$0",
			  "^" + wpdir + "keyword/([A-Za-z_0-9-])/?$" => wpdir + "index.php?keyword=$1",
			  "^" + wpdir + "(.+)/?$" => wpdir + "index.php/$1"
			)

	Edit `/etc/php5/cgi/php.ini` so that these lines are correct:

		display_errors = stderr
		error_log = /var/www/logs/php_errors.log

	Create some directories we just specified, but don't yet have:

		# mkdir /var/www/run /var/www/cache /var/www/cache/compress /var/www/cache/uploads /var/www/logs /var/www/servers /var/www/servers/{hostname} /var/www/servers/{hostname}/htdocs
		# chown -R www-data:www-data /var/www/run /var/www/cache /var/www/logs/ /var/www/servers

	Restart the local server and sync these settings with the other server (both need the same configuration so that either can run the service off the same data).

		# /etc/init.d/lighttpd restart
		# scp -r /etc/php5 /etc/lighttpd root@alice:/etc/

4. Test that either server can run either service.

	On the database server:

		# stop mysql
		# ifconfig eth0:1 down
		# umount /var/mysql/
		# drbdadm secondary mysql

	On the web application server:

		# drbdadm primary mysql
		# mount /dev/drbd0 /var/mysql/
		# ifconfig eth0:1 priv.f.f.f netmask 255.255.128.0
		# start mysql
		# mysql -p
		# stop mysql
		# ifconfig eth0:1 down
		# umount /var/mysql/
		# drbdadm secondary mysql
		# /etc/init.d/lighttpd stop
		# umount /var/www
		# drbdadm secondary www

	On the database server:

		# drbdadm primary mysql
		# drbdadm primary www
		# mount /dev/drbd0 /var/mysql/
		# mount /dev/drbd1 /var/www/
		# /etc/init.d/lighttpd start
		# ps ax | grep light
		# /etc/init.d/lighttpd stop
		# umount /var/www
		# drbdadm secondary www
		# ifconfig eth0:1 priv.f.f.f 255.255.128.0
		# start mysql

	On the web application server:

		# drbdadm primary www
		# mount /dev/drbd1 /var/www
		# /etc/init.d/lighttpd start

## Configure a failover cluster

Just a note here before we start. I would have liked to run corosync[^corosync] rather than heartbeat[^heartbeat] based solely on my understanding of pacemaker[^pacemaker], how and why it split from heartbeat. However, corosync seems to mostly require multicast networking to work[^mcast] and after some struggles, I've learned that Linode doesn't support multicast.

1. Set up heartbeat and pacemaker

	Install and configure software on both servers

		# apt-get install heartbeat pacemaker

	Edit `/usr/lib/ocf/resource.d/heartbeat/Filesystem` to comment out this whole block. This will fix a problem with the script's handling of jfs filesystems. Besides, they say right in the code, "Why should a filesystem resource agent magically load a kernel module?" I agree. Lemme handle that part and just mount the drive please.

		# vi /usr/lib/ocf/resource.d/heartbeat/Filesystem

			#       if [ "X${HOSTOS}" != "XOpenBSD" ];then
			#               # Insert SCSI module
			#               # TODO: This probably should go away. Why should the filesystem
			#               # RA magically load a kernel module?
			#               $MODPROBE scsi_hostadapter >/dev/null
			#
			#               if [ -z "$FSTYPE" -o "$FSTYPE" = none ]; then
			#                       : No FSTYPE specified, rely on the system has the right file-system support already
			#               else
			#                       # Insert Filesystem module
			#                       $MODPROBE $FSTYPE >/dev/null
			#                       grep -e "$FSTYPE"'$' /proc/filesystems >/dev/null
			#                       if [ $? -ne 0 ] ; then
			#                               ocf_log err "Couldn't find filesystem $FSTYPE in /proc/filesystems"
			#                               return $OCF_ERR_INSTALLED
			#                       fi
			#               fi
			#       fi


	Configuration files:

		# vi /etc/ha.d/authkeys

			auth 1
			1 sha1 Don'tTell

		# chmod 600 /etc/heartbeat/authkeys

		# vi /etc/ha.d/ha.cf

			autojoin none
			logfacility daemon
			keepalive 2
			deadtime 15
			warntime 5
			initdead 120
			udpport 694
			ucast eth0 priv.a.a.a
			ucast eth0 priv.g.g.g
			node alice
			node gertrude
			auto_failback on
			use_logd yes
			crm respawn

	Propogate the configuration and restart each server

		# scp -r /etc/ha.d/* root@gertrude:/etc/ha.d/

		# /etc/init.d/heartbeat restart

	Disable automatic start of Lighttpd (using LSB) and MySQL (using Upstart)

		# update-rc.d lighttpd disable
		# vi /etc/init/mysql.conf

			# Comment out the startup
			#start on (net-device-up
			#          and local-filesystems
			#         and runlevel [2345])


	Configure the resources

		# vi /usr/lib/ocf/resource.d/linbit/drbd

			OCF_RESKEY_drbdconf_default="/usr/local/etc/drbd.conf"

		# crm configure

			primitive db_alert ocf:heartbeat:MailTo \
				params email="root" subject="(db)"
			primitive db_drbd ocf:linbit:drbd \
				params drbd_resource="mysql" \
				op start interval="0" timeout="240" \
				op stop interval="0" timeout="100"
			primitive db_fs ocf:heartbeat:Filesystem \
				params device="/dev/drbd0" directory="/var/mysql" fstype="jfs" \
				op start interval="0" timeout="60" \
				op stop interval="0" timeout="120"
			primitive db_ip ocf:heartbeat:IPaddr \
				params ip="priv.f.f.f" cidr_netmask="24"
			primitive db_mysql lsb:mysql \
				op monitor interval="0" enabled="false"

			primitive www_alert ocf:heartbeat:MailTo \
				params email="root" subject="(www)"
			primitive www_drbd ocf:linbit:drbd \
				params drbd_resource="www" \
				op start interval="0" timeout="240" \
				op stop interval="0" timeout="100"
			primitive www_fs ocf:heartbeat:Filesystem \
				params device="/dev/drbd1" directory="/var/www" fstype="ext4" \
				op start interval="0" timeout="60" \
				op stop interval="0" timeout="120"
			primitive www_ip ocf:heartbeat:IPaddr \
				params ip="pub.f.f.f" cidr_netmask="24"
			primitive www_lighty lsb:lighttpd \
				op monitor interval="0" enabled="false"

			group db db_ip db_fs db_mysql db_alert \
				meta target-role="Started"

			group www www_fs www_ip www_lighty www_alert \
				meta target-role="Started"

			ms ms_db_drbd db_drbd \
				meta master-max="1" master-node-max="1" clone-max="2" clone-node-max="1" notify="true"

			ms ms_www_drbd www_drbd \
				meta master-max="1" master-node-max="1" clone-max="2" clone-node-max="1" notify="true"

			location db_gertrude db 10: gertrude
			location db_alice db 100: alice
			location www_gertrude www 100: gertrude
			location www_alice www 10: alice

			colocation db_on_drbd inf: db ms_db_drbd:Master
			colocation www_on_drbd inf: www ms_www_drbd:Master

			order db_after_drbd inf: ms_db_drbd:promote db:start
			order www_after_drbd inf: ms_www_drbd:promote www:start

			property cluster-infrastructure="Heartbeat" \
				stonith-enabled="false" \
				no-quorum-policy="ignore" \
				default-resource-stickiness="5"



[^ubuntu]: I used to use [Debian][], and got the install down to 1.3GB comfortably. But since I started running Ubuntu on my desktop I've simplified my life and got [Ubuntu Server][]

[Debian]: http://www.debian.org "You know, Debian. The standard Linux distro."

[Ubuntu Server]: http://www.ubuntu.com/business/server/overview "Ubuntu is based on Debian, so it's got that going for it."

[^toklas]: We name our servers after couples who inspired each other in their art and life. The example shown refers to Alice B. Toklas, partner and lover of Gertrude Stein for nearly 40 years, author of The Alice B. Toklas Cookbook which contains among other recipes what we often refer to as "magic brownies" (She called it "Haschich Fudge".) Gertrude was pretty cool too.

[^corosync]: This cluster management stuff gets complicated. I have to really wrap my head around it when I'm making it work, and I still only have 80% confidence I understand what it's doing, even if I have 100% confidence what I've done works. And this is a how-to, not a why-for, so I don't want to bog it down a lot. So I'll just send you to [the Wikipedia page for the Corosync project](http://en.wikipedia.org/wiki/Corosync_(project) "Corosync on Wikipedia gives a decent overview, not too technical").

[^heartbeat]: Keeping in line with the last footnote, here's the [Wikipedia page for Linux-ha (high availability Linux)](http://en.wikipedia.org/wiki/Linux-HA "High Availability Linux on Wikipedia"), the project that puts out heartbeat.

[^pacemaker]:
	Okay, so you get some theory anyway. Heartbeat is how the system knows which members of a cluster are online. Pacemaker is how it determines which resources belong with cluster node. Got it? Pacemaker strikes me as the bad boy of Linux high availability. It got spun off from the Linux-ha project back in '07 in a brouhaha that involved one guy leaving the project and not replying to emails, the head of kernel R&D at SuSE talking street, really just a big clusterf**k. Well, I may be overstating it, but the first time I configured one of these pairs, I didn't have to deal with the politics of project development to try to figure out where all the pieces came from and how they went together.

	Oh, Pacemaker doesn't have a Wikipedia page, but there's a [Wiki page at clusterlabs.org](http://clusterlabs.org/wiki/Pacemaker "Wiki page on Pacemaker at ClusterLabs.org").

[^mcast]: [IP Multicasting](http://en.wikipedia.org/wiki/IP_multicast "IP Multicasting article at Wikipedia") is a way for one computer to talk to more than one computer at a time, in one transmission. It's kinda like tuning into a TV channel - you pick the channel you want to listen to, and then there's all this stuff happening. Except not at Linode.
