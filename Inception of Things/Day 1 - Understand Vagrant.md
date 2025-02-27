- It is a useful tool that sets up, in a very efficient way, a virtual environment using providers such as VirtualBox, qemu, or even Docker. We can frankly say that we are beginning the journey of Infrastrucuture as Code because we create and manage systems using CODE. LINES OF CODE.
  
  Even though I am not a professional (talking about having 99 years of experience) I've come across:

	-  Installing Linux distros iso image
	-  Setting up a machine with that ISO
	-  And install the necessary tools for my environment to run an application
	-  And at times take snapshots of my machine in case if I break something 
	  

That seems lie a few steps but it does take a while to set up. And sometimes I'd like to set up more than just one machine. That will take double my time. And I don't want to spend that much.

Vagrant can be useful when you're trying to set up a machine in minutes, and not just one, but multiple.

```ruby
API_VERSION = "2"

Vagrant.configure(API_VERSION) do |config|

  # Configuration of the first machine
  config.vm.define "kquetat-S" do |server|
    server.vm.box = "generic/alpine318"
    server.vm.hostname = "kquetat-S"
    server.vm.network "private_network", ip: "192.168.56.110"
    server.vm.provider "virtualbox" do |vb|
      vb.cpus = "1"
      vb.memory = "512"
    end
  end

   # Configuration of the second machine
  config.vm.define "kquetat-SW" do |worker|
    worker.vm.box = "generic/alpine318"
    worker.vm.hostname = "kquetat-SW"
    worker.vm.network "private_network", ip: "192.168.56.111"
    worker.vm.provider "virtualbox" do |vb|
      vb.cpus = "1"
      vb.memory = "512"
    end
  end

  # Configure SSH no pw
  config.ssh.insert_key = false
 
end
```
