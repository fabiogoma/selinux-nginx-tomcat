hosts = ['tomcat', 'nginx']

Vagrant.configure('2') do |config|
  config.vm.box = 'centos/7'

  config.vm.synced_folder '.', '/vagrant', disabled: true

  config.vm.provider :libvirt do |libvirt|
    libvirt.driver = 'kvm'
    libvirt.cpus = 2
    libvirt.memory = '2048'
  end

  hosts.each do |host|
    config.vm.define "#{host}" do |node|
      node.vm.provision 'ansible' do |ansible|
        ansible.limit = "#{host}"
        ansible.compatibility_mode = '2.0'
        ansible.verbose = false
        ansible.playbook = "provisioning/#{host}-playbook.yml"
      end
    end
  end
end
