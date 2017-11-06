# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.


Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "ubuntu/trusty64"
  config.vm.hostname = "myhive"
  config.vm.network "public_network", bridge: "WLAN"
  #config.proxy.http     = "http://10.0.2.2:8080"
  #config.proxy.https    = "http://10.0.2.2:8080"
  #config.proxy.no_proxy = "localhost,127.0.0.1,myhive"

  config.vm.provider "virtualbox" do |v|
    v.memory = 2048
  end

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision "shell", inline: <<-SHELL
    set -xe

    apt-get update


    ###### 安装 JDK -- 参考 https://www.digitalocean.com/community/tutorials/how-to-install-java-with-apt-get-on-ubuntu-16-04
    apt-get -y install default-jdk
    echo "export JAVA_HOME=/usr/lib/jvm/java-1.7.0-openjdk-amd64" >/etc/profile.d/java.sh


    ###### 安装伪分布模式 Hadoop-2.8.2 -- 参考 http://www.powerxing.com/install-hadoop
    id hadoop || useradd -m hadoop -s /bin/bash
    echo "hadoop ALL=(ALL) NOPASSWD:ALL" >/etc/sudoers.d/hadoop
    su - hadoop -c "ssh-keygen -f /home/hadoop/.ssh/id_rsa -P ''; cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys"
    curl -kO https://www-us.apache.org/dist/hadoop/common/hadoop-2.8.2/hadoop-2.8.2.tar.gz
    md5sum hadoop-2.8.2.tar.gz
    tar -zxf hadoop-2.8.2.tar.gz --transform 's|^hadoop-2.8.2|hadoop|' -C /usr/local/
    rm -f hadoop-2.8.2.tar.gz
    sed -i "s|^export JAVA_HOME=.*|export JAVA_HOME=/usr/lib/jvm/java-1.7.0-openjdk-amd64|" /usr/local/hadoop/etc/hadoop/hadoop-env.sh
    cp -an /usr/local/hadoop/etc/hadoop/core-site.xml /usr/local/hadoop/etc/hadoop/core-site.xml.cyp
    cp -an /usr/local/hadoop/etc/hadoop/hdfs-site.xml /usr/local/hadoop/etc/hadoop/hdfs-site.xml.cyp
    cat >/usr/local/hadoop/etc/hadoop/core-site.xml <<\EOF
<configuration>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>file:/usr/local/hadoop/tmp</value>
        <description>Abase for other temporary directories.</description>
    </property>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
EOF
    cat >/usr/local/hadoop/etc/hadoop/hdfs-site.xml <<\EOF
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:/usr/local/hadoop/tmp/dfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:/usr/local/hadoop/tmp/dfs/data</value>
    </property>
</configuration>
EOF
    chown -R hadoop: /usr/local/hadoop/
    su - hadoop -c "/usr/local/hadoop/bin/hdfs namenode -format"
    cat >/etc/profile.d/hadoop.sh <<\EOF
export HADOOP_HOME=/usr/local/hadoop
export HADOOP_COMMON_LIB_NATIVE_DIR=\\$HADOOP_HOME/lib/native
export PATH=\\$PATH:\\$HADOOP_HOME/bin:\\$HADOOP_HOME/sbin
EOF


  SHELL
end
