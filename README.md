# Deployment pacakage was created basing on the latest CentOS 7 full image, and was tested on it.

# In order to start deploying virtual box machines (with vagrant), and later on starting ansible automation - few of the steps will be required to install additional repositories, binaries, to satisfy missing dependencies, or to change default configuration files. 

# These are listed below, as shell commands, which must be ran over privillaged root user, or with sudo. This probably could be scripted over local shell script, or with little help from the kickstarter - since at this point we do not have ansible available yet.

wget http://download.virtualbox.org/virtualbox/rpm/rhel/virtualbox.repo -P /etc/yum.repos.d/

wget https://releases.hashicorp.com/vagrant/2.2.9/vagrant_2.2.9_x86_64.rpm -P ~/

yum -y install gcc dkms make qt libgomp patch kernel-headers kernel-devel binutils glibc-headers glibc-devel font-forge epel-release ansible VirtualBox-5.1 git

/sbin/rcvboxdrv setup

yum localinstall -y vagrant_*

vagrant plugin install vagrant-dns

vagrant dns --start

echo "   StrictHostKeyChecking no" >> /etc/ssh/ssh_config
echo "   UserKnownHostsFile=/dev/null" >> /etc/ssh/ssh_config

# Another step is to clone our git repository directly the hypervisor host file-system.

git clone https://github.com/asilewicz/gitlab.kernel.tx.git

# At this point we have everything, and we are all ready to start the automation tasks...

vagrant up

# Unfortunatelly there is no elegant way of introducing ansible execution straight to the Vagrantfile with multi VMs deployment (or at least not one that I would be aware of at this moment). 
# Possible resolutions over Vagrant triggers, loops, if conditions - didn't prove to be really useful. Another possible soulution could be by creating small shell wrapper to execute first "vagrant up", and after that when receiving successful exit code, to start ansible automation... for now this would be the second command that we will have to use manually...

ansible-playbook main.yml -i hosts

# After successful deployment of the virtualbox VMs, and completing ansible tasks, the GitLab server URL will be accessible from the hypervisor ip-address, on port 8443, and also on the https://gitlab-server.kernel.tx - both of which might be directly reached from web-browser. Here the 'root' user password will have to be also created during the first login.

####

# Final remarks :

# GitLab Runner will be available for picking up the jobs from GitLab Server (shell exector will be used - as it is default). In the process GitLab Runner token was extracted from gitlab-rails - this is of course only one of the approches to complete the runner registration task. Another solution could be injecting our own token into /etc/gitlab/gitlab.rb before initializing the database, and also we could predefine initial root password during that step (ref. https://docs.gitlab.com/omnibus/settings/database.html#seed-the-database-fresh-installs-only).

# As there is of course a lot of 'room for improvment', i.e. by splitting this playbook into ansible roles - making it more universal for VMs deployment; by propagating dns records directly into the dns server (dehydrated, lexicon), or by using trusted certificates/internal self-signed certificate chain  - but it would became much more project speciffied with these changes. 

# Thank you for reading!
