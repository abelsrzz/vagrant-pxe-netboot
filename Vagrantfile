num_cli = 4

Vagrant.configure("2") do |config|
  config.vm.define "servidor" do |subconfig|
       subconfig.vm.box = "debian/buster64"
        subconfig.vm.hostname = "servidor"
        subconfig.vm.network :private_network, ip: "192.168.100.5",
        virtualbox__intnet: "lan1"

	subconfig.vm.provider :virtualbox do |vb|
		vb.name = "servidor"
		vb.gui = false
	end


  subconfig.vm.provision "shell", inline: <<-SHELL
	sudo su -
	apt update -y && apt upgrade -y && apt install -y dnsmasq
	echo "dhcp-range=192.168.100.50,192.168.100.150,255.255.255.0,12h
dhcp-boot=pxelinux.0
enable-tftp
tftp-root=/srv/tftp" > /etc/dnsmasq.conf
	mkdir /srv/tftp
	systemctl restart dnsmasq
	cd /srv/tftp
	wget http://http.kali.org/kali/dists/kali-rolling/main/installer-amd64/current/images/netboot/netboot.tar.gz
	tar -zxf netboot.tar.gz && rm netboot.tar.gz
SHELL


end
(1..num_cli).each do |x|  
	config.vm.define "cliente#{x}", autostart: false do |cli|
              cli.vm.box = "TimGesekus/pxe-boot"
        cli.vm.hostname = "cliente#{x}"
        cli.vm.network "private_network", type: "dhcp",
	:adapter => 1, virtualbox__intnet: "lan1"
	
	cli.vm.provider :virtualbox do |vb|
		vb.name = "cliente#{x}"
		vb.gui = true
	end 
    end
  end
end
