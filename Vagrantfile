# Ansible don't expect a role in working directory, to fix this set roles_path
# to parent directory. The working directory name must be the same as the
# name of the role!
ENV['ANSIBLE_ROLES_PATH'] = ".."

Vagrant.configure(2) do |config|
	config.vm.box = "martin-v/debian-jessie-libvirt"
	config.vm.hostname = "letsencrypttest"
	config.vm.provision "shell",
		inline: "apt-get -y install nginx-light"
	config.vm.provision :ansible do |ansible|
		ansible.playbook = "tests/test.yml"
		ansible.verbose = "v"
		ansible.sudo = true
	end
end
