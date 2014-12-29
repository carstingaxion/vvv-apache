# -*- mode: ruby -*-
# vi: set ft=ruby :

dir = Dir.pwd
vagrant_dir = File.expand_path(File.dirname(__FILE__))

Vagrant.configure("2") do |config|

  # Store the current version of Vagrant for use in conditionals when dealing
  # with possible backward compatible issues.
  vagrant_version = Vagrant::VERSION.sub(/^v/, '')

  # Configurations from 1.0.x can be placed in Vagrant 1.1.x specs like the following.
  config.vm.provider :virtualbox do |v|
    v.customize ["modifyvm", :id, "--memory", 1024]

    v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    v.customize ["modifyvm", :id, "--natdnsproxy1", "on"]    
  end

  # Forward Agent
  #
  # Enable agent forwarding on vagrant ssh commands. This allows you to use identities
  # established on the host machine inside the guest. See the manual for ssh-add
  config.ssh.forward_agent = true

  # Default Ubuntu Box
  #
  # This box is provided by Vagrant at vagrantup.com and is a nicely sized (290MB)
  # box containing the Ubuntu 12.0.4 Precise 32 bit release. Once this box is downloaded
  # to your host computer, it is cached for future use under the specified box name.
  config.vm.box = "precise32"
  config.vm.box_url = "http://files.vagrantup.com/precise32.box"

  config.vm.hostname = "vvv-apache"

  # Local Machine Hosts
  #
  # If the Vagrant plugin hostsupdater (https://github.com/cogitatio/vagrant-hostsupdater) is
  # installed, the following will automatically configure your local machine's hosts file to
  # be aware of the domains specified below. Watch the provisioning script as you may be
  # required to enter a password for Vagrant to access your hosts file.
  #
  # By default, we'll include the domains setup by VVV through the vvv-hosts file
  # located in the www/ directory.
  #
  # Other domains can be automatically added by including a vvv-hosts file containing
  # individual domains separated by whitespace in subdirectories of www/.
  if defined? VagrantPlugins::HostsUpdater

    # Capture the paths to all vvv-hosts files under the www/ directory.
    paths = []
    Dir.glob(vagrant_dir + '/www/**/vvv-hosts').each do |path|
      paths << path
    end

    # Parse through the vvv-hosts files in each of the found paths and put the hosts
    # that are found into a single array.
    hosts = []
    paths.each do |path|
      new_hosts = []
      file_hosts = IO.read(path).split( "\n" )
      file_hosts.each do |line|
        if line[0..0] != "#"
          new_hosts << line
        end
      end
      hosts.concat new_hosts
    end

    # Pass the final hosts array to the hostsupdate plugin so it can perform magic.
    config.hostsupdater.aliases = hosts
    config.hostsupdater.remove_on_suspend = true
  end

  # Default Box IP Address
  #
  # This is the IP address that your host will communicate to the guest through. In the
  # case of the default `192.168.50.4` that we've provided, VirtualBox will setup another
  # network adapter on your host machine with the IP `192.168.50.1` as a gateway.
  #
  # If you are already on a network using the 192.168.50.x subnet, this should be changed.
  # If you are running more than one VM through VirtualBox, different subnets should be used
  # for those as well. This includes other Vagrant boxes.
  
  # default
  config.vm.network :private_network, ip: "192.168.50.4"

  #
  # https://github.com/Varying-Vagrant-Vagrants/VVV/issues/442#issue-42059056
  #config.vm.network :public_network
  
  #config.vm.network :public_network, :bridge=>"Dell Wireless 1703 802.11b/g/n (2.4GHz)"

  #config.vm.network "public_network", ip: "192.168.2.333"

  





  # Make bitbucket and github avaiable via VM
  # https://github.com/Varying-Vagrant-Vagrants/VVV/issues/360#issuecomment-51645545
  #config.ssh.username = "carstenbach"
  #config.ssh.private_key_path = [ "F:\\Users\\caba\\.vagrant.d\\insecure_private_key", "F:\\Users\\caba\\.ssh\\id_rsa.pub" ]
  config.ssh.private_key_path = [ '~/.vagrant.d/insecure_private_key', '~/.ssh/id_rsa', '~/.ssh/github_rsa' ]

  
  #config.vm.provision :shell, :inline => "echo -e '#{File.read("#{Dir.home}/.ssh/id_rsa")}' > 'F:\\Users\\caba\\.ssh\\id_rsa'"
  #config.vm.provision :shell, :inline => "echo -e '#{File.read("#{Dir.home}/.ssh/id_rsa.pub")}' > 'F:\\Users\\caba\\.ssh\\id_rsa.pub'"
  

  # https://github.com/DSpace/vagrant-dspace/blob/master/Vagrantfile#L105
  #
  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 8080 on the VM.
  # 
  # https://github.com/Varying-Vagrant-Vagrants/VVV/issues/263#issuecomment-36492020

  config.vm.network :forwarded_port, guest: 80, host: 8080, auto_correct: true
  config.vm.network :forwarded_port, guest: 22, host: 1234, auto_correct: true

  # If a port collision occurs (i.e. port 8080 on local machine is in use),
  # then tell Vagrant to use the next available port between 8081 and 8100
  #config.vm.usable_port_range = 8081..8100



  # THIS NEXT PART IS TOTAL HACK (only necessary for running Vagrant on Windows)
  # Windows currently doesn't support SSH Forwarding when running Vagrant's "Provisioning scripts" 
  # (e.g. all the "config.vm.provision" commands below). Although running "vagrant ssh" (from Windows commandline) 
  # will work for SSH Forwarding once the VM has started up, "config.vm.provision" commands in this Vagrantfile DO NOT.
  # Supposedly there's a bug in 'net-ssh' gem (used by Vagrant) which causes SSH forwarding to fail on Windows only
  # See: https://github.com/mitchellh/vagrant/issues/1735
  #      https://github.com/mitchellh/vagrant/issues/1404
  # See also underlying 'net-ssh' bug: https://github.com/net-ssh/net-ssh/issues/55
  #
  # Therefore, we have to "hack it" and manually sync our SSH keys to the Vagrant VM & copy them over to the 'root' user account
  # (as 'root' is the account that runs all Vagrant "config.vm.provision" scripts below). This all means 'root' should be able 
  # to connect to GitHub as YOU! Once this Windows bug is fixed, we should be able to just remove these lines and everything 
  # should work via the "config.ssh.forward_agent=true" setting.
  # ONLY do this hack/workaround if the local OS is Windows.
  if Vagrant::Util::Platform.windows?

      # MORE SECURE HACK. You MUST have a ~/.ssh/id_rsa (BitBucket specific) SSH key to copy to VM
      # (ensures we are not just copying all your local SSH keys to a VM)
      if File.exists?(File.join(Dir.home, ".ssh", "id_rsa"))
          # Read local machine's BitBucket SSH Key (~/.ssh/id_rsa)
          bitbucket_ssh_key = File.read(File.join(Dir.home, ".ssh", "id_rsa"))
          # Copy it to VM as the /root/.ssh/id_rsa key
          config.vm.provision :shell, :inline => "echo 'Windows-specific: Copying local Bitbucket SSH Key to VM for provisioning...' && mkdir -p /root/.ssh && echo '#{bitbucket_ssh_key}' > /root/.ssh/id_rsa && chmod 600 /root/.ssh/id_rsa"
      else
          # Else, throw a Vagrant Error. Cannot successfully startup on Windows without a BitBucket SSH Key!
          raise Vagrant::Errors::VagrantError, "\n\nERROR: BitBucket SSH Key not found at ~/.ssh/id_rsa (required for 'vagrant-dspace' on Windows).\nYou can generate this key manually OR by installing GitHub for Windows (http://windows.github.com/)\n\n"
      end  

      # MORE SECURE HACK. You MUST have a ~/.ssh/github_rsa (GitHub specific) SSH key to copy to VM
      # (ensures we are not just copying all your local SSH keys to a VM)
      if File.exists?(File.join(Dir.home, ".ssh", "github_rsa"))
          # Read local machine's GitHub SSH Key (~/.ssh/github_rsa)
          github_ssh_key = File.read(File.join(Dir.home, ".ssh", "github_rsa"))
          # Copy it to VM as the /root/.ssh/github_rsa key
          config.vm.provision :shell, :inline => "echo 'Windows-specific: Copying local GitHub SSH Key to VM for provisioning...' && mkdir -p /root/.ssh && echo '#{github_ssh_key}' > /root/.ssh/github_rsa && chmod 600 /root/.ssh/github_rsa"
      else
          # Else, throw a Vagrant Error. Cannot successfully startup on Windows without a GitHub SSH Key!
          raise Vagrant::Errors::VagrantError, "\n\nERROR: GitHub SSH Key not found at ~/.ssh/github_rsa (required for 'vagrant-dspace' on Windows).\nYou can generate this key manually OR by installing GitHub for Windows (http://windows.github.com/)\n\n"
      end  

  end

  ####
  # Provisioning Scripts
  #    These scripts run in the order in which they appear, and setup the virtual machine (VM) for us.
  ####

  # Create a '/etc/sudoers.d/root_ssh_agent' file which ensures sudo keeps any SSH_AUTH_SOCK settings
  # This allows sudo commands (like "sudo ssh git@github.com") to have access to local SSH keys (via SSH Forwarding)
  # See: https://github.com/mitchellh/vagrant/issues/1303
  config.vm.provision :shell do |shell|
      shell.inline = "touch $1 && chmod 0440 $1 && echo $2 > $1"
      shell.args = %q{/etc/sudoers.d/root_ssh_agent "Defaults    env_keep += \"SSH_AUTH_SOCK\""}
  end

  # Turn off annoying console bells/beeps in Ubuntu (only if not already turned off in /etc/inputrc)
  config.vm.provision :shell, :inline => "grep '^set bell-style none' /etc/inputrc || echo 'set bell-style none' >> /etc/inputrc"











  # Drive mapping
  #
  # The following config.vm.synced_folder settings will map directories in your Vagrant
  # virtual machine to directories on your local machine. Once these are mapped, any
  # changes made to the files in these directories will affect both the local and virtual
  # machine versions. Think of it as two different ways to access the same file. When the
  # virtual machine is destroyed with `vagrant destroy`, your files will remain in your local
  # environment.

  # /srv/database/
  #
  # If a database directory exists in the same directory as your Vagrantfile,
  # a mapped directory inside the VM will be created that contains these files.
  # This directory is used to maintain default database scripts as well as backed
  # up mysql dumps (SQL files) that are to be imported automatically on vagrant up
  config.vm.synced_folder "database/", "/srv/database"
  if vagrant_version >= "1.3.0"
    config.vm.synced_folder "database/data/", "/var/lib/mysql", :mount_options => [ "dmode=777", "fmode=777" ]
  else
    config.vm.synced_folder "database/data/", "/var/lib/mysql", :extra => 'dmode=777,fmode=777'
  end

  # /srv/config/
  #
  # If a server-conf directory exists in the same directory as your Vagrantfile,
  # a mapped directory inside the VM will be created that contains these files.
  # This directory is currently used to maintain various config files for php and
  # Apache as well as any pre-existing database files.
  config.vm.synced_folder "config/", "/srv/config"

  # /srv/www/
  #
  # If a www directory exists in the same directory as your Vagrantfile, a mapped directory
  # inside the VM will be created that acts as the default location for Apache sites. Put all
  # of your project files here that you want to access through the web server
  if vagrant_version >= "1.3.0"
    config.vm.synced_folder "www/", "/srv/www/", :owner => "www-data", :mount_options => [ "dmode=775", "fmode=774" ]
  else
    config.vm.synced_folder "www/", "/srv/www/", :owner => "www-data", :extra => 'dmode=775,fmode=774'
  end

  # Customfile - POSSIBLY UNSTABLE
  #
  # Use this to insert your own (and possibly rewrite) Vagrant config lines. Helpful
  # for mapping additional drives. If a file 'Customfile' exists in the same directory
  # as this Vagrantfile, it will be evaluated as ruby inline as it loads.
  #
  # Note that if you find yourself using a Customfile for anything crazy or specifying
  # different provisioning, then you may want to consider a new Vagrantfile entirely.
  if File.exists?(File.join(vagrant_dir,'Customfile')) then
    eval(IO.read(File.join(vagrant_dir,'Customfile')), binding)
  end

  # Provisioning
  #
  # Process one or more provisioning scripts depending on the existence of custom files.
  #
  # provison-pre.sh acts as a pre-hook to our default provisioning script. Anything that
  # should run before the shell commands laid out in provision.sh (or your provision-custom.sh
  # file) should go in this script. If it does not exist, no extra provisioning will run.
  if File.exists?(File.join(vagrant_dir,'provision','provision-pre.sh')) then
    config.vm.provision :shell, :path => File.join( "provision", "provision-pre.sh" )
  end

  # provision.sh or provision-custom.sh
  #
  # By default, Vagrantfile is set to use the provision.sh bash script located in the
  # provision directory. If it is detected that a provision-custom.sh script has been
  # created, that is run as a replacement. This is an opportunity to replace the entirety
  # of the provisioning provided by default.
  if File.exists?(File.join(vagrant_dir,'provision','provision-custom.sh')) then
    config.vm.provision :shell, :path => File.join( "provision", "provision-custom.sh" )
  else
    config.vm.provision :shell, :path => File.join( "provision", "provision.sh" )
  end

  # provision-post.sh acts as a post-hook to the default provisioning. Anything that should
  # run after the shell commands laid out in provision.sh or provision-custom.sh should be
  # put into this file. This provides a good opportunity to install additional packages
  # without having to replace the entire default provisioning script.
  if File.exists?(File.join(vagrant_dir,'provision','provision-post.sh')) then
    config.vm.provision :shell, :path => File.join( "provision", "provision-post.sh" )
  end
end