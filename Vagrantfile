# -*- mode: ruby -*-
# vim: set ft=ruby :
home = ENV['HOME'] #home dir
MACHINES = {
  :otuslinux => {
       # :box_name => "centos-7-5",
       # :box_name => "tsvetkovivan1993/centos-7-5_not_shared_folders",
       #:box_name => "centos-7_new",
        :box_name => "centos/7",
        :ip_addr => '192.168.100.3',
	:disks => {
		:sata1 => {
			:dfile =>  home + '/VirtualBox VMs/sata_1.vdi',
			:size => 250,
			:port => 1
		},
		:sata2 => {
			:dfile =>  home + '/VirtualBox VMs/sata_2.vdi',
                        :size => 250, # Megabytes
			:port => 2
		},
                :sata3 => {
			:dfile =>  home + '/VirtualBox VMs/sata_3.vdi',
                        :size => 250,
                        :port => 3
                },
                :sata4 => {
			:dfile =>  home + '/VirtualBox VMs/sata_4.vdi',
                        :size => 250, # Megabytes
                        :port => 4
                },
                :sata5 => {
			:dfile =>  home + '/VirtualBox VMs/sata_5.vdi',
                        :size => 250, # Megabytes
                        :port => 5
	       },
                :sata6 => {
			:dfile =>  home + '/VirtualBox VMs/sata_6.vdi',
                        :size => 250, # Megabytes
                        :port => 6
                },
                :sata7 => {
                        :dfile =>  home + '/VirtualBox VMs/sata_7.vdi',
                        :size => 250, # Megabytes
                        :port => 7
               },
                :sata8 => {
                        :dfile =>  home + '/VirtualBox VMs/sata_8.vdi',
                        :size => 250, # Megabytes
                        :port => 8
               },
                :sata9 => {
                        :dfile =>  home + '/VirtualBox VMs/sata_9.vdi',
                        :size => 250, # Megabytes
                        :port => 9
               },
                :sata10 => {
                        :dfile =>  home + '/VirtualBox VMs/sata_10.vdi',
                        :size => 250, # Megabytes
                        :port => 10
               }



	}

		
  },
}


Vagrant.configure("2") do |config|
#if Vagrant.has_plugin?("vagrant-vbguest")
#    config.vbguest.auto_update = false
#end
  MACHINES.each do |boxname, boxconfig|

      config.vm.define boxname do |box|

          box.vm.box = boxconfig[:box_name]
          box.vm.host_name = boxname.to_s

          #box.vm.network "forwarded_port", guest: 3260, host: 3260+offset

          box.vm.network "private_network", ip: boxconfig[:ip_addr]

          box.vm.provider :virtualbox do |vb|
            	  vb.customize ["modifyvm", :id, "--memory", "1024"]
                  needsController = false
		  boxconfig[:disks].each do |dname, dconf|
			  unless File.exist?(dconf[:dfile])
				vb.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
                                needsController =  true
                          end

		  end
                  if needsController == true
                     vb.customize ["storagectl", :id, "--name", "SATA", "--add", "sata" ]
                     boxconfig[:disks].each do |dname, dconf|
                         vb.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
                     end
                  end
          end

 	  box.vm.provision "shell", inline: <<-SHELL
	      mkdir -p ~root/.ssh
              cp ~vagrant/.ssh/auth* ~root/.ssh
	      echo "SYSTEM IS UPDATING"
	      echo "-----------------------------------------------"
	      sudo yum update > /dev/null 2>&1
	      sudo  yum install -y mdadm smartmontools hdparm gdisk vim  net-tools > /dev/null 2>&1
              echo "-------------FINISH-------------"
  	  SHELL

	  #---------create_raid5---------------
          box.vm.provision "shell", inline: <<-RAID_MOUNT
              #create raid10 and mounting
              sudo mdadm --zero-superblock --force /dev/sd{b,c,d,e,f,g,h,i,j,k} > /dev/null 2>&1
              sudo mdadm --create --verbose /dev/md0 -l 5 -n 10 /dev/sd{b,c,d,e,f,g,h,i,j,k} > /dev/null 2>&1
              sudo mkdir  /etc/mdadm
              sudo mkdir  /raid
              sudo echo "DEVICE partitions" > /etc/mdadm/mdadm.conf
              sudo mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm/mdadm.conf
	      echo "===========RAID CREATED=========="
              #create volume
              sudo parted -s /dev/md0 mklabel gpt
              sudo parted /dev/md0 mkpart primary ext4 0% 10%
              sudo parted /dev/md0 mkpart primary ext4 10% 20%
              sudo parted /dev/md0 mkpart primary ext4 20% 30%
              sudo parted /dev/md0 mkpart primary ext4 30% 40%
              sudo parted /dev/md0 mkpart primary ext4 40% 50%
              sudo parted /dev/md0 mkpart primary ext4 50% 60%
              sudo parted /dev/md0 mkpart primary ext4 60% 70%
              sudo parted /dev/md0 mkpart primary ext4 70% 80%
              sudo parted /dev/md0 mkpart primary ext4 80% 90%
              sudo parted /dev/md0 mkpart primary ext4 90% 100%
              sudo mkdir -p /raid/part{1,2,3,4,5,6,7,8,9,10}

              for i in $(seq 1 10); do mount /dev/md0p$i /raid/part$i; done
              for k in $(seq 1 10); do sudo mkfs.ext4 /dev/md0p$k; done
              for j in $(seq 1 10); do mount /dev/md0p$j /raid/part$j; done
	      echo "===========PARTS IS MOUNTED=========="
	      mount | grep "^/dev" | grep -v sd | awk {'print $1,$3,$5, " defaults  0 2"'} >> /etc/fstab
	      echo "FSTAB IS CHANGED"
	      cat /etc/fstab
	      echo "===========lsblk report========="
	      lsblk
              echo "===========MDSTAT==========="
	      cat /proc/mdstat
              echo "============================"
              echo "CREATING TEST FILE IN /RAID/PART1-10"
	      touch /raid/part{1,2,3,4,5,6,7,8,9,10}/file.txt
	      chown -R vagrant:vagrant /raid

          RAID_MOUNT
#-------------------------------------------------------------

      end
  end
end

