# -*- mode: ruby -*-
# vi: set ft=ruby :

hosts = %q(
127.0.0.1     localhost localhost.localdomain localhost4 localhost4.localdomain4
::1           localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.100.10 rm1
192.168.100.20 ex0
192.168.100.21 ex1
)

Vagrant.configure("2") do |config|

  config.vm.box = "centos/8"
  config.vm.synced_folder ".", "/vagrant", type: "rsync"

  config.vm.provision "shell" do |s|
    s.privileged = true,
    s.inline = %Q(
      systemctl disable --now firewalld
      setenforce Permissive
      echo "#{hosts}" > /etc/hosts
      cd /etc/yum.repos.d/
      sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
      sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*
      dnf install -y epel-release
      dnf config-manager --set-enabled powertools
      dnf install -y munge
      echo 123456789123456781234567812345678 > /etc/munge/munge.key
      chown munge:munge /etc/munge/munge.key
      chmod 600 /etc/munge/munge.key
      systemctl enable --now munge
    )
  end

  config.vm.define  "rm1" do |node|
    node.vm.network "private_network", ip: "192.168.50.10"
    node.vm.hostname = "rm1"
    node.vm.provision "shell" do |s|
      s.privileged = true,
      s.inline = %q(
        dnf install -y slurm-slurmctld
        ln -sf /vagrant/slurm.conf /etc/slurm/slurm.conf
        systemctl restart slurmctld
      )
    end
  end

  nodes = %w(ex0 ex1)
  (0..(nodes.length - 1)).each do |num|
    name = nodes[num]
    config.vm.define "#{name}" do |node|
      node.vm.hostname = name
      node.vm.network "private_network", ip: "192.168.50.#{20+num}"
      node.vm.provision "shell" do |s|
        s.privileged = true,
        s.inline = %q(
          dnf install -y slurm-slurmd
          echo 'SLURMD_OPTIONS=--conf-server rm1' >> /etc/sysconfig/slurmd
          systemctl restart slurmd
          rm -rf /etc/slurm
          ln -sf /var/spool/slurm/d/conf-cache/ /etc/slurm
        )
      end
    end
  end

end
