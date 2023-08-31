
## PASM VAULT ANSIBLE PLUGIN

### Introduction 
   
PASM vault ANSIBLE plugin to fetch secrets from PASM vault to ANSIBLE playbook using lookup plugin.This document will cover configuration and steps for the same.

### Prerequisites

1:- From vault side:
    
    a:- Access policy with required permissions for the user.

    b:- CACERT of Key-Control vault

    c:- API URL of vault and credentials.

2:- Ansible must be installed.

3:- Python 3 must be installed and 'requests' module must be installed using 'pip'. Additionally, if the output is required in yaml format, then 'pyyaml' must be installed using 'pip'.

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
Note: Users have the option to specify the output format as either YAML or JSON. By default, the format will be YAML.

Example :-To specify the format, the user must enter the last variable as either "json" or "yaml".
"{{ lookup('pasm', '<box-name1>:<secret-name1>','<box-name2>:<secret-name2>',.....,'yaml') }}"
"{{ lookup('pasm', '<box-name1>:<secret-name1>','<box-name2>:<secret-name2>',.....,'json') }}"
```

```
Example call to pasm lookup : -
var="{{ lookup('pasm', '<box-name1>:<secret-name1>','<box-name2>:<secret-name2>',.....,'<yaml or json>') }}"
```
```
Ansible Playbook Example:-
---
- name: PASM vault lookup plugin
  gather_facts: false
  hosts: all
  vars:
    pasm: "{{ lookup('pasm', 'ansible-box','file')}}"
  tasks:
    - name: Display pasm secret
      debug:
        msg: "{{pasm}}"
```
```
Example:- 

var="{{ lookup('pasm', '<box-name>:<Key-value type secret>','json') }}"

output :-
  var : {
  "box-name": {
    "secret-name": {
      "username": "password"
    }
  }
}

Example:- 

var="{{ lookup('pasm', '<box-name>:<Key-value type secret>','yaml') }}"

output :-
  var :
box-name:
  secret-name:
    username: password


Example:- 

var="{{ lookup('pasm', '<box-name1>:<Key-value type secret1>',''<box-name2>:<Key-value type secret2>'.....','json') }}"

output :-
  var : {
  "box-name1": {
    "secret-name1": {
      "username": "password"
    }
  },
  "box-name2": {
    "secret-name2": {
      "username": "password"
    }
  }
}

Example:- 

var="{{ lookup('pasm', '<box-name1>:<Key-value type secret1>',''<box-name2>:<Key-value type secret2>'.....','yaml') }}"

output :-
  var : 
box-name1:
  secret-name1:
    username: password
box-name2:
  secret-name2:
    username: password

Example:-
  var="{{ lookup('pasm', '<box-name>:<password/text/file type secret>','json') }}"

output :-
  var : 
  {
  "box-name": {
    "secret-name": "secret-value"
  }
}
Example:-
  var="{{ lookup('pasm', '<box-name>:<password/text/file type secret>','yaml') }}"

output :-
  var : 
box-name:
  secret-name: secret-value

Example:- var="{{ lookup('pasm', '<box-name>:<file type secret>','json') }}"  

output :-
  var : {
  "box-name": {
    "secret-name": "secret-decoded value of file"
  }
}
Example:- var="{{ lookup('pasm', '<box-name>:<file type secret>','yaml') }}"  

output :-
 var :
box-name:
  secret-name: secret-decoded value of file

Example:- var="{{ lookup('pasm', '<box-name1>:<password/text/file type secret>','<box-name1>:<Key-value type secret1>','<box-name2>:<password/text/file type secret>','<box-name2>:<Key-value type secret1>','json') }}"   

output :-
  var :
  {
  "box-name1": {
    "secret-name1": {
      "username": "password"
    },
    "secret-name2": "secret-value"
  },
  "box-name2": {
    "secret-name1": {
      "username1": "password1"
    },
    "secret-name2": "secret-value"
  }
}

Example:- var="{{ lookup('pasm', '<box-name1>:<password/text/file type secret>','<box-name1>:<Key-value type secret1>','<box-name2>:<password/text/file type secret>','<box-name2>:<Key-value type secret1>','yaml') }}"   

output :-
  var :
box-name1:
  secret-name1:
    username: password
  secret-name2: secret-value
box-name2:
  secret-name1:
    username1: password1
  secret-name2: secret-value

```
## Reference

[Look up plugin ansible](https://docs.ansible.com/ansible/latest/plugins/lookup.html#enabling-lookup-plugins)
