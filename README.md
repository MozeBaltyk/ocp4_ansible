## Ansible Playbook for OpenShift 4 and Jinja templates for Bastion/Helper Node setup

### Create VM with OpenTofu

```bash
MAC_ADDR=$(date +%s | md5sum | head -c 6 | sed -e 's/\([0-9A-Fa-f]\{2\}\)/\1:/g' -e 's/\(.*\):$/\1/' | sed -e 's/^/52:54:00:/';echo);

cd ./libvirt
tofu init;
tofu plan -out=terraform.tfplan \
  -var "hostname={{ hostname }}" \
  -var "mac_address=${MAC_ADDR}";
tofu apply "terraform.tfplan";

# Get the ip
sleep 60
tofu refresh && tofu output ips;

ssh -i .key.private ansible@<IP>

tofu destroy -auto-approve
```

Here is the directory structure of all the files in this directory:

```bash
$ tree
.
├── LICENSE
├── README.md
├── ansible.cfg
├── files
│   └── set-dns-serial.sh
├── handlers
│   └── main.yml
├── inventory
├── tasks
│   ├── configure_bind_dns.yml
│   ├── configure_dhcpd.yml
│   ├── configure_haproxy_lb.yml
│   └── configure_tftp_pxe.yml
├── templates
│   ├── default.j2
│   ├── dhcpd-uefi.conf.j2
│   ├── dhcpd.conf.j2
│   ├── haproxy.cfg.j2
│   ├── named.conf.j2
│   ├── pxe-bootstrap.j2
│   ├── pxe-master.j2
│   ├── pxe-worker.j2
│   ├── reverse.j2
│   └── zonefile.j2
└── vars
    └── main.yml

5 directories, 21 files
```

The `tasks` folder contain all the tasks to configure:
- Bind DNS server
- DHCP Server
- HAProxy Load balancer
- PXE / TFTP Server

### Setting variables
Modify `vars/main.yml` and set all required variables for configuring DNS, DHCP, HAProxy and TFTP/PXE boot.

```bash
vim vars/main.yml
```

### Running the tasks with Ansible

This tasks in this repository are not functional on their own. You need to follow our guide on [computingforgeeks.com](https://computingforgeeks.com) shared in the link below:

[How To Deploy OpenShift Container Platform on KVM](https://computingforgeeks.com/how-to-deploy-openshift-container-platform-on-kvm/)
