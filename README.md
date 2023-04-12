Ansible Role: Linux add SCSI dev
=========

Add mount new volume to Linux host.

Requirements
------------

Hosts require the `sg3_utils` package to provide the `rescan-scsi-bus.sh` script.

Role Variables
--------------

Variables are defined in the defaults/main.yml file. In this example, the SPU serial numbers are stored in an ansible vault as well.

    # State for all volumes (present|absent)
    volume_state: present

    # Specify LUN export method (does not affect underlying volume)
    # (present|all) -> All nPod servers can access export
    # (host) -> Make export available to a single host. Requires host_uuid
    # (local) -> Make export available only to the local host that owns the volume
    # (absent) -> Remove the volume export
    export_type: local

    # nPod name to use when managing volumes/exports
    npod_name: "K8s_Lenovo"

    host_uuid:

    # List of volumes to manage (create or remove)
    volumes:
      - name: "server-10-local-kadalu"
        size: 1000000000000
        mirrored: true
        owner_spu_serial: "{{ server-10.spu-serial }}"
        backup_spu_serial: "{{ server-09-spu-serial }}"
        state: "{{ volume_state }}"

      - name: "server-11-local-kadalu"
        size: 1000000000000
        mirrored: true
        owner_spu_serial: "{{ server-11-spu-serial }}"
        backup_spu_serial: "{{server-12-spu-serial }}"
        state: "{{ volume_state }}"

      - name: "server-12-local-kadalu"
        size: 1000000000000
        mirrored: true
        owner_spu_serial: "{{server-12-spu-serial }}"
        backup_spu_serial: "{{ server-11-spu-serial }}"
        state: "{{ volume_state }}"

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
        - { role: jeditmt.linux_add_scsi_dev }

License
-------

MIT

Author Information
------------------

Aaron Patten
aaronpatten@gmail.com
