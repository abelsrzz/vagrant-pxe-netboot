<div align="center" id="cabecera">
        <h1>PXE Vagrant Netboot</h1>
        <a href="https://ibb.co/PNDRhVR"><img src="https://i.ibb.co/W6VQyRQ/Vagrant.png" alt="Vagrant_logo"       border="0" width="15%"></a>
</div>

<div id="inicio">
        <p>A Vagrantfile that provides an enviroment for PXE boot, to solve problems with OS deployment with tools like clonezilla. </p>
</div>


<div id="necesitamos">
<h3>What we will need</h3>
<p>If you don't have the needed tools, get them with the next comands</p>
        
  ```
$ sudo apt update && sudo apt upgrade && sudo apt install vagrant && sudo apt install virtualbox
  ```

<p>If you have issues installing Vagrant and Virtualbox at the same time, you will need to execute the following lines</p>
        
  ```
$ sudo apt update && sudo apt upgrade && sudo apt install vagrant
$ wget -O- https://www.virtualbox.org/download/oracle_vbox_2016.asc 
$ sudo gpg --dearmor --yes --output /usr/share/keyrings/oracle-virtualbox-2016.gpg
```

<div id="explicacion">
<h3>Creating Vagrantfile</h3>
        
<p>We will need to create a folder where we will store our <b>Vagrantfile</b> and everything we will want to sync with the virtual machine.</p>

```
$ mkdir PXEBOOTvagrant && cd PXEBOOTvagrant
```
        
<p>Then we will need to create the <b>Vagrantfile</b>, you can choose any text editor, in my case I will be using <code>nano</code></p>

```
$ nano Vagrantfile
```
        
<p>After creating our <b>Vagrantfile</b>, we must paste the following code <b>(Lines that start with "#" are comments, so they aren't needed)</b></p>

<div id="Vagrantfile">
<h3>Define the Vagrantfile</h3>
        
```
#We set the number of client machines that we want
num_cli = 4

Vagrant.configure("2") do |config|
        
#We set the Server machine with the box that we want, I will be using debian/buster64 but you can choose any linux box in https://app.vagrantup.com/boxes/search

  config.vm.define "server" do |subconfig|
       subconfig.vm.box = "debian/buster64"
        subconfig.vm.hostname = "server"
        
#We set the network in our machine, we will set a private network with the IP adress that we prefer
        subconfig.vm.network :private_network, ip: "192.168.100.5",

#We set a second adapter with a intnet of virtualbox
        virtualbox__intnet: "lan1"

        
#Optional machine configurations
	subconfig.vm.provider :virtualbox do |vb|
        
#We set an especific name for our Server machine
		vb.name = "server"
		vb.gui = false
	end


#Then we set up the server that will be holding the netboot image

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
        
        
#The following line will define the OS that our clients will have, I will use kali, but you can set other OS with the links in OsNetbootLinks.txt on the top of the page
	wget http://http.kali.org/kali/dists/kali-rolling/main/installer-amd64/current/images/netboot/netboot.tar.gz
	tar -zxf netboot.tar.gz && rm netboot.tar.gz
SHELL


        
#Once we have our server up, we set up our clients
        
end

#We set a loop with the number of clients that we defined at the top of the file
(1..num_cli).each do |x|  
        
	config.vm.define "client#{x}", autostart: false do |cli|

#We define a empty box prepared for pxe
              cli.vm.box = "TimGesekus/pxe-boot"
        cli.vm.hostname = "client#{x}"
        
#We set up the network of our VM, we need a private dhcp network with a lan and we will set in the first adapter of our machine
        cli.vm.network "private_network", type: "dhcp",
	:adapter => 1, virtualbox__intnet: "lan1"
	
        
#Lastly we will set the name of our client machine and we auto show it when we do vagrant up
	cli.vm.provider :virtualbox do |vb|
		vb.name = "client#{x}"
		vb.gui = true
	end 
    end
  end
end
```
</div>
</div>



<div id="vagrantup">
<h3>Iniciating the machines</h3>
        <p>After setting up our <b>Vagrantfile</b> we vagrant up our Server</p>
        
```
$ vagrant up
```
<p>The Server will delay several time to init because it has to download and transfer the needed files</p>
<p>When it finishes, we can vagrant up our clients.</p>

```
$ vagrant up client1
```
	
<p>In this case I ounly launched <code>client1</code>, but you can also launch <code>client2,client3...</code>. <b>(The number of clients that we can launch is set up on top of the vagrant file "num_cli = x ")</b>
<p>When the machine sets up, it will pxe boot with an installer of the OS that we set up in the provision of the Server in our <b>Vagrantfile</b>
If the OS that you need inst in our file, you can easily search "exampleOS" netboot and get the link that you need
</div>
