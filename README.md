# 401_errors

Project to report 401 errors and demonstrate GitLab CI

ISSUE: https://github.com/ansible/ansible/issues/75503

It has been observed that using Ansible Vault encrypted strings are not properly handled by `ansible.netcommon.httpapi` resulting in `ansible.module_utils.connection.ConnectionError: HTTP Error 401: Unauthorized`.

To demonstrate, two test cases are configured in inventory files `inventory_sbx_n9kv_httpapi.yml` and `inventory_sbx_n9kv_cli.yml`.
Two hosts, `sandbox-nxos-1.cisco.com` and `131.226.217.151` are specified. They are the same host in the Cisco DevNet Sandbox.
One host entry uses a clear-text password, the other host uses an Ansible Vault encrypted string. The first task in the playbook displays the values of username and password of the Nexus switch.

The playbook `configure_device_interfaces.yml` is executed twice, once using the HTTPAPI connection method, the second using Network CLI.

Note the HTTP Error 401 is observed when using HTTPAPI, but not when using Network CLI. In both cases the clear-text password is successfully authenticated.

## Edit by @gmuloc

* The password for the vault variables is `password`
* It is possible that the sandbox will be misbehaving from nxapi point of view (
    Bad Gatewat 502) If so it is possible to restart the API using:

    ```
    conf t
    # Disable the feature
    no feature nxapi
    # Re enable the feature. it will enable it on port 443
    feature nxapi
    ```

* To observe with wireshark the payload sent, disable ssl in `
    playbooks/inventory_sbx_n9kv_httpapi.yml `(the playbook will
    fail but packet capture will show the issue)

    The packet sent to connect when using vault encrypted password looks like:

    ```
     POST /ins HTTP/1.1
     Accept-Encoding: identity
     Content-Length: 150
     Host: 131.226.217.151:80
     User-Agent: Python-urllib/3.10
     Content-Type: application/json
     Authorization: Basic
     YWRtaW46eydfX2Fuc2libGVfdmF1bHQnOiAnJEFOU0lCTEVfVkFVTFQ7MS4xO0FFUzI1NlxuMzczNTMxNjU2NjMyMzY2NDYxMzA2MjY0MzMzMTMyMzMzMDY2Mzk2NDM0MzUzMjMwMzMzMjM4MzIzODM0MzczODM2MzUzODM1MzYzNzYyNjRcbjM0NjQ2MTM4NjMzODMyNjEzNzM1NjI2NDM1MzY2MzM0NjM2MzMwMzgzNzMwNjY2MzBhMzIzNTMyNjM2NjM4NjEzNTMyMzQzMTMyMzUzMTMwXG4zMjM1MzY2MTMzMzYzMTMyNjMzMjYxMzkzMDY2MzA2MzM5NjMzNzMwMzU2MTY2MzgzNTYzNjQ2NDM2Mzg2MjYyMzQzMTMyNjMzMzYzMzc2NFxuNjMzOTM2MzQ2MjMxMzkzMzYxMGEzNjM4MzQzODY1MzMzNDMwMzYzMTY1NjM2NDMzMzYzNDMwMzY2MzY1NjYzMTY0MzMzNDM5NjEzMTYzNjNcbjY2MzZcbid9
     Connection: close

     ```

     The Authorization will be deciphered as:
     ```
     admin:{'__ansible_vault':
     '$ANSIBLE_VAULT;1.1;AES256\n37353165663236646130626433313233306639643435323033323832383437383635383536376264\n3464613863383261373562643536633463633038373066630a323532636638613532343132353130\n32353661333631326332613930663063396337303561663835636464363862623431326333633764\n6339363462313933610a363834386533343036316563643336343036636566316433343961316363\n6636\n'}%
     ```


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
