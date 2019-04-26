# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "kkdt/c7dev"
  config.vm.define "m2package"
  config.vm.hostname = "m2package"

  config.vm.provider "virtualbox" do |vb|
      vb.name = "m2package"
      vb.memory = 512
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
      <url>http://localhost:8080/repository/internal/</url>
      <mirrorOf>external:*</mirrorOf>
    </mirror>
  </mirrors>
</settings>
EOF
    chown -R vagrant:vagrant /home/vagrant/.m2
    echo "Starting Apache Archiva"
    su - vagrant -c '/opt/archiva/bin/archiva console > archiva.log &'
  SHELL

end
