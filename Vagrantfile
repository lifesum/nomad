# -*- mode: ruby -*-
# vi: set ft=ruby :
#

LINUX_BASE_BOX = "bento/ubuntu-16.04"
FREEBSD_BASE_BOX = "jen20/FreeBSD-11.1-RELEASE"

Vagrant.configure(2) do |config|
	# Compilation and development boxes
	config.vm.define "linux", autostart: true, primary: true do |vmCfg|
		vmCfg.vm.box = LINUX_BASE_BOX
		vmCfg.vm.hostname = "linux"
		vmCfg = configureProviders vmCfg,
			cpus: suggestedCPUCores()

		vmCfg = configureLinuxProvisioners(vmCfg)

		vmCfg.vm.synced_folder '.',
			'/opt/gopath/src/github.com/hashicorp/nomad'

		vmCfg.vm.provision "shell",
			privileged: false,
			path: './scripts/vagrant-linux-unpriv-bootstrap.sh'
	end

	config.vm.define "freebsd", autostart: false, primary: false do |vmCfg|
		vmCfg.vm.box = FREEBSD_BASE_BOX
		vmCfg.vm.hostname = "freebsd"
		vmCfg = configureProviders vmCfg,
			cpus: suggestedCPUCores()

		vmCfg.vm.synced_folder '.',
			'/opt/gopath/src/github.com/hashicorp/nomad',
			type: "nfs",
			bsd__nfs_options: ['noatime']

		vmCfg.vm.provision "shell",
			privileged: true,
			path: './scripts/vagrant-freebsd-priv-config.sh'

		vmCfg.vm.provision "shell",
			privileged: false,
			path: './scripts/vagrant-freebsd-unpriv-bootstrap.sh'
	end	

	# Test Cluster (Linux)
	1.upto(3) do |n|
		serverName = "nomad-server%02d" % [n]
		clientName = "nomad-client%02d" % [n]
		serverIP = "10.199.0.%d" % [10 + n]
		clientIP = "10.199.0.%d" % [20 + n]

		config.vm.define serverName, autostart: false, primary: false do |vmCfg|
			vmCfg.vm.box = LINUX_BASE_BOX
			vmCfg.vm.hostname = serverName
			vmCfg = configureProviders(vmCfg)
			vmCfg = configureLinuxProvisioners(vmCfg)

			vmCfg.vm.provider "virtualbox" do |_|
				vmCfg.vm.network :private_network, ip: serverIP
			end
		
			vmCfg.vm.synced_folder '.',
				'/opt/gopath/src/github.com/hashicorp/nomad'
	
			vmCfg.vm.provision "shell",
				privileged: true,
				path: './scripts/vagrant-linux-priv-zeroconf.sh'
		end
		
		config.vm.define clientName, autostart: false, primary: false do |vmCfg|
			vmCfg.vm.box = LINUX_BASE_BOX
			vmCfg.vm.hostname = clientName
			vmCfg = configureProviders(vmCfg)
			vmCfg = configureLinuxProvisioners(vmCfg)
			
			vmCfg.vm.provider "virtualbox" do |_|
				vmCfg.vm.network :private_network, ip: clientIP
			end
			
			vmCfg.vm.synced_folder '.',
				'/opt/gopath/src/github.com/hashicorp/nomad'

			vmCfg.vm.provision "shell",
				privileged: true,
				path: './scripts/vagrant-linux-priv-zeroconf.sh'
		end
	end
end

def configureLinuxProvisioners(vmCfg)
	vmCfg.vm.provision "shell",
		privileged: true,
		inline: 'rm -f /home/vagrant/linux.iso'

	vmCfg.vm.provision "shell",
		privileged: true,
		path: './scripts/vagrant-linux-priv-go.sh'

	vmCfg.vm.provision "shell",
		privileged: true,
		path: './scripts/vagrant-linux-priv-config.sh'

	vmCfg.vm.provision "shell",
		privileged: true,
		path: './scripts/vagrant-linux-priv-consul.sh'

	vmCfg.vm.provision "shell",
		privileged: true,
		path: './scripts/vagrant-linux-priv-vault.sh'

	vmCfg.vm.provision "shell",
		privileged: true,
		path: './scripts/vagrant-linux-priv-rkt.sh'

	return vmCfg
end

<<<<<<< HEAD
# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

DEFAULT_CPU_COUNT = 2
$script = <<SCRIPT
GO_VERSION="1.9.0"

export DEBIAN_FRONTEND=noninteractive

sudo dpkg --add-architecture i386
sudo apt-get update

# Install base dependencies
sudo DEBIAN_FRONTEND=noninteractive apt-get install -y build-essential curl git-core mercurial bzr \
     libpcre3-dev pkg-config zip default-jre qemu silversearcher-ag \
     jq htop vim unzip tree                             \
     liblxc1 lxc-dev lxc-templates                      \
     gcc-5-aarch64-linux-gnu binutils-aarch64-linux-gnu \
     libc6-dev-i386 linux-libc-dev:i386                 \
     gcc-5-arm-linux-gnueabihf gcc-5-multilib-arm-linux-gnueabihf binutils-arm-linux-gnueabihf \
     gcc-mingw-w64 binutils-mingw-w64

