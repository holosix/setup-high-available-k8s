


# join master node
kubeadm join k8s-master01:8443 --token v5geh6.izc7drqsi0u7vqze     --discovery-token-ca-cert-hash sha256:27ae57dd1fe4f2414976b6c5f2cd7f53951472ed5c7f6e59310fd2b80b502fc9     --control-plane --certificate-key 714317e6802ff1ce7b3e722acae2e802c47abb549b44e55c87de55e1bb30eb58

# join worker node
kubeadm join k8s-master01:8443 --token v5geh6.izc7drqsi0u7vqze     --discovery-token-ca-cert-hash sha256:27ae57dd1fe4f2414976b6c5f2cd7f53951472ed5c7f6e59310fd2b80b502fc9

# init first master node
kubeadm init --control-plane-endpoint "k8s-master01:8443" --upload-certs --pod-network-cidr "10.0.0.0/16" --apiserver-advertise-address "192.168.0.101"














