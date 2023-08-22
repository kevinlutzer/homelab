# Setup Of Cluster Node From Scratch

1. Install the Raspberry Pi 64 bit Lite OS onto an SD Card. Make sure you enable ssh-key only authentication
1. Boot the pi with the created sd card, and ssh into the default user.
1. Modify the `/etc/dhcpcd.conf` to set a static IP address for the node. Add this IP to the `k3snodes` group in the ansible hosts file. Current with how you have it setup on your mac, that file is located `.ansible/hosts`.
1. Copy the ssh key the raspberry pi image burner tool adds to `~/.ssh/authorized_keys` to the root ssh key directory. Run: `sudo mkdir -p /root/.ssh && sudo cp ~/.ssh/authorized_keys /root/.ssh`. This is needed for the current ansible configuration.
1. Update the `/etc/ssh/sshd_config` file to set the `LogLevel INFO` and `PermitRootLogin yes`. 
1. Reboot the raspberry pi
1. Run the `playbooks/updatepackages.yml` playbook. This will do a dist-upgrade and update the package registry
1. Run the `playbooks/setupraspberrypi.yml` playbook. That will update the cgroup memory config, iptables and other k3s specific requirements
1. Copy the kubectl config file from the k3s master to your working machine: `scp root@master-ip:/etc/rancher/k3s/k3s.yaml ~/.kube/config`.
