# RancherOS

Download `docker-machine` and add it to the current shell `$PATH`:

```bash
wget -qO docker-machine https://github.com/docker/machine/releases/download/v0.16.1/docker-machine-Linux-x86_64
chmod +x docker-machine
wget -qO docker-machine-driver-kvm https://github.com/dhiltgen/docker-machine-kvm/releases/download/v0.10.0/docker-machine-driver-kvm-ubuntu16.04
chmod +x docker-machine-driver-kvm
export PATH="$PATH:$PWD"
```

Start the libvirt network:

```bash
virsh net-list --all
virsh net-dumpxml vagrant-libvirt
virsh net-start vagrant-libvirt
```

If you do not have the `vagrant-libvirt` network, you could create it with:

```bash
virsh net-define <(
cat <<EOF
<network connections='1' ipv6='yes'>
  <name>vagrant-libvirt</name>
  <uuid>4e871bdd-3466-44c7-9c0c-e4d9f6e712dc</uuid>
  <forward mode='nat'>
    <nat>
      <port start='1024' end='65535'/>
    </nat>
  </forward>
  <bridge name='virbr0' stp='on' delay='0'/>
  <mac address='52:54:00:4d:20:fd'/>
  <ip address='192.168.121.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.121.1' end='192.168.121.254'/>
    </dhcp>
  </ip>
</network>
EOF
)
```

Create the libvirt VM:

```bash
docker-machine create -d kvm --help
docker-machine create -d kvm \
    --kvm-boot2docker-url https://releases.rancher.com/os/latest/rancheros.iso \
    --kvm-cpu-count 2 \
    --kvm-memory 2048 \
    --kvm-network vagrant-libvirt \
    ros
```

You could, alternatively, create the VirtualBox VM:

```bash
docker-machine create -d virtualbox --help
docker-machine create -d virtualbox \
    --virtualbox-boot2docker-url https://releases.rancher.com/os/latest/rancheros.iso \
    --virtualbox-cpu-count 2 \
    --virtualbox-memory 2048 \
    ros
```

Dump the docker environment variables:

```bash
docker-machine env ros
```

Enter the VM and execute some informational commands:

```bash
docker-machine ssh ros
sudo su -l
ros os list
ros service list
#ros console switch alpine
exit
```

Under libvirt, also try the qemu commands:

```bash
docker-machine ssh ros sudo ros service enable qemu-guest-agent
virsh qemu-agent-command ros '{"execute":"guest-ping"}'
virsh qemu-agent-command ros '{"execute":"guest-info"}'

# TODO figure out how to fix the error:
#       error: argument unsupported: QEMU guest agent is not configured
#      it seems docker-machine-driver-kvm does not properly configure the VM.
```

Finally, destroy/remove the VM:

```bash
docker-machine rm ros -y
```
