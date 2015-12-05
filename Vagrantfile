Vagrant.configure(2) do |config|
  config.vm.box = "debian/jessie64"
  config.vm.network "forwarded_port", guest: 4000, host: 4000
  config.vm.provision "shell", inline: <<-SHELL
	sudo apt-get update
	sudo apt-get -y install build-essential git ruby ruby-dev nodejs
	sudo gem install github-pages
	cd /vagrant
	jekyll serve --detach
  SHELL
end
