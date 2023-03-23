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
      dnf install -y munge wget git gcc python36 python3-devel libtiff-devel libjpeg-devel openjpeg2-devel zlib-devel freetype-devel lcms2-devel libwebp-devel tcl-devel tk-devel harfbuzz-devel fribidi-devel libraqm-devel libimagequant-devel libxcb-devel python3-devel redhat-rpm-config python3-pillow python3-pillow-tk
      dnf group install -y  'Development Tools'
      echo 123456789123456781234567812345678 > /etc/munge/munge.key
      chown munge:munge /etc/munge/munge.key
      chmod 600 /etc/munge/munge.key
      systemctl enable --now munge
      pip3 install "dask[complete]"
    )
  end

  config.vm.define  "rm1" do |node|
    node.vm.network "private_network", ip: "192.168.50.10"
    node.vm.hostname = "rm1"
    node.vm.provision "shell" do |s|
      s.privileged = true,
      s.inline = %q(
        dnf install -y slurm-slurmctld wget git python36 python3-devel libtiff-devel libjpeg-devel openjpeg2-devel zlib-devel freetype-devel lcms2-devel libwebp-devel tcl-devel tk-devel harfbuzz-devel fribidi-devel libraqm-devel libimagequant-devel libxcb-devel python3-devel redhat-rpm-config python3-pillow python3-pillow-tk
        ln -sf /vagrant/slurm.conf /etc/slurm/slurm.conf
        systemctl restart slurmctld
        pip3 install "dask[complete]"
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
          dnf install -y slurm-slurmd wget git python36 python3-devel libtiff-devel libjpeg-devel openjpeg2-devel zlib-devel freetype-devel lcms2-devel libwebp-devel tcl-devel tk-devel harfbuzz-devel fribidi-devel libraqm-devel libimagequant-devel libxcb-devel python3-devel redhat-rpm-config python3-pillow python3-pillow-tk
          echo 'SLURMD_OPTIONS=--conf-server rm1' >> /etc/sysconfig/slurmd
          systemctl restart slurmd
          rm -rf /etc/slurm
          ln -sf /var/spool/slurm/d/conf-cache/ /etc/slurm
          pip3 install "dask[complete]"
        )
      end
    end
  end

end
