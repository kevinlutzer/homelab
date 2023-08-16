# Setup Of Cluster Node From Scratch

1. Install the Raspberry Pi 64 bit Lite OS onto an SD Card. Make sure you enable ssh-key only authentication
2. Modify the `cmdline.txt` file to include `cgroup_memory=1 cgroup_enable=memory` at the end of the file. This is needed by k3s
3. `modify the bottom of the config.txt` file to include `arm_64bit=1`.
4. Boot the pi with the created sd card, and ssh into the default user.
5. Modify the `/etc/dhcpcd.conf` to set a static IP address for the node. Add this IP to the `k3snodes` group in the ansible hosts file. Current with how you have it setup on your mac, that file is located `.ansible/hosts`.
6. Copy the ssh key the raspberry pi image burner tool adds to `~/.ssh/authorized_keys` to the root ssh key directory. Run: `sudo mkdir -p /root/.ssh && sudo cp ~/.ssh/authorized_keys /root/.ssh`. This is needed for the current ansible configuration.
7. Update the `/etc/ssh/sshd_config` file to set the `LogLevel INFO` and `PermitRootLogin yes`. 
8. Reboot the raspberry pi