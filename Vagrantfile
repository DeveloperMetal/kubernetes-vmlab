# Plugins
#
# Check if the first argument to the vagrant
# command is plugin or not to avoid the loop
if ARGV[0] != 'plugin'

    # Define the plugins in an array format
    required_plugins = [
        'vagrant-env',
        'vagrant-vbguest',
    ]         
  
    plugins_to_install = required_plugins.select { |plugin| not Vagrant.has_plugin? plugin }
    if not plugins_to_install.empty?
  
        puts "Installing plugins: #{plugins_to_install.join(' ')}"
        if system "vagrant plugin install #{plugins_to_install.join(' ')}"
            exec "vagrant #{ARGV.join(' ')}"
        else
            abort "Installation of one or more plugins has failed. Aborting."
        end
  
    end
end

Vagrant.configure("2") do |config|
    config.ssh.insert_key = false
    config.env.enable # Enable vagrant-env(.env)

    config.vm.provider "virtualbox" do |v|
        v.memory = ENV["RAM_TOTAL"]
        v.cpus = 2
    end
      
    config.vm.define "master" do |master|
        master.vm.box = ENV["IMAGE_NAME"]
        master.vm.network "public_network", bridge: ENV["NETWORK_PUBLIC_BRIDGE"], ip: ENV["MASTER_PUBLIC_IP"]
        master.vm.network "private_network", ip: "192.168.50.10"
        master.vm.network "forwarded_port", guest: 6443, host: 6443
        master.vm.hostname = "master"
        master.vm.provision "ansible" do |ansible|
            ansible.config_file = "ansible.cfg"
            ansible.playbook = "kubernetes-setup/master-playbook.yml"
            ansible.extra_vars = {
                node_ip: "192.168.50.10",
                image_code: ENV["IMAGE_CODE"]
            }
        end
    end

    (1..ENV["TOTAL_NODES"].to_i).each do |i|
        config.vm.define "node-#{i}" do |node|
            node.vm.box = ENV["IMAGE_NAME"]
            node.vm.network "private_network", ip: "192.168.50.#{i + 10}"
            node.vm.hostname = "node-#{i}"
            node.vm.provision "ansible" do |ansible|
                ansible.config_file = "ansible.cfg"
                ansible.playbook = "kubernetes-setup/node-playbook.yml"
                ansible.extra_vars = {
                    node_ip: "192.168.50.#{i + 10}",
                    image_code: ENV["IMAGE_CODE"],
                }
            end
        end
    end
end