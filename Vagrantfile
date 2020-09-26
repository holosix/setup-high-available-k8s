$install_utilities = <<SCRIPT
#!/bin/bash
sudo yum install epel-release -y
sudo yum install nano -y
sudo yum install telnet -y
SCRIPT

$install_docker = <<SCRIPT
#!/bin/bash
sudo yum install -y yum-utils
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install docker-ce docker-ce-cli containerd.io -y
sudo systemctl enable --now docker
SCRIPT

$install_k8s = <<SCRIPT
#!/bin/bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
sudo mv kubernetes.repo /etc/yum.repos.d/kubernetes.repo
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
sudo systemctl enable --now kubelet
sudo systemctl daemon-reload
sudo systemctl restart kubelet
SCRIPT

$install_haproxy = <<SCRIPT
sudo yum install epel-release -y
sudo yum install haproxy -y
sudo rm -f /etc/haproxy/haproxy.cfg
cat <<EOF | sudo tee /etc/haproxy/haproxy.cfg
#---------------------------------------------------------------------
# Example configuration for a possible web application.  See the
# full configuration options online.
#
#   http://haproxy.1wt.eu/download/1.4/doc/configuration.txt
#
#---------------------------------------------------------------------

#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global
    # to have these messages end up in /var/log/haproxy.log you will
    # need to:
    #
    # 1) configure syslog to accept network log events.  This is done
    #    by adding the '-r' option to the SYSLOGD_OPTIONS in
    #    /etc/sysconfig/syslog
    #
    # 2) configure local2 events to go to the /var/log/haproxy.log
    #   file. A line like the following can be added to
    #   /etc/sysconfig/syslog
    #
    #    local2.*                       /var/log/haproxy.log
    #
    log         127.0.0.1 local2

    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon

    # turn on stats unix socket
    stats socket /var/lib/haproxy/stats

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000

#---------------------------------------------------------------------
# apiserver frontend which proxys to the masters
#---------------------------------------------------------------------
frontend apiserver
    bind *:8443
    mode tcp
    option tcplog
    default_backend apiserver
#---------------------------------------------------------------------
# round robin balancing for apiserver
#---------------------------------------------------------------------
backend apiserver
    option httpchk GET /healthz
    http-check expect status 200
    mode tcp
    option ssl-hello-chk
    balance     roundrobin
        server k8s-master01 192.168.0.101:6443 check
        server k8s-master02 192.168.0.102:6443 check
        server k8s-master03 192.168.0.103:6443 check
EOF

sudo systemctl enable --now haproxy
SCRIPT

Vagrant.configure("2") do |config|
        # Define base image
        config.vm.box = "bento/centos-7"
        # Manage /etc/hosts on host and VMs
        config.hostmanager.enabled = false
        config.hostmanager.manage_host = true
        config.hostmanager.include_offline = true
        config.hostmanager.ignore_private_ip = false

        config.vm.define :master01 do |master01|
        master01.vm.provider :virtualbox do |v|
                v.name = "k8s-master01"
                v.customize ["modifyvm", :id, "--memory", "4096", "--cpus", "2"]
        end
        master01.vm.network :private_network, ip: "192.168.0.101"
        master01.vm.hostname = "k8s-master01"
        master01.vm.provision :hostmanager
        master01.vm.provision :shell, :inline => $install_haproxy
        master01.vm.provision :file, :source => "calico.yaml", :destination => "calico.yaml"

        master01.vm.provision :shell, :inline => $install_utilities
        master01.vm.provision :shell, :inline => $install_docker
        master01.vm.provision :file, :source => "kubernetes.repo", :destination => "kubernetes.repo"
        master01.vm.provision :shell, :inline => $install_k8s
        end

        config.vm.define :master02 do |master02|
          master02.vm.provider :virtualbox do |v|
            v.name = "k8s-master02"
            v.customize ["modifyvm", :id, "--memory", "4096", "--cpus", "2"]
          end
          master02.vm.network :private_network, ip: "192.168.0.102"
          master02.vm.hostname = "k8s-master02"
          master02.vm.provision :hostmanager
          master02.vm.provision :shell, :inline => $install_utilities
          master02.vm.provision :shell, :inline => $install_docker
          master02.vm.provision :file, :source => "kubernetes.repo", :destination => "kubernetes.repo"
          master02.vm.provision :shell, :inline => $install_k8s
        end

        config.vm.define :node01 do |node01|
          node01.vm.provider :virtualbox do |v|
            v.name = "k8s-node01"
            v.customize ["modifyvm", :id, "--memory", "4096", "--cpus", "2"]
          end
          node01.vm.network :private_network, ip: "192.168.0.104"
          node01.vm.hostname = "k8s-node01"
          node01.vm.provision :hostmanager
          node01.vm.provision :shell, :inline => $install_utilities
          node01.vm.provision :shell, :inline => $install_docker
          node01.vm.provision :file, :source => "kubernetes.repo", :destination => "kubernetes.repo"
          node01.vm.provision :shell, :inline => $install_k8s
        end

end
