Vagrant.configure(2) do |config|
  config.vm.box = "centos/7"
  config.vm.provision "docker" do |d|
    d.pull_images "quay.io/coreos/clair:latest"
  end
  config.vm.provision "shell",
    inline: "if [ -d ~vagrant/sync ] ; then chcon -Rt svirt_sandbox_file_t ~vagrant/sync/config ; else echo 'no sync folder present'; fi"
end
