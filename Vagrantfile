# -*- mode: ruby -*-
# vi: set ft=ruby :

$box = 'ubuntu/trusty64'
$box_url = nil
$box_version = nil
$rancher_version = 'latest'
$ip_prefix = '192.168.33'
$N_NODES = 3

Vagrant.configure(2) do |config|
  if (/cygwin|mswin|mingw|bccwin|wince|emx/ =~ RUBY_PLATFORM) != nil
    config.vm.synced_folder ".", "/vagrant", mount_options: ["dmode=700,fmode=600"]
  else
    config.vm.synced_folder ".", "/vagrant"
  end

  config.vm.define "cd" do |d|
    d.vm.box = $box
    d.vm.box_url = $box_url unless $box_url.nil?
    d.vm.box_version = $box_version unless $box_version.nil?
    d.vm.hostname = "cd"
    d.vm.network "private_network", ip: "#{$ip_prefix}.10"
    d.vm.provision :shell, path: "scripts/bootstrap_ansible.sh"
    d.vm.provision :shell, inline: "PYTHONUNBUFFERED=1 ansible-playbook /vagrant/ansible/cd.yml -c local"
    d.vm.provision :shell, inline: "PYTHONUNBUFFERED=1 ansible-playbook /vagrant/ansible/rancher-server.yml -c local"
    d.vm.synced_folder "./jenkins", "/data/jenkins", create: true
    d.vm.synced_folder "./.vagrant/machines", "/machines"
    d.vm.provider "virtualbox" do |v|
      v.memory = 2048
      v.cpus = 2
    end
  end

  (1..$N_NODES).each do |i|
    config.vm.define "node-#{i}" do |d|
      d.vm.box = $box
      d.vm.box_url = $box_url unless $box_url.nil?
      d.vm.box_version = $box_version unless $box_version.nil?
      d.vm.hostname = "node#{i}"
      d.vm.network "private_network", ip: "#{$ip_prefix}.#{i+11}"
      d.vm.provision :shell, path: "scripts/bootstrap_ansible.sh"
      d.vm.provision :shell, inline: "PYTHONUNBUFFERED=1 ansible-playbook /vagrant/ansible/node.yml -c local"
      d.vm.provision :shell, inline: "PYTHONUNBUFFERED=1 ansible-playbook /vagrant/ansible/rancher-registeragent.yml -c local"
      d.vm.provider "virtualbox" do |v|
        v.memory = 1024
        v.cpus = 1
      end
    end
  end

  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :box
  end
  if Vagrant.has_plugin?("vagrant-vbguest")
    config.vbguest.auto_update = false
    config.vbguest.no_install = true
    config.vbguest.no_remote = true
  end
end
