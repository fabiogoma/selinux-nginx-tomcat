Vagrant.configure('2') do |config|
  config.vm.box = 'centos/7'
  #Replace eno1 with the apropriate name for your host main interface
  config.vm.network 'public_network', bridge: "eno1"
  config.vm.synced_folder '.', '/vagrant', disabled: true
  config.vm.provider 'virtualbox' do |vb|
    vb.memory = '2048'
  end

  config.vm.define 'tomcat' do |node|
    node.vm.hostname = 'tomcat'
    node.vm.provision 'ansible' do |ansible|
      ansible.limit = 'tomcat'
      ansible.compatibility_mode = '2.0'
      ansible.verbose = false
      ansible.playbook = 'provisioning/tomcat-playbook.yml'
    end
  end

  config.vm.define 'nginx' do |node|
    node.vm.hostname = 'nginx'
    node.vm.provision 'ansible' do |ansible|
      ansible.limit = 'nginx'
      ansible.compatibility_mode = '2.0'
      ansible.verbose = false
      ansible.playbook = 'provisioning/nginx-playbook.yml'
    end
  end
end
