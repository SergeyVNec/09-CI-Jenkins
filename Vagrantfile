Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-20.04"
  config.vm.hostname = "jenkins-httpd"
  config.vm.network "private_network", ip: "192.168.56.111"
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
    vb.cpus = 2
  end
  config.ssh.insert_key = false
end
