# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  domain_name = ENV['GPP_HYRAX_ANSIBLE_DOMAIN_NAME'].to_s.empty? ? 'gpp-hyrax-ansible.test' : ENV['GPP_HYRAX_ANSIBLE_DOMAIN_NAME']
  ssh_key = ENV['GPP_HYRAX_ANSIBLE_SSH_KEY_PATH'].to_s.empty? ? "#{Dir.home}/.ssh/id_rsa.pub" : ENV['GPP_HYRAX_ANSIBLE_SSH_KEY_PATH']

  config.vm.box = "rhel-7.6"

  vms = {
    'default' => {
      ip: '192.168.10.100',
      user: 'vagrant',
      box: "rhel-7.6"
    }
  }

  vms.each do |name, options|
    config.vm.define name do |node|
      node.vm.hostname = "#{name}.#{domain_name}"
      node.vm.network 'private_network', {ip: options.fetch(:ip)}
      node.vm.box = options.fetch(:box, 'rhel-7.6')
      node.vm.provider 'virtualbox' do |v|
        v.cpus = options.fetch(:cpus, 1)
        v.memory = options.fetch(:memory, 512)
      end
      user = options.fetch(:user, 'vagrant')

      node.vm.provision 'shell' do |s|
        ssh_pub_key = File.readlines(ssh_key).first.strip
        # ensure we can SSH directly to the VM
        s.inline = <<-SHELL
          set -ex
          echo #{ssh_pub_key} >> /home/#{user}/.ssh/authorized_keys
        SHELL
      end
    end
  end
  
  config.trigger.before :destroy do |trigger|
    $script = <<-SCRIPT
    subscription-manager unsubscribe --all; true
    subscription-manager unregister; true
    SCRIPT

    trigger.run_remote = {inline: $script}
  end
end
