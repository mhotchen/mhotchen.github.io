Vagrant.configure(2) do |config|
  config.vm.box = "debian/jessie64"
  config.vm.network "forwarded_port", guest: 4000, host: 4000
  config.vm.provision "shell", inline: <<-SHELL
	sudo apt-get update
	sudo apt-get -y install build-essential git ruby ruby-dev nodejs
	sudo gem install github-pages
	cd /vagrant
	while [ "1" == "1" ]; do
      jekyll serve --detach --drafts
      sleep 2
      pgrep -f jekyll | xargs kill -9
    done
  SHELL
end
