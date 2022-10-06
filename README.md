Ansible role: OpenNebula Host
=============================

Deploy an OpenNebula virtualisation host and add it to an existing cluster.

Role Variables
--------------

```
ENTRY POINT: main - Deploy an OpenNebula host

        Deploy an OpenNebula virtualisation host and add it to an
        existing cluster.

OPTIONS (= is mandatory):

= one_api_password
        Password for API authentication

        type: str

- one_api_url
        URL for the OpenNebula API (relative to the frontend server)
        [Default: http://127.0.0.1:2633/RPC2]
        type: str

- one_api_username
        Username for API authentication
        [Default: ansible]
        type: str

- one_apt_key_fingerprint
        Fingerprint for the OpenNebula apt key
        [Default: (null)]
        type: str

- one_datastore_mounts
        Disk mounts for individual datastores
        [Default: (null)]
        elements: dict
        type: list

        OPTIONS:

        = disk_uuid
            UUID of the disk being mounted

            type: str

        = id
            ID of the datastore to mount to

            type: int

- one_datastores_disk_uuid
        UUID of the disk to mount to the datastores root directory
        [Default: (null)]
        type: str

- one_default_vdc
        Name of the default virtual datacenter
        [Default: default]
        type: str

= one_frontend
        Hostname of the frontend server

        type: str

= one_frontend_key
        Contents of the frontend host key

        type: str

= one_host
        Properties for the host

        type: dict

        OPTIONS:

        = cluster_id
            The cluster the host belongs to

            type: int

        = key
            Contents of the SSH host key

            type: str

        = network_interfaces
            The network interfaces connected to virtual bridges

            elements: dict
            type: list

            OPTIONS:

            = bridge
                Name of the virtual bridge to create

                type: str

            = extport
                Name of the external interface

                type: str

        - private_net
            Configuration for the private virtual network used by
            normal users
            [Default: (null)]
            type: dict

            OPTIONS:

            = bridge
                Name of the bridge connecting the internal and
                external networks

                type: str

            = external
                Configuration for the external network

                type: dict

                OPTIONS:

                = bridge
                    Name of the bridge

                    type: str

                = gateway
                    Default gateway

                    type: str

                - ip
                    IP address of the bridge interface
                    [Default: (null)]
                    type: str

                = network
                    IP subnet of the network

                    type: str

            = internal
                Configuration for the internal network

                type: dict

                OPTIONS:

                = bridge
                    Name of the bridge

                    type: str

                = ip
                    IP address of the bridge interface

                    type: str

                = network
                    IP subnet of the network

                    type: str

        - reserved_cpu
            Amount of CPU reserved given in number of logical CPUs
            multiplied by 100 (1 logical CPU = 100), use a negative
            number for oversubscription which can be calculated by:
            -100 * num_logical_cpus * (oversubscription_factor - 1)
            [Default: 0]
            type: int

        - reserved_mem
            Amount of memory reserved in KB
            [Default: 0]
            type: int

        - type
            The host type
            [Default: (null)]
            type: str

        - vms_per_thread
            Number of VMs that can be assigned to one CPU thread when
            pinned
            [Default: 1]
            type: int

        - vms_pinned
            If true, the host will support CPU pinning
            [Default: False]
            type: bool

- one_host_datastores
        Names of datastores the host contains
        [Default: ['Default', 'Images', 'Files']]
        elements: str
        type: list

= one_host_type
        The default host type

        type: str

- one_install_version
        Version of OpenNebula to install
        [Default: 6.4.0]
        type: str

- one_lvm_datastores
        LVM datastores for the host
        [Default: (null)]
        elements: dict
        type: list

        OPTIONS:

        - mkfs_ext_opts
            Options for ext filesystems created in the datastore,
            example for RAID drives: -b <block_size> -E
            stride=<chunk_size / block_size>,stripe-width=<stride *
            num_data_bearing_drives>
            [Default: (null)]
            type: str

        - mkfs_xfs_opts
            Options for xfs filesystems created in the datastore,
            example for RAID drives: -b size=<block_size> -d
            su=<chunk_size>,sw=<num_data_bearing_drives>
            [Default: (null)]
            type: str

        = tier
            Name of the storage tier

            type: str

        = vg
            Name of the volume group

            type: str

= one_ssh_pubkey
        Contents of the oneadmin ssh public key

        type: str

- one_version
        Version of the OpenNebula repository to track
        [Default: 6.4]
        type: str

- openvswitch_apt_key_fingerprint
        Fingerprint for the Open vSwitch apt key
        [Default: (null)]
        type: str
```

Installation
------------

This role can either be installed manually with the ansible-galaxy CLI tool:

    ansible-galaxy install git+https://github.com/wandansible/opennebula.host,main,opennebula.host
     
Or, by adding the following to `requirements.yml`:

    - name: wandansible.opennebula.host
      src: https://github.com/wandansible/opennebula.host

Roles listed in `requirements.yml` can be installed with the following ansible-galaxy command:

    ansible-galaxy install -r requirements.yml

Example Playbook
----------------

    - hosts: opennebula_host_servers
      roles:
         - role: wandansible.opennebula.host
           become: true
           vars:
             one_version: "5.12"

             one_frontend: "cloud.example.org"
             one_api_username: ansible
             one_api_password: CHANGEME
             one_frontend_key: "ssh-ed25519 AAAA..."
             one_ssh_pubkey: "ssh-ed25519 AAAA..."

             one_datastores_disk_uuid: aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee
             one_datastore_mounts:
               - id: 0
                 disk_uuid: vvvvvvvv-wwww-xxxx-yyyy-zzzzzzzzzzzz

             one_lvm_datastores:
               - tier: HDD
                 vg: hdd_raid60
                 mkfs_ext_opts: -b 4096 -E stride=128,stripe-width=1536
                 mkfs_xfs_opts: -b size=4096 -d su=512K,sw=12
               - tier: SSD
                 vg: ssd_raid6
                 mkfs_ext_opts: -b 4096 -E stride=128,stripe-width=768
                 mkfs_xfs_opts: -b size=4096 -d su=512K,sw=6

             one_host:
               network_interfaces:
                 - bridge: br-external
                   extport: eno1
               private_net:
                 internal:
                   bridge: br-private
                   network: 192.168.0.0/24
                   ip: 192.168.0.254/24
                 external:
                   bridge: br-external
                   network: 192.0.2.1/24
                   gateway: 192.0.2.254
                 bridge: br0
               cluster_id: 0
