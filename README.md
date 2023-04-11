# Change host file
```bash
ansible-playbook -i hosts ../update_hosts_file.yml --ask-become-pass
```

# Install zabbix server
ssh into database vm to update packages repository
```bash
sudo apt update
```
- Change postgres-client in role `zabbix_server` in `community.zabbix`, make sure postgres-client and database host 's postgres is same version (currently using postgres 14)
```bash
path: ~/.ansible/collections/ansible_collections/community/zabbix/roles/zabbix_server/tasks/Debian.yml
Change task install postgresql client package to
`postgresql-client-14`
```
- Change max_connections of postgres to 100 because timescaledb-tune change to 25
```bash
ansible-playbook -i hosts zabbix-server.yml
```

# Post-installation when move zabbix to another machine (new ip addresses)
- Remove old ip in /etc/hosts manually (ssh to machine) and use ansible to update hosts file
```bash
ansible-playbook -i ansible/hosts update_hosts_file.yml
```
- Use virt-manager through ssh to manage virtual machines remotely
```bash
virt-manager -c 'qemu+ssh://monitor@<host_vm_ip_address>/system?keyfile=/home/vpoat/.ssh/id_monitor'
```
- Add macvtap network interface for virtual machines, and update netplan config (in /etc/netplan/) as below:
```bash
network:
  ethernets:
    ens3:
      addresses:
      - 192.168.122.11/24
      dhcp4: false
      gateway4: 192.168.122.1
      match:
        name: ens3
      set-name: ens3
    ens9:
      addresses:
      - 192.168.12.248/24
      dhcp4: false
      gateway4: 192.168.12.1
      match:
        name: ens9
      nameservers:
        addresses:
        - 192.168.12.11
        - 192.168.12.12
      set-name: ens9
```
then run `sudo netplan apply` to apply change
- Change ip host granted for mysql account (zabbix-server):
  - ssh to zabbix-database vm
  - login mysql with root user:
  ```bash
  mysql -uroot -proot
  select * from information_schema.user_privileges; # show granted host ip for users
  ```
  - change ip granted for user `zabbix-server`, `192.168.122.11` is old granted ip and `192.168.12.248` is new ip of zabbix-server
  ```bash
  rename user `zabbix-server@192.168.122.11` to `zabbix-server@192.168.12.248`;
  ```
