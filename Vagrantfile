Vagrant.configure('2') do |config|
  config.vm.define 'mailserver' do |mailserver|
    mailserver.vm.box = 'ubuntu/jammy64'
    mailserver.vm.hostname = 'mailserver'
    mailserver.vm.network 'private_network', ip: '192.168.56.252'
    mailserver.vm.provision 'file', source: '~/.ssh/id_rsa', destination: '~/.ssh/id_rsa'
    # mailserver.vm.network 'forwarded_port', guest: 80, host: 8080
    mailserver.vm.provision 'shell', inline: 'bash <(curl -sL https://raw.githubusercontent.com/krlex/docker-installation/master/install.sh)'

    mailserver.vm.provider :virtualbox do |v|
      v.customize ['modifyvm', :id, '--natdnshostresolver1', 'on']
      v.customize ['modifyvm', :id, '--cpus', 4]
      v.customize ['modifyvm', :id, '--memory', 4096]
      v.customize ['modifyvm', :id, '--name', 'mailserver']
    end
  end
end
