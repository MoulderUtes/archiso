**CARLI-Custom-Arch-Linux-ISO**

Most this information is from /usr/share/docs/archiso but I'm rewriting it here for learning reasons

**Setup:**

Get packages:

		'archiso' libraries,mkarchiso
		'arch-install-scripts' arch-chroot,genfstab,pacstrap
		'mkinitcpio'
		'mkinitcpico-archiso' not sure what the diffrence is
Libraries:

	Get Profile
		'cp /usr/share/archiso/config/releng /(your directory)' #releng is the recomended monthly install iso baseline is the bare packages to boot
		 profile structure is under /usr/share/doc/archiso/README.profile.rst
**Configure:**

Packages:

	edit packages.x86_64
		Needed:
			linux
			linux-firmware
			base
			video drivers if window manager
		AUR or Custom Packages:
			edit pacman.conf and add custom repo
				[customrepo]
				SigLevel = Optional TrustAll
				Server = file:///path/to/customrepo
		Multilib:
			uncomment multilib repositry in pacman.conf
Files:

	airootfs becomes root directory default permissions are 644 and 755 (owned by root) 
	add files to airrootfs
			iptables
			networking
			config
			etc
ISO Config:

	edit profiledef.sh 
		<iso_name>-<iso_version>-<arch>.iso (e.g. archlinux-202010-x86_64.iso)
		iso_name
		iso_label
		iso_publisher : publisher of the iso
		iso_application : use case for iso
		iso_version
		install_dir : name of directory of the image when unpacked (8 characters long a-z 0-9)
		bootmodes
			bios.syslinux.mbr : syslinux for x86 bios disk boot (requires syslinux directory)
			bios.sylinux.eltorito : syslinux x86 bios optical disk (requires syslinux directory)
			uefi-x64.systemd-boot.esp : Sytemd-boot x86_64 UEFI disk boot (requires efiboot directory)
			uefi-x64.systemd-boot.eltorito : Systemd-boot x86_64 UEFI optical disk (requires efiboot directory)
			bios el torito must be before uefi el toritio boot mode
		arch : what architecture
		pacman_conf : file defaults to /etc/pacman.conf
		airootfs_image_type
			squashfs : squashfs immage form airootfs
			ext4+squashfs : ext4 partition and coppy airootfs and make squashfs image
		airootfs_image_tool_options : optins to pass to the airootfs tool
Systemd:

	enable services/sockets/times for live make the sys links just like sytemctl enable
		mkdir -p (workindir)/airootfs/etc/systemd/system/(what every multi-user or what)
		ln -s /usr/lib/systemd/system/(whatever) (workingdir)/airootfs/etc/systemd/system/(whatever.wants)/
		for x at boot ln -s /usr/lib/systemd/system/lxdm.service archlive/airootfs/etc/systemd/system/display-manager.service
Autologin:

	edit airootfs/etc/systemd/system/getty@tty1.service.d/autologin.conf
		[Service]
		ExecStart=
		ExecStart=-/sbin/agetty --autologin username --noclear %I 38400 linux
User:

	edit /airrootfs/etc /passwd /shadow /group /gshadow
		/passwd
			(user):x:1000:1000::/home/archie:/usr/bin/(shell)
		/shadow
			gen hash with opessl passwd -6 
			add it to /shadow
			(user):$6$archiesalt$1yystReWRMUYWmt7fTR/BjcRWrmF//984HxCL6QxCMeDes0pEBRG3v1Jyqp1I1/x46kmU7KyjDfTXikqtq3YY.:14871::::::
		/group
			add users group
			(user):x:1000:
		/gshadow
			(user):!*::
Install via SSH:

	Enable sshd.service: 
		mkdir -p archlive/airootfs/etc/systemd/system/multi-user.target.wants
		ln -s /usr/lib/systemd/system/sshd.service archlive/airootfs/etc/systemd/system/multi-user.target.wants/
	Make .ssh in home dir:
		mkdir /airootfs/(root or home/user)/.ssh
	Add pub key (optional?):
		cat ~/.ssh/id_ed25519.pub >> archlive/airootfs/root/.ssh/authorized_keys
	Set permisons for .ssh and authorized_keys file:
		edit /profile.sh
			file_permissions=(
  			...
  			["/root"]="0:0:0750"
  			["/root/.ssh"]="0:0:0700"
  			["/root/.ssh/authorized_keys"]="0:0:0600"
			)
Auto Connect WIFI:

	mkdir -p /airootfs/var/lib/iwd
	add stuff to it
	edit permisions in /profiledef.sh
		...
		file_permissions=(
 		...
  		["/var/lib/iwd"]="0:0:0700"
		)
Gimp:

	for splash change syslinux/splash.png
	for general edit /airoot/etc/default/grub
			add GRUB_THEME="/usr/share/grub/themes/(theme)/theme.txt
**Build ISO:**
	mkarchiso -v -w /(work_dir) /o /(out_dir) /(profile)