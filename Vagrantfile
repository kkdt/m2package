# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "kkdt/c7dev"
  config.vm.define "m2package"
  config.vm.hostname = "m2package"

  config.vm.provider "virtualbox" do |vb|
      vb.name = "m2package"
      vb.memory = 1024
      vb.cpus = 1
  end

  # apache archiva
  config.vm.network "forwarded_port", guest: 8080, host: 8000

  config.vm.provision "archiva", type: "shell", inline: <<-SHELL
    mkdir -p /home/vagrant/.m2
cat <<EOF >> /home/vagrant/.m2/settings.xml
<settings>
  <mirrors>
    <mirror>
      <id>archiva.default</id>
      <url>http://localhost:8080/repository/cots/</url>
      <mirrorOf>external:*</mirrorOf>
    </mirror>
  </mirrors>
</settings>
EOF
    chown -R vagrant:vagrant /home/vagrant/.m2
    su - vagrant -c '/opt/archiva/bin/archiva start'
  SHELL

end
