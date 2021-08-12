# 401_errors

Project to report 401 errors and demonstrate GitLab CI

Connection Method: ansible.netcommon.httpapi
--------------------------------------------

```
root@1ac64bb77719:/workspaces/401_errors/playbooks# ansible-playbook ./configure_device_interfaces.yml -i inventory_httpapi.yml --ask-vault-password
Vault password: 

PLAY [configure device interfaces] ************************************************************************************************

TASK [Configure Layer 2 interfaces] ***********************************************************************************************
ok: [chip-nexus-tr.sandbox.wwtatc.local] => (item=63)
ok: [chip-nexus-tr.sandbox.wwtatc.local] => (item=64)

TASK [Configure Layer 2 interfaces] ***********************************************************************************************
ok: [chip-nexus-tr.sandbox.wwtatc.local] => (item=63)
ok: [chip-nexus-tr.sandbox.wwtatc.local] => (item=64)

TASK [Configure interface as Layer 3] *********************************************************************************************
changed: [chip-nexus-tr.sandbox.wwtatc.local] => (item={'name': 'Ethernet1/60', 'description': 'UPLINK', 'address': '192.0.2.1/31'})

TASK [Specify IP address for interface] *******************************************************************************************
changed: [chip-nexus-tr.sandbox.wwtatc.local] => (item={'name': 'Ethernet1/60', 'description': 'UPLINK', 'address': '192.0.2.1/31'})

PLAY RECAP ************************************************************************************************************************
chip-nexus-tr.sandbox.wwtatc.local : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

Connection Method: ansible.netcommon.network_cli
------------------------------------------------

```
root@1ac64bb77719:/workspaces/401_errors/playbooks# ansible-playbook ./configure_device_interfaces.yml -i inventory_network_cli.yml --ask-vault-password
Vault password: 

PLAY [configure device interfaces] ************************************************************************************************

TASK [Configure Layer 2 interfaces] ***********************************************************************************************
ok: [chip-nexus-tr.sandbox.wwtatc.local] => (item=63)
ok: [chip-nexus-tr.sandbox.wwtatc.local] => (item=64)

TASK [Configure Layer 2 interfaces] ***********************************************************************************************
ok: [chip-nexus-tr.sandbox.wwtatc.local] => (item=63)
ok: [chip-nexus-tr.sandbox.wwtatc.local] => (item=64)

TASK [Configure interface as Layer 3] *********************************************************************************************
changed: [chip-nexus-tr.sandbox.wwtatc.local] => (item={'name': 'Ethernet1/60', 'description': 'UPLINK', 'address': '192.0.2.1/31'})

TASK [Specify IP address for interface] *******************************************************************************************
changed: [chip-nexus-tr.sandbox.wwtatc.local] => (item={'name': 'Ethernet1/60', 'description': 'UPLINK', 'address': '192.0.2.1/31'})

PLAY RECAP ************************************************************************************************************************
chip-nexus-tr.sandbox.wwtatc.local : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

Author
------

Joel W. King @joelwking