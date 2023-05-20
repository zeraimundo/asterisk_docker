Vagrant.configure("2") do |asterisk|
  asterisk.vm.box = "centos/7"
  asterisk.vm.hostname = "asterisk"
  asterisk.vm.network "public_network"
  asterisk.vm.provider "virtualbox" do |vb|
    vb.name = "Asterisk-Docker"
    vb.memory = "2048"
    vb.cpus = "2"
  end
  asterisk.vm.provision "shell", inline: <<-SHELL
  sudo yum update -y
  sudo yum install curl git vim make wget -y
  SHELL
end
