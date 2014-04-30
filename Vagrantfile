# -*- mode: ruby -*-
# vim: set ft=ruby sw=3 ts=3 et:

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
   config.vm.hostname = "francalab-precise64"

   # Precise32 box from Hashicorp (the company behind Vagrant)
   #config.vm.box = "hashicorp/precise32"
   #config.vm.box_url = "https://vagrantcloud.com/hashicorp/precise32/version/1/provider/virtualbox.box"

   # Precise64 box from Hashicorp (the company behind Vagrant)
   config.vm.box = "hashicorp/precise64"
   config.vm.box_url = "https://vagrantcloud.com/hashicorp/precise64/version/1/provider/virtualbox.box"

   # Precise64 box from Ubuntu cloud images
   #config.vm.box = "ubuntu/precise64"
   # If above box does not exist locally, fetch it here:
   #config.vm.box_url = "http://cloud-images.ubuntu.com/vagrant/precise/current/precise-server-cloudimg-amd64-vagrant-disk1.box"

   # To run eclipse we need more than default RAM 512MB And we might as well
   # set a useful name also, which I prefer to have equal to the hostname that
   # was defined above, but to make it unique a timestamp is added also.
   # Increase video RAM as well, it doesn't cost much and we will run
   # graphical desktops after all.
   #vmname = config.vm.hostname + "-" + `date +%Y%m%d%H%M`.to_s
   #vmname.chomp!      # Without this there is a newline character in the name :-o
   config.vm.provider :virtualbox do |vb|
      # Don't boot with headless mode
      vb.gui = true

      #vb.customize [ "modifyvm", :id, "--name", vmname ]
      vb.customize [ "modifyvm", :id, "--memory", "1536" ]
      vb.customize [ "modifyvm", :id, "--vram", "128" ]
   end

   config.vm.provision :shell, inline:
      '# Quick fix to enable vbguest on VirtualBox 4.3.10
      # See https://github.com/mitchellh/vagrant/issues/3341
      if [ ! -e /usr/lib/VBoxGuestAdditions ]; then
          sudo ln -s /opt/VBoxGuestAdditions-4.3.10/lib/VBoxGuestAdditions \
	      /usr/lib/VBoxGuestAdditions || true
      fi'

   # Warning to user
   config.vm.provision :shell, inline:
      'echo "***************************************************************"
       echo "Starting provisioning. "
       echo
       echo "!!!!!!!!!!!!"
       echo "!!!      !!! You may see errors in dpkg-preconfigure."
       echo "!!! NOTE !!! It will look like a significant error in the final step"
       echo "!!!      !!! (cannot reopen stdin, etc). This can be ignored."
       echo "!!!!!!!!!!!!"
       echo
       echo "When provisioning is done, halt the VM, then boot normally "
       echo "with a GUI inside Virtualbox, i.e. not using vagrant..."
       echo
       echo "Then run eclipse, probably at: ~vagrant/tools/autoeclipse/eclipse"
       echo "***************************************************************"'

   # Install prerequisites
   config.vm.provision :shell, inline:
      'sudo apt-get update; sudo apt-get install -y curl wget unzip openjdk-6-jre'

   # Run the eclipse + franca installer script
   config.vm.provision :shell, :path => "script.sh"

   # Create desktop icon
   config.vm.provision :shell, inline:
      'desktopdir=/home/vagrant/Desktop
   sudo mkdir -p $desktopdir
   shortcut=/home/vagrant/Desktop/Eclipse.desktop
   cat <<EOT >$shortcut
[Desktop Entry]
Encoding=UTF-8
Type=Application
Name=Eclipse with Franca
Name[en_US]=Eclipse with Franca
Icon=/home/vagrant/tools/autoeclipse/eclipse/icon.xpm
Exec=/home/vagrant/tools/autoeclipse/eclipse/eclipse
EOT

chmod 755 $shortcut

# The above created files, and others are owned by root after provisioning 
# Fix that:
sudo chown -R vagrant:vagrant /home/vagrant

# Remove other users than vagrant -- makes things less confusing when logging in
# Notice that some Vagrant boxes (most notably hashicorp/precise64)
# do not have those extra users defined
USERS_TO_DISABLE="ubuntu"
for user in $USERS_TO_DISABLE; do
    if id $user &>/dev/null; then
		echo Disabling user $user
        #sudo deluser $user
		sudo usermod -L -e 1 $user
    fi
done
      '

   # Warning, again
   config.vm.provision :shell, inline:
   'echo "***************************************************************"
    echo "Executing final step: install LXDE graphical environment"
    echo
    echo "!!!!!!!!!!!! Reminder:"
    echo "!!! NOTE !!! Installing LXDE fails in dpkg-preconfigure."
    echo "!!!!!!!!!!!! This can be ignored, it seems OK anyway - try it."
    echo
    echo "When provisioning is done, halt the VM, then boot normally "
    echo "with a GUI inside Virtualbox, i.e. not using vagrant..."
    echo ""
    echo "Then run eclipse, probably at: ~vagrant/tools/autoeclipse/eclipse"
    echo "Read the project README!"
    echo "***************************************************************"'

   # Install graphical environment
   config.vm.provision :shell, inline:
   'sudo apt-get install -y lxde

    # Workaround for debconf database corruption
    # See http://forums.debian.net/viewtopic.php?f=10&t=101659
    sudo /usr/share/debconf/fix_db.pl
   '

   # ----------------------------------------------
   # If VM will run some network services e.g. web browser
   # make a port forward so we can test them from the host:
   # host:4567 -->  Virtual Machine port 80
   # ----------------------------------------------

   # config.vm.network :forwarded_port, host: 4567, guest: 80

end
