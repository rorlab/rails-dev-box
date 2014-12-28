# 가상머신을 이용한 레일스 애플리케이션 개발환경

## 개요

레일스 애플리케이션 개발시에 로컬 머신의 개발환경은 각자 다를 수 있어서 동일한 소스를 각기 다른 머신에서 개발할 때 *"works on my machine"* 버그가 발생할 수 있다. 이 때 **Vagrant**를 이용하면 개발환경(서버 프로비져닝 포함)을 패키징하여 **box** 형태로 공유할 수 있게 된다. 따라서 각자 이 **Vagrant box**를 다운로드 받은 후 각자의 프로젝트 디렉토리에서 `vagrant init path/to/vagrantbox`를 실행한 후 `vagrant up`명령을 실행하면 동일한 개발 환경을 가상 머신으로 구축할 수 있다. 이후 `vagrant ssh` 명령을 실행하여 해당 가상 머신으로 진입하여 `cd /vagrant` 명령을 실행하면 호스트 머신 상의 프로젝트 디렉토리로 이동하게 된다. 여기서에서 `bundle install`한 후 `bin/rails s -b 192.168.56.101`명령으로 서버를 실행한 후 호스트 브라우저에서 `http://192.168.56.101:3000`으로 접속한다. 물론 가상 서버내에서 데이터베이스를 생성한 후 권한 설정을 해 놓은 후 DB 마이그레이션 작업을 해 놓아야 한다. 이 글은 깃헙의 레일스 그룹에서 레일스 코어 개발을 위해 만든 `rails-dev-box`(Vagrant box)를 포크한 후 약간수정하여 만든 [`rorlab/rails-dev-box`](https://github.com/rorlab/rails-dev-box)를 예로 들어 설명한다.

> **주의 :** 초보자의 경우 혼란을 가져올 수 있는 것은 현재 설명하고 있는 것을 개발환경에서 development 모드로 서버를 실행하다는 것이다.

## 사전준비 작업

1. **Vagrant** 설치하기 https://www.vagrantup.com/downloads

    : **Vagrant**는 쉽게 설정할 수 있고 재생산이 가능하며 이식성이 있는 개발환경을 만들어 준다. 쉡스크리트, **Chef**, **Puppet**과 같은 **provisoning tool**을 이용하여 **VirtualBox**, **VMware**, **AWS** 등과 같은 **provider**에  가상 머신을 설치하고 소프트웨어를 설정할 있다. 더 자세한 내용은 웹사이트를 참조하기 바란다. https://docs.vagrantup.com/v2/

2. **VirtualBox** 설치하기 https://www.virtualbox.org/wiki/Downloads

       : 가장 손쉽게 구할 수 있는 가상머신 툴이며 [VirtualBox의 완벽설치](http://niceit.tistory.com/187)를 참고하면 설치에 대해서 자세히 알 수 있다


## 1. base box로부터 Rails 애플리케이션 개발을 위한 가상머신 생성하기

1. git 소스를 클론 받아 `vagrant up` 명령으로 가상머신(디폴트: **Virtualbox**)을 생성한다. 이 소스의 **Vagrantfile**에는 쉡스크립트(`bootstrap.sh` 파일)을 이용한 `provisioning` 작업(머신에 일련의 소프트웨어를 설치하는 과정)이 포함되어 있다.

    ``` bash
    $ git clone git@github.com:rorlab/rails-dev-box.git rails-dev
    $ cd rails-dev
    $ vagrant up
    ```
    > **참고 :**
      Vagrantfile
  ```ruby
  # -*- mode: ruby -*-
  # vi: set ft=ruby :
  Vagrant.configure('2') do |config|
    config.vm.box      = 'ubuntu/trusty64'
    config.vm.hostname = 'rorlab-dev'
    config.vm.provision :shell, path: 'bootstrap.sh', keep_color: true
    config.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"
  end
  ```
  [bootstrap.sh](https://github.com/rorlab/rails-dev-box/blob/master/bootstrap.sh)


2. **ssh**로 가상머신으로 진입한 후 환경을 확인하고

    ``` bash
    $ vagrant ssh
    vagrant@rails-dev:~$ ruby -v
    ruby 2.2.0preview1 (2014-09-17 trunk 47616) [x86_64-linux-gnu]

    vagrant@rails-dev:~$ rails -v
    Rails 4.2.0

    vagrant@rails-dev:~$ sudo vi /etc/network/interfaces.d/eth1.cfg
    # The host-only network interface
    auto eth1
    iface eth1 inet static
    address 192.168.56.101
    netmask 255.255.255.0
    network 192.168.56.0
    broadcast 192.168.56.255
    ```

3. `exit` 명령으로 게스트 가상머신에서 빠져나온 후 현재의 가상머신을 **package**하여 새로운 **box**를 생성한다. 방금 생성된 **box** 파일을 `~/VagrantBoxes/rails-dev.box`로 이동한다.

    ``` bash
    vagrant@rails-dev:~$ exit
    $ vagrant package
    $ mv package.box ~/VagrantBoxes/rails-dev.box
    ```

4. 이제 서버 **provisioning 작업**이 되어 있는 **Vagrant box** 파일(rails-dev.box)을 얻게 되었다. 지금부터는 이 **box**를 이용하여 레일스 프로젝트에서 개발을 위한 환경으로 사용할 수 있는 가상머신을 생성하도록 해 보자.

## 2. Vagrant로 레일스 애플리케이션 개발 환경 구축하기

* 레일스 프로젝트 디렉토리로 이동한다.

    ```bash
    $ cd rails-project
    $ vagrant init ~/VagrantBoxes/rails-dev.box
    ```

    > **주의 :** 지금까지의 작업을 공개 서버로 올려 놓았으니 이것을 바로 **base box**로 사용해도 된다. **[lucius/rorlab_dev](https://atlas.hashicorp.com/lucius/boxes/rorlab_dev)**
    ```bash
    $ vagrant init lucius/rorlab_dev
    ```
    그러나 **[atlas.hashicorp.com](http://atlas.hashicorp.com)** 서버로 부터 box를 다운로드 받는데는 시간이 걸리므로 시간적 여유가 있을 때 아래와 같이 로컬 머신으로 받아 놓는 것도 좋은 방법이다.
      ```bash
$ vagrant box add -f lucius/rorlab_dev
==> box: Loading metadata for box 'lucius/rorlab_dev'
    box: URL: https://atlas.hashicorp.com/lucius/rorlab_dev
==> box: Adding box 'lucius/rorlab_dev' (v1.1.0) for provider: virtualbox
    box: Downloading: https://atlas.hashicorp.com/lucius/boxes/rorlab_dev/versions/1.1.0/providers/virtualbox.box
==> box: Successfully added box 'lucius/rorlab_dev' (v1.1.0) for 'virtualbox'!
      ```


* 위의 결과로 생성된 **Vagrantfile**을 아래와 같이 수정한다. 이미 **box** 이미지에는 게스트 가상머신에서 고정 **IP(192.168.56.101)**를 사용할 수 있도록 **eth1** 네트워크 인터페이스 옵션을 설정해 놓았기 때문에 이를 사용하기 위해 `config.vm.network` 옵션을 추가해 준다. **어댑터 2**에 **eth1**이 연결되어 있고 **3000**포트로 연결할 것이기 때문에, `adapter` 옵션을 지정하고 **3000**포트로 **포트포워딩**한다.


    ``` ruby
    # -*- mode: ruby -*-
    # vi: set ft=ruby :

    # All Vagrant configuration is done below. The "2" in Vagrant.configure
    # configures the configuration version (we support older styles for
    # backwards compatibility). Please don't change it unless you know what
    # you're doing.

    Vagrant.configure(2) do |config|
      config.vm.box = "~/VagrantBoxes/rorlab-dev.box"
      config.vm.hostname = 'rorlab-dev'
      config.vm.synced_folder ".", "/vagrant", :nfs => true
      config.vm.network "private_network", adapter: 2, ip: "192.168.56.101", auto_config: false
      config.vm.network "forwarded_port", adapter: 2, guest: 3000, host: 3000
    end
    ```

## 3. 레일스 애플리케이션에 접속하기

* 로컬 머신의 레일스 애플리케이션 디렉토리에서 가상머신으로 접속한다.

  ```
  $ vagrant ssh
  Welcome to Ubuntu 14.04.1 LTS (GNU/Linux 3.13.0-43-generic x86_64)

   * Documentation:  https://help.ubuntu.com/

    System information as of Sat Dec 27 03:21:29 UTC 2014

    System load:  0.3               Processes:           123
    Usage of /:   4.2% of 39.34GB   Users logged in:     0
    Memory usage: 8%                IP address for eth0: 10.0.2.15
    Swap usage:   0%                IP address for eth1: 192.168.56.101

    Graph this data and manage this system at:
      https://landscape.canonical.com/

    Get cloud support with Ubuntu Advantage Cloud Guest:
      http://www.ubuntu.com/business/services/cloud


  Last login: Sat Dec 27 03:20:19 2014 from 10.0.2.2
  vagrant@rorlab-dev:~$
  ```

* MySQL 서버로 접속하여 애플리케이션에서 사용할 데이터베이스를 생성하고 권한을 준다.

  ```
vagrant@rails-dev:~$ mysql -u root -p
  Enter password:
  Welcome to the MySQL monitor.  Commands end with ; or \g.
  Your MySQL connection id is 36
  Server version: 5.5.40-0ubuntu0.14.04.1 (Ubuntu)

  Copyright (c) 2000, 2014, Oracle and/or its affiliates. All rights reserved.

  Oracle is a registered trademark of Oracle Corporation and/or its
  affiliates. Other names may be trademarks of their respective
  owners.

  Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

  mysql> show databases;
  +------------------------+
  | Database               |
  +------------------------+
  | information_schema     |
  | activerecord_unittest  |
  | activerecord_unittest2 |
  | mysql                  |
  | performance_schema     |
  +------------------------+
  5 rows in set (0.01 sec)

  mysql> CREATE USER 'deployer'@'localhost';
  Query OK, 0 rows affected (0.01 sec)

  mysql> CREATE DATABASE blog_development DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
  Query OK, 1 row affected (0.01 sec)

  mysql> GRANT ALL PRIVILEGES ON blog_development.* to 'deployer'@'localhost';
  Query OK, 0 rows affected (0.00 sec)

  mysql> exit
  Bye
  vagrant@rorlab-dev:~$ mysql -u deployer -p
  Enter password:
  Welcome to the MySQL monitor.  Commands end with ; or \g.
  Your MySQL connection id is 37
  Server version: 5.5.40-0ubuntu0.14.04.1 (Ubuntu)

  Copyright (c) 2000, 2014, Oracle and/or its affiliates. All rights reserved.

  Oracle is a registered trademark of Oracle Corporation and/or its
  affiliates. Other names may be trademarks of their respective
  owners.

  Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

  mysql> show databases;
  +--------------------+
  | Database           |
  +--------------------+
  | information_schema |
  | blog_development   |
  +--------------------+
  2 rows in set (0.00 sec)
  ```


* `/vagrant` 디렉토리로 이동하고,

  ```
  $ vagrant@rails-dev:~$ cd /vagrant
  $ vagrant@rails-dev:~$ bin/rake db:schema:load
  $ vagrant@rails-dev:~$ bin/rails s -b 192.168.56.101
  ```

* 이제 브라우저에서 **http://192.168.56.101:3000** 으로 접속한다.


## Problem Solving

```bash
Warning: Running `gem pristine --all` to regenerate your installed gemspecs (and deleting then reinstalling your bundle if you use bundle --path) will improve the startup performance of Spring.
```




--

_*References :*_

1. [[Virtual Box] : Host로부터 Guest로 고정 IP로 연결하기](http://rorla.rorlab.org/rblogs/25)
2. [Using Vagrant for Rails Development](https://gorails.com/guides/using-vagrant-for-rails-development)
3. [How to configure a development environment for Rails using Vagrant](http://robertobartolome.com/configure-development-environment-rails-using-vagrant/)
4. [How To Install Ruby on Rails with rbenv on Debian 7 (Wheezy)](https://www.digitalocean.com/community/tutorials/how-to-install-ruby-on-rails-with-rbenv-on-debian-7-wheezy)
5. [Introducing Foreman](http://blog.daviddollar.org/2011/05/06/introducing-foreman.html)
6. [Use NFS to speed up your Vagrant](https://coderwall.com/p/uaohzg/use-nfs-to-speed-up-your-vagrant)
7. [Ubuntu / Kubuntu 에서 간단하게 Qt 설치하기](http://qtguru.blogspot.kr/2014/03/ubuntu-kubuntu-qt.html)