# Setup go, for development of Nomad
SRCROOT="/opt/go"
SRCPATH="/opt/gopath"

# Get the ARCH
ARCH=`uname -m | sed 's|i686|386|' | sed 's|x86_64|amd64|'`

# Install Go
if [[ $(go version) == "go version go${GO_VERSION} linux/${ARCH}" ]]; then
    echo "Go ${GO_VERSION} ${ARCH} already installed; Skipping"
else
    cd /tmp
    wget -q https://storage.googleapis.com/golang/go${GO_VERSION}.linux-${ARCH}.tar.gz
    tar -xf go${GO_VERSION}.linux-${ARCH}.tar.gz
    sudo rm -rf $SRCROOT
    sudo mv go $SRCROOT
    sudo chmod 775 $SRCROOT
    sudo chown vagrant:vagrant $SRCROOT
fi

# Setup the GOPATH; even though the shared folder spec gives the working
# directory the right user/group, we need to set it properly on the
# parent path to allow subsequent "go get" commands to work.
sudo mkdir -p $SRCPATH
sudo chown -R vagrant:vagrant $SRCPATH 2>/dev/null || true
# ^^ silencing errors here because we expect this to fail for the shared folder

cat <<EOF >/tmp/gopath.sh
export GOPATH="$SRCPATH"
export GOROOT="$SRCROOT"
export PATH="$SRCROOT/bin:$SRCPATH/bin:\$PATH"
EOF
sudo mv /tmp/gopath.sh /etc/profile.d/gopath.sh
sudo chmod 0755 /etc/profile.d/gopath.sh
source /etc/profile.d/gopath.sh

# Install Docker
if [[ -f /etc/apt/sources.list.d/docker.list ]]; then
    echo "Docker repository already installed; Skipping"
else
    echo deb https://apt.dockerproject.org/repo ubuntu-`lsb_release -c | awk '{print $2}'` main | sudo tee /etc/apt/sources.list.d/docker.list
    sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
    sudo apt-get update
fi
sudo DEBIAN_FRONTEND=noninteractive apt-get install -y docker-engine

# Restart docker to make sure we get the latest version of the daemon if there is an upgrade
sudo service docker restart

# Make sure we can actually use docker as the vagrant user
sudo usermod -aG docker vagrant

# Setup Nomad for development
cd /opt/gopath/src/github.com/hashicorp/nomad && make bootstrap

# Install rkt, consul and vault
bash scripts/install_rkt.sh
bash scripts/install_rkt_vagrant.sh
bash scripts/install_consul.sh
bash scripts/install_vault.sh

# Set hostname's IP to made advertisement Just Work
sudo sed -i -e "s/.*nomad.*/$(ip route get 1 | awk '{print $NF;exit}') $(hostname)/" /etc/hosts

# CD into the nomad working directory when we login to the VM
grep "cd /opt/gopath/src/github.com/hashicorp/nomad" ~/.profile || echo "cd /opt/gopath/src/github.com/hashicorp/nomad" >> ~/.profile
SCRIPT

def configureVM(vmCfg, vmParams={
                  numCPUs: DEFAULT_CPU_COUNT,
                }
               )
  # When updating make sure to use a box that supports VMWare and VirtualBox
  vmCfg.vm.box = "bento/ubuntu-16.04" # 16.04 LTS

  vmCfg.vm.provision "shell", inline: $script, privileged: false
  vmCfg.vm.synced_folder '.', '/opt/gopath/src/github.com/hashicorp/nomad'

  # We're going to compile go and run a concurrent system, so give ourselves
  # some extra resources. Nomad will have trouble working correctly with <2
  # CPUs so we should use at least that many.
  cpus = vmParams.fetch(:numCPUs, DEFAULT_CPU_COUNT)
  memory = 2048

  vmCfg.vm.provider "parallels" do |p, o|
    p.memory = memory
    p.cpus = cpus
  end

  vmCfg.vm.provider "virtualbox" do |v|
    v.memory = memory
    v.cpus = cpus
  end

  ["vmware_fusion", "vmware_workstation"].each do |p|
    vmCfg.vm.provider p do |v|
      v.enable_vmrun_ip_lookup = false
      v.gui = false
      v.memory = memory
      v.cpus = cpus
    end
  end
  return vmCfg
=======
def configureProviders(vmCfg, cpus: "2", memory: "2048")
	vmCfg.vm.provider "virtualbox" do |v|
		v.memory = memory
		v.cpus = cpus
	end

	["vmware_fusion", "vmware_workstation"].each do |p|
		vmCfg.vm.provider p do |v|
			v.enable_vmrun_ip_lookup = false
			v.vmx["memsize"] = memory
			v.vmx["numvcpus"] = cpus
		end
	end

	vmCfg.vm.provider "virtualbox" do |v|
		v.memory = memory
		v.cpus = cpus
	end

	return vmCfg
>>>>>>> upstream/pr/3171
end

def suggestedCPUCores()
	case RbConfig::CONFIG['host_os']
	when /darwin/
		Integer(`sysctl -n hw.ncpu`) / 2
	when /linux/
		Integer(`cat /proc/cpuinfo | grep processor | wc -l`) / 2
	else
		2
	end
end
