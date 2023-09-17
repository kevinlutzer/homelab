# Homelab

Documentation for Ansible Scripts, Kubernetes Manifest and other files needed to run my homelab.

# Setup Of A Cluster Master or Worker Node From Scratch

1. Install the Raspberry Pi 64 bit Lite OS onto an SD Card. Make sure you enable ssh-key only authentication
1. Boot the pi with the created sd card, and ssh into the default user.
1. Modify the `/etc/dhcpcd.conf` to set a static IP address for the node. Add this IP to the `k3snodes` group in the ansible hosts file. Current with how you have it setup on your mac, that file is located `.ansible/hosts.ini`. Example config for the hosts.ini file can be found in the next section.
1. Copy the ssh key the raspberry pi image burner tool adds to `~/.ssh/authorized_keys` to the root ssh key directory. Run: `sudo mkdir -p /root/.ssh && sudo cp ~/.ssh/authorized_keys /root/.ssh`. This so that ansible can run commands with sudo priveledges on the raspberry pi. 
1. Reboot the raspberry pi.
1. Run the `k3s/playbooks/setupraspberrypi.yml` playbook. That will update the os, the cgroup memory config, add config so that the ican USB to SATA convert mounts SSDs as "storage devices", and other k3s specific requirements
1. Copy the kubectl config file from the k3s master to your working machine: `scp root@<MASTER_IP>:/etc/rancher/k3s/k3s.yaml ~/.kube/config`. 
1. Modify `~/.kube/config` to specify the raspberry pi master ip as the server ip
1. To setup a master node run the `k3s/playbookssetupmaster.yml` playbook. To create a worker node, run the `k3s/playbookssetupnodes.yml`. Note that for this playbook you will need to specify the server token from the master node. This can be fetched by running `ssh node@<MASTER IP> "sudo cat /var/lib/rancher/k3s/server/node-token"`

## Example hosts.ini 

```ini
[k3smaster]
192.168.4.200 var_disk=sda var_blkid=asd

[k3snodes]
192.168.4.201 var_disk=sda var_blkid=asd


[cube:children]
k3smaster
k3snodes

[k3smaster:vars]
ansible_user=root

[k3snodes:vars]
ansible_user=root
```

## Setup of a Raspberry Pi with External Storage

This is needed for the Longhorn manager. Any node that you want to have persistent storage will need to follow these steps.

1. Verify what path the SSD is attached to by running `ansible cube -b -m shell -a "lsblk -f"`. In most cases this will be `/dev/sda`.
1. Specify that path for the node's ip in the hosts.ini file as the variable `var_disk`.
1. Wipe the SSD disk by running `ansible <HOST IP OR GROUP> -b -m shell -a "wipefs -a /dev/{{ var_disk }}"`
1. Format SSD as an `ext4` partition with `ansible <HOST IP OR GROUP> -b -m filesystem -a "fstype=ext4 dev=/dev/{{ var_disk }}"` 
1. Get the block id of the partition with `ansible <HOST IP OR GROUP> -b -m shell -a "blkid -s UUID -o value /dev/{{ var_disk }}"`, add that id to the host.ini file as the variable `var_blkid`
1. Mount the block with `ansible <HOST IP OR GROUP> -m ansible.posix.mount -a "path=/storage01 src=UUID={{ var_uuid }} fstype=ext4 state=mounted" -b`

Once the node has come online, The longhorn controller should pick this up and setup the needed containers for that node. 

# Adding a Service

All k3s services are defined in `k3s/service`. The name of the directory containing the manifests for the service <strong>needs</strong> to be the same name as the samespace for that service. Example structure of a service's directory: 

ntfy
├── ns.yml 
├── deployment.yml
├── certificate.yml
├── issuer.yml
├── service.yml
├── ingress.yml

For services that require ssl/tls termination on the ingress, you must create a cloudflare api key secret like `kubectl create secret generic -n <NAMESPACE> cloudflare-api-key-secret --from-literal=api-key=<VALUE>`. 
This allows `dns-01` challenges to happen within cert-manager. The namespace must be created first, followed by the issuer, certificate, deployment, service and ingress in that order.

# Longhorn Notes:

Longhorn was instantiated with:

``` bash
helm repo add longhorn https://charts.longhorn.io
helm repo update
helm install longhorn longhorn/longhorn --namespace longhorn-system --create-namespace --set defaultSettings.defaultDataPath="/storage01"
```

The rest of longhorn (service/certs) can be managed with the manifests in the longhorn directory.