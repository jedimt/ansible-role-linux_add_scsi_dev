Ansible Role: Linux add SCSI dev
=========

Add mount new volume to Linux host.

Requirements
------------

Hosts require the `sg3_utils` package to provide the `rescan-scsi-bus.sh` script.

Role Variables
--------------

    # Set if we should create a filesystem (true) or leave as raw device (false)
    create_fs: true

    # Filesystem type - required if create_fs is true (ext4|xfs)
    fstype: xfs

Dependencies
------------

This role requires that the `jedimt.nebulon_manage_volumes` role be executed to provision volumes from Nebulon to use.

Example Playbook
----------------

    # ===========================================================================
    # Create Nebulon Volumes
    # ===========================================================================

    - name: Create Nebulon Volumes
      hosts: localhost
      connection: local
      tags: play_neb_vols
      gather_facts: false

      # module_defaults requires nebulon.nebulon_on version 1.2.1 or later
      module_defaults:
        group/nebulon.nebulon_on.nebulon:
          neb_username: "{{ vault_neb_username }}"
          neb_password: "{{ vault_neb_password }}"

      vars_files:
        # Ansible vault with all required passwords
        - "../../credentials.yml"
        # Ansible vault with server and SPU serials
        - "../../serials.yml"

      roles:
        - { role: jedimt.nebulon_manage_volumes }

    # ===========================================================================
    # Mount new volumes to Linux hosts
    # ===========================================================================

    - name: Mount new volumes to hosts
      hosts: k8s_nodes
      tags: play_scsi_dev
      gather_facts: true

      roles:
        - { role: jeditmt.linux_add_scsi_dev,
            # Create a filesystem (true) or leave as raw device (false)
            create_fs: true

            # Filesystem type - required if create_fs is true (ext4|xfs)
            fstype: xfs
        }

License
-------

MIT

Author Information
------------------

Aaron Patten
aaronpatten@gmail.com
