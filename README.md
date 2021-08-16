# 401_errors

Project to report 401 errors and demonstrate GitLab CI


It has been observed that using Ansible Vault encrypted strings are not properly handled by `ansible.netcommon.httpapi` resulting in `ansible.module_utils.connection.ConnectionError: HTTP Error 401: Unauthorized`.

To demonstrate, two test cases are configured in inventory files `inventory_sbx_n9kv_httpapi.yml` and `inventory_sbx_n9kv_cli.yml`.
Two hosts, `sandbox-nxos-1.cisco.com` and `131.226.217.151` are specified. They are the same host in the Cisco DevNet Sandbox.
One host entry uses a clear-text password, the other host uses an Ansible Vault encrypted string. The first task in the playbook displays the values of username and password of the Nexus switch.

The playbook `configure_device_interfaces.yml` is executed twice, once using the HTTPAPI connection method, the second using Network CLI.

Note the HTTP Error 401 is observed when using HTTPAPI, but not when using Network CLI. In both cases the clear-text password is successfully authenticated.

Using `ansible_connection: ansible.netcommon.httpapi`
-----------------------------------------------------

Execute the playbook using the argument shown:

```
ansible-playbook configure_device_interfaces.yml -i inventory_sbx_n9kv_httpapi.yml  -t cli --ask-vault-password

```

```shell
root@1ac64bb77719:/workspaces/401_errors/playbooks# ansible-playbook configure_device_interfaces.yml -i inventory_sbx_n9kv_httpapi.yml  -t cli --ask-vault-password
Vault password: 

PLAY [configure device interfaces] ****************************************************************************************************************************************

TASK [debug credentials] **************************************************************************************************************************************************
ok: [sandbox-nxos-1.cisco.com] => {
    "msg": "admin::Admin_1234!::ansible.netcommon.httpapi"
}
ok: [131.226.217.151] => {
    "msg": "admin::Admin_1234!::ansible.netcommon.httpapi"
}

TASK [Configure the switch similar to using CLI commands] *****************************************************************************************************************
An exception occurred during task execution. To see the full traceback, use -vvv. The error was: ansible.module_utils.connection.ConnectionError: HTTP Error 401: Unauthorized
fatal: [131.226.217.151]: FAILED! => {"changed": false, "module_stderr": "Traceback (most recent call last):\n  File \"/root/.ansible/tmp/ansible-local-69577f060trp9/ansible-tmp-1629126620.1739006-69911-227352806068616/AnsiballZ_nxos_config.py\", line 102, in <module>\n    _ansiballz_main()\n  File \"/root/.ansible/tmp/ansible-local-69577f060trp9/ansible-tmp-1629126620.1739006-69911-227352806068616/AnsiballZ_nxos_config.py\", line 94, in _ansiballz_main\n    invoke_module(zipped_mod, temp_path, ANSIBALLZ_PARAMS)\n  File \"/root/.ansible/tmp/ansible-local-69577f060trp9/ansible-tmp-1629126620.1739006-69911-227352806068616/AnsiballZ_nxos_config.py\", line 40, in invoke_module\n    runpy.run_module(mod_name='ansible_collections.cisco.nxos.plugins.modules.nxos_config', init_globals=None, run_name='__main__', alter_sys=True)\n  File \"/usr/local/lib/python3.8/runpy.py\", line 207, in run_module\n    return _run_module_code(code, init_globals, run_name, mod_spec)\n  File \"/usr/local/lib/python3.8/runpy.py\", line 97, in _run_module_code\n    _run_code(code, mod_globals, init_globals,\n  File \"/usr/local/lib/python3.8/runpy.py\", line 87, in _run_code\n    exec(code, run_globals)\n  File \"/tmp/ansible_cisco.nxos.nxos_config_payload_vwb25qjn/ansible_cisco.nxos.nxos_config_payload.zip/ansible_collections/cisco/nxos/plugins/modules/nxos_config.py\", line 606, in <module>\n  File \"/tmp/ansible_cisco.nxos.nxos_config_payload_vwb25qjn/ansible_cisco.nxos.nxos_config_payload.zip/ansible_collections/cisco/nxos/plugins/modules/nxos_config.py\", line 449, in main\n  File \"/tmp/ansible_cisco.nxos.nxos_config_payload_vwb25qjn/ansible_cisco.nxos.nxos_config_payload.zip/ansible_collections/cisco/nxos/plugins/module_utils/network/nxos/nxos.py\", line 129, in get_connection\n  File \"/tmp/ansible_cisco.nxos.nxos_config_payload_vwb25qjn/ansible_cisco.nxos.nxos_config_payload.zip/ansible/module_utils/connection.py\", line 195, in __rpc__\nansible.module_utils.connection.ConnectionError: HTTP Error 401: Unauthorized\n", "module_stdout": "", "msg": "MODULE FAILURE\nSee stdout/stderr for the exact error", "rc": 1}
...ignoring
[WARNING]: To ensure idempotency and correct diff the input configuration lines should be similar to how they appear if present in the running configuration on device
changed: [sandbox-nxos-1.cisco.com]

PLAY RECAP ****************************************************************************************************************************************************************
131.226.217.151            : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=1   
sandbox-nxos-1.cisco.com   : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

Using `ansible_connection: ansible.netcommon.network_cli`
---------------------------------------------------------

Execute the playbook using the arguments shown:

```
ansible-playbook configure_device_interfaces.yml -i inventory_sbx_n9kv_cli.yml  -t cli --ask-vault-password
```

```shell
root@1ac64bb77719:/workspaces/401_errors/playbooks# ansible-playbook configure_device_interfaces.yml -i inventory_sbx_n9kv_cli.yml  -t cli --ask-vault-password
Vault password: 

PLAY [configure device interfaces] ****************************************************************************************************************************************

TASK [debug credentials] **************************************************************************************************************************************************
ok: [sandbox-nxos-1.cisco.com] => {
    "msg": "admin::Admin_1234!::ansible.netcommon.network_cli"
}
ok: [131.226.217.151] => {
    "msg": "admin::Admin_1234!::ansible.netcommon.network_cli"
}

TASK [Configure the switch similar to using CLI commands] *****************************************************************************************************************
[WARNING]: To ensure idempotency and correct diff the input configuration lines should be similar to how they appear if present in the running configuration on device
changed: [131.226.217.151]
changed: [sandbox-nxos-1.cisco.com]

PLAY RECAP ****************************************************************************************************************************************************************
131.226.217.151            : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
sandbox-nxos-1.cisco.com   : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

Author
------

Joel W. King @joelwking