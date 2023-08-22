# PASM-Vault-Ansible-Plugin

### Introduction

PASM vault ANSIBLE plugin to fetch secrets from PASM vault to ANSIBLE playbook using lookup plugin.This document will cover configuration and steps for the same.

### Installation

Step 1:- Download or copy the lookup plugin onto the same machine where ANSIBLE is installed

```
Note: Please refrain from altering the file names.
```

Step 2:- To activate a PASM lookup, you have two options. First, you can place it in a lookup_plugins directory that should be located next to your play. Alternatively, you can place it inside the plugins/lookup/ directory.
```
EXAMPLE:- IN centos location will be /usr/shares/ansible/plugins/lookup
```

### configuration

Before running the plugin, there are some configurations that need to be completed.

Setting up the necessary environmental variables is essential in the same machine where ansible and playbook is present.

1 `PASM_URL` :- API url of PASM VAULT

2 `PASM_USERNAME` :- PASM VAULT username

3 `PASM_PASSWORD` :- PASM VAULT password for the same username

4 `PASM_CACERT_PATH`:- The CA certificate path of PASM VAULT (should be located on the same machine where Ansible and the playbook are present).


```
Command to run to set up the above environment variables.

export PASM_URL=<API-url-PASM-VAULT>
export PASM_USERNAME=<PASM-VAULT-USERNAME>
export PASM_PASSWORD=<PASM-VAULT-PASSWORD>
export PASM_CACERT_PATH=<PASM-VAULT-CA-CERT-PATH>

```

## Usage/Examples

```
var="{{ lookup('pasm', '<box-name>', '<secret-name>') }}"
```

## Reference

[Look up plugin ansible](https://docs.ansible.com/ansible/latest/plugins/lookup.html#enabling-lookup-plugins)
