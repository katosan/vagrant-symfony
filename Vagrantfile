Vagrant.configure("2") do |config|
    # Configure the box to use
    config.vm.box       = 'precise64'
    config.vm.box_url   = 'http://files.vagrantup.com/precise64.box'

    config.vm.hostname = "jobcloud.helios"

    # Configure the network interfaces
    config.vm.network :private_network, ip:    "192.168.3.10"
    config.vm.network :forwarded_port, guest: 22,    host: 2201, id: 'ssh'
    config.vm.network :forwarded_port, guest: 80,    host: 8089
    config.vm.network :forwarded_port, guest: 8081,  host: 8081
    config.vm.network :forwarded_port, guest: 3306,  host: 3307
    config.vm.network :forwarded_port, guest: 27017, host: 27017


    # Configure shared folders
    config.vm.synced_folder ".",  "/vagrant", id: "vagrant-root", :nfs => true
    config.vm.synced_folder "..", "/var/www", id: "application",  :nfs => true

    # Configure VirtualBox environment
    config.vm.provider :virtualbox do |v|
        v.name = (0...8).map { (65 + rand(26)).chr }.join
        v.customize [ "modifyvm", :id, "--memory", 512 ]
    end

    # Provision the box
    config.vm.provision :ansible do |ansible|
        ansible.extra_vars = { ansible_ssh_user: 'vagrant' }
        ansible.playbook = "ansible/site.yml"
    end

    if (File.directory? File.expand_path("~/.dotfiles"))
      config.vm.synced_folder "~/.dotfiles", "/home/vagrant/.dotfiles", {
        :create => true,
        :mount_options => [ 'dmode=0755' ]
      }
    end
end
