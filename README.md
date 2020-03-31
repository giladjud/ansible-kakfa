Kafka Cluster Role and Playbook 
=========

This role sets up Kafka on a cluster of servers 

AWS Prerequisites
------------

### Amazon Linux servers

#### Internet access from the Kafka servers

### Open ports 

* between the three servers: Ports: 2181, 2888, 3888, 9092
* from Clients to kafka servers: Port: 9092
* from Ansible server to Kafka serves: Port: 22

### Disks and File Systems

In addition to the root file system have two more:

| Disk | Size | mount point |
|---|---|---|
| ZooKeeper | 10Gb	| /zookeeper |
| Kafka | 240Gb | /kafka | 

OS Level Prerequisites
------------

### SSH Access to the servers

Add ansible inventory ssh public key to user `ec2-user`

### Create file systems

Example commands:

```
fdisk /dev/xvdb
  n #new partition default values
  w # write and quit
mkfs.ext4 /dev/xvdb1
```
To create the mounts:

Example lines in `/etc/fstab`

```
/dev/xvdd1   /zookeeper  ext4    defaults        0   0
/dev/xvdb1   /kafka      ext4    defaults        0   0
```

```
mkdir /kafka /zookeeper
mount -a
```

### Add users:

```
sudo groupadd -g 502 kafkagroup
sudo useradd -m -d /home/kafkauser -s /bin/bash -G 502 -u 502 kafkauser
```

### Update ownership of mounted filesystems

```
sudo chown kafkauser:kafkagroup /kafka
sudo chown kafkauser:kafkagroup /zookeeper
```

Ansible Prerequisites
------------

### `hosts`
Update the `inventories/dev/hosts` to fit your host names:

```
[kafka_servers]
kafka1.dev.example.com zk_id=1
kafka2.dev.example.com zk_id=2
kafka3.dev.example.com zk_id=3
```

### Encrypted private SSH key

Add the private Key to the vault file - don't forget to encrypt it 

See example in file `inventories/dev/group_vars/all/vault`

Ansible Playbooks
------------

### Setup terminal to run `ansible-playbook`

Assumptions:
- The `vault` file is encrypted 
- Vault password set in a file - e.g. `$HOME/.vault/dev`

Set Environment Variables:

```
export ANSIBLE_VAULT_PASSWORD_FILE=$HOME/.vault/dev
export ANSIBLE_INVENTORY=inventories/dev
```

### Get private key for environment

```
ansible-playbook playbooks/local-get-private-key.yml
```

This will place the Environment's private key in `ssh/id_rsa_ansible`

Note the `.gitignore`:

```
*.retry
.idea/
*.log
ssh/
```

### Run the Kafka Cluster ansible playbook

```
ansible-playbook playbooks/kafka-cluser.yml
```