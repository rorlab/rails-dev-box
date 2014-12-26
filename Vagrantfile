# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure('2') do |config|
  config.vm.box      = 'ubuntu/trusty64'
  config.vm.hostname = 'rorlab-dev'
  config.vm.network "forwarded_port", adapter: 2, guest: 3000, host: 3000
  config.vm.network "private_network", adapter: 2, ip: "192.168.56.101", auto_config: false
  config.vm.provision :shell, path: 'bootstrap.sh', keep_color: true
  config.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"
end