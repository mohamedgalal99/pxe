PXE worked
ALternate boot loaders

	- ISOLinux  CD/DVD
	- PXELinux  Network
	- SYSlinux 	USB
	- EXRLinux	Boot from hard drive

	-All come from one package
		config files are very similar
		    isolinux.cfg  syslinux.cfg  extlinux.conf  pxelinux.cfg

	-Config dnsmsq
		- dnsmasq provide => caching dns serv, dhcp ser, tftp serv, pxe serv

		-Install dnsmasq and test dns cache

			yum install dnsmasq bind-utils
			systemctl start dnsmasq
			systemctl enable dnsmasq
			systemctl status dnsmasq
			dig google.com @localhost

		-Config dnsmasq as dhcp server

			cd /etc/
			vim dnsmasq.conf
				interface=eth0
				domain=example.com
				dhcp-range=eth0,192.168.1.3,192.168.1.253,255.255.255.0,1h
				dhcp-option=3,192.168.1.1	//default route
				dhcp-option=6,192.168.1.2	//dns
				dhcp-option=28,192.168.1.255	//broadcast
				
				pxe-prompt="Press F8 for menu.",60	//wait 60 sec
				pxe-service=x86PC, "Boot to Local HardDisk"	//boot from local disk
				pxe-service=x86PC, "Install Centos7", pxelinux	//select install Centos, and search about file pxelinux

				enable-tftp
				tftp-root=/var/lib/tftpboot 	//enable connection to tftp

			mkdir /var/lib/tftpboot
			systemctl restart dnsmasq
			systemctl status dnsmasq

		-configure PXELinux
			cd /var/lib/tftpboot
			yum install syslinux
			cp /usr/share/syslinux/pxelinux.0 .
			cp /usr/share/syslinux/menu.c32 .
			mkdir pxelinux.cfg
			cd !$
			vim default
				dafult menu.c32
				prompt 0
				menu title Install R Us
				lable centos7
				menu lable Install CentOs7
					kernel centos7/vmlinuz
					append initrd=centos7/initrd.img 
					method=ftp://192.168.1.2/pub/centos7

			mkdir /var/lib/tftproot/centos7

		-Configure vsftpd
			yum install vsftpd
			cd /etc
			mv vsftpd.conf vsftpd.old
			vim vsftpd.conf
				listen=YES
				local_enable=No
				anonymous_enable=YES
				write_enable=NO
				anon_root=/var/ftp
			mkdir -m 755 /var/ftp/pub/centos7
			rsync -arv /mnt/ /var/ftp/pub/centos7/
			cd /var/lib/tftpboot/centos7
			cp /var/ftp/pub/centos7/isolinux/vmlinuz .
			cp /var/ftp/pub/centos7/isolinux/initrd.img .
			cd ..
			restorcon -Rv .
			systemctl start vsftpd
			systemctl enable vsftpd
			systemctl status vsftpd
			
