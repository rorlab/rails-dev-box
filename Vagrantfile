# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure('2') do |config|
  config.vm.box      = 'ubuntu/trusty64'
  config.vm.hostname = 'rorlab-dev'
  config.vm.provision :shell, path: 'bootstrap.sh', keep_color: true
  config.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"
end