require 'yaml'
Vagrant.configure("2") do |config|
    # Configure the box to use
    config.vm.box       = 'ubuntu/artful64'
    #config.vm.box_url   = 'http://files.vagrantup.com/artful64.box'

    if File.exist?('../vagrant.yml') then
        sconf = YAML::load(File.read("../vagrant.yml"))
        hostname = sconf['vagrant']['hostname']
        extip = sconf['vagrant']['ip']
        extname = sconf['vagrant']['name']
        folder = sconf['vagrant']['folder']
        forwards = sconf['vagrant']['forwards']
    else
        hostname = 'default.helios'
        extip = '192.168.33.90'
        extname = (0...8).map { (65 + rand(26)).chr }.join
        folder = false
        forwards = false
    end

    config.vm.hostname = hostname

    # Configure the network interfaces
    config.vm.network :private_network, ip:   extip
    config.vm.network :forwarded_port, guest: 8081,  host: 8081
    config.vm.network :forwarded_port, guest: 27017, host: 27017

    if forwards != false then
        forwards.each do |entry|
            entry.each do |key, glob|
                host = glob['host']
                guest = glob['guest']
                config.vm.network :forwarded_port, guest: guest, host: host, id: key, :nfs => true
            end
        end
    else
        config.vm.network :forwarded_port, guest: 22,    host: 2201, id: 'ssh'
        config.vm.network :forwarded_port, guest: 3306,  host: 3307
        config.vm.network :forwarded_port, guest: 80,    host: 8089
    end




    config.ssh.forward_agent = true

    # Configure shared folders
    if folder != false then
        folder.each do |entry|
            entry.each do |key, glob|
                host = glob['host']
                guest = glob['guest']
                config.vm.synced_folder host,  guest, id: key, :nfs => true
            end
        end
    else
        config.vm.synced_folder ".",  "/vagrant", id: "vagrant-root", :nfs => true
        config.vm.synced_folder "..", "/var/www", id: "application",  :nfs => true
    end

    # Configure VirtualBox environment
    config.vm.provider :virtualbox do |v|
        v.name = extname
        v.customize [ "modifyvm", :id, "--memory", 1024 ]
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
