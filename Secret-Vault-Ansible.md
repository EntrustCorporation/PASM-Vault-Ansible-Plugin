# Fetch Secrets in Ansible Playbooks (PASM)

This guide shows how to fetch a secret from the PASM Vault inside an Ansible playbook using the `pasm` lookup plugin. It is written for users who are new to Ansible and the Vault. Follow the steps in order. Every step has a command, the expected result, and what to do if it fails.

## How it works

1. You copy the `pasm.py` lookup plugin to the machine that runs Ansible.
2. You set four environment variables that tell the plugin where the Vault is and how to log in.
3. A playbook calls `lookup('pasm', '<BOX>:<SECRET>')`.
4. The plugin logs in to the Vault, checks out the secret and returns it to the playbook.

The Vault only releases the secret if an **access policy** (Vault User Role) allows your Vault user to read it.

## Placeholders

Replace these values with your own everywhere they appear.

| Placeholder | Meaning |
|---|---|
| `<VAULT_IP>` | IP or host name of the Vault |
| `<VAULT_UUID>` | UUID of the Vault (used in the login URL) |
| `<VAULT_USER>` | Vault user name (for example `name@example.com`) |
| `<CA_CERT_FILE>` | Vault CA certificate file name (PEM format) |
| `<BOX>` | Box name in the Vault (no `:` in the name) |
| `<SECRET>` | Secret name in the Vault (no `:` in the name) |
| `<ANSIBLE_HOST>` | Linux machine that runs Ansible |

## Where to run the commands

Run everything on the Linux machine that runs `ansible-playbook` (the Ansible control machine). Not on the Vault server and not on the target hosts.

```bash
ssh root@<ANSIBLE_HOST>
```

The `export` variables in Step 4 only live in the terminal where you set them. If you open a new session or the session times out, repeat Step 4.

## Prerequisites

- A Linux machine with Python 3.
- Network access from that machine to the Vault over HTTPS.
- The Vault CA certificate (PEM file). See "How to download a KeyControl certificate" in the Vault admin guide.
- The Vault login URL: `https://<VAULT_IP>/vault/1.0/Login/<VAULT_UUID>/`. To find it, see [Retrieving Vault details](https://docs.hytrust.com/KeyControlVault/Online/Content/Books/Admin-Guide/Vault-Management/Retrieving-Vault-Details.html) in the admin guide.
- A Vault user that is a member of an access policy with the **Vault User Role**, and that policy includes the box and secret you want.
- The `pasm.py` lookup plugin file.

> The URL you see in the browser, such as `https://<VAULT_IP>/pasm/<UUID>/box/...`, is the web page. It is **not** the login URL. Use the login URL shown above.

## Step 1. Install Ansible and Python packages

Example for Oracle Linux, RHEL, Rocky and Alma:

```bash
dnf install -y epel-release
dnf install -y ansible python3-pip
python3 -m pip install requests pyyaml
```

Example for Ubuntu and Debian:

```bash
apt update
apt install -y ansible python3-pip
python3 -m pip install requests pyyaml
```

If the SSH session drops during installation, run it again. You can use `tmux` to keep the session alive.

Check:

```bash
ansible --version
python3 -m pip show requests pyyaml
```

Expected: an Ansible version is printed, and both `requests` and `PyYAML` are listed.

If you get `ansible: command not found`, Ansible is not installed on this machine. Install it here, or run the playbooks from another machine.

## Step 2. Copy the CA certificate to the Ansible machine

If the certificate is on a Windows computer, run this in Windows PowerShell:

```powershell
scp "C:\path\to\<CA_CERT_FILE>" root@<ANSIBLE_HOST>:/root/<CA_CERT_FILE>
```

Check on the Linux machine:

```bash
ls -l /root/<CA_CERT_FILE>
```

Expected: the file is listed. A Windows path such as `C:\Users\...` does not work on Linux. The file must be on the Linux machine.

## Step 3. Install the lookup plugin

Create the folder and copy the plugin. Do not rename `pasm.py`.

```powershell
ssh root@<ANSIBLE_HOST> "mkdir -p /root/ansible/lookup_plugins"
scp "C:\path\to\pasm.py" root@<ANSIBLE_HOST>:/root/ansible/lookup_plugins/pasm.py
```

Check on the Linux machine:

```bash
ls -l /root/ansible/lookup_plugins/pasm.py
```

Expected: `pasm.py` is listed.

> `pasm.py` is a plugin that Ansible loads. Do not run it directly in the shell.

## Step 4. Set the environment variables

Run these in the same terminal where you will run the playbook. The password is typed at a prompt and is not echoed.

```bash
export PASM_URL='https://<VAULT_IP>/vault/1.0/Login/<VAULT_UUID>/'
export PASM_USERNAME='<VAULT_USER>'
export PASM_CACERT_PATH='/root/<CA_CERT_FILE>'
export ANSIBLE_LOOKUP_PLUGINS=/root/ansible/lookup_plugins
read -s -p 'PASM password: ' PASM_PASSWORD; export PASM_PASSWORD; echo
```

Check (this does not print the password):

```bash
env | grep -E '^(PASM_URL|PASM_USERNAME|PASM_CACERT_PATH|ANSIBLE_LOOKUP_PLUGINS)='
```

Expected: the four values are listed.

> Do not paste the prompt text (the part in square brackets ending with `#`) when copying commands. Paste only the commands.

## Step 5. Check the Vault side

In the Vault web interface:

1. Find the box and secret name (for example box `box1`, secret `my-secret`). These are the names you use in the playbook.
2. Open **Manage Access Policies**.
3. Open a policy with the role **Vault User Role** (or create one).
4. Under **Local Users**, add your Vault user.
5. Under **Resources**, select the box and secret (or the box). Make sure the row is selected, not just listed.
6. Save the policy.

If the secret uses **Secondary Approval**, raise a checkout request in the Vault and wait for approval before running the playbook.

## Step 6. Create and run a test playbook

```bash
cat > /root/playbook.yml <<'EOF'
---
- name: Test PASM lookup
  hosts: localhost
  gather_facts: false
  tasks:
    - name: Fetch secret
      debug:
        msg: "{{ lookup('pasm', '<BOX>:<SECRET>', 'yaml') }}"
EOF

ansible-playbook /root/playbook.yml
```

Expected output:

```text
TASK [Fetch secret] ***********************************************
ok: [localhost] => {
    "msg": "<BOX>:\n  <SECRET>: <secret value>\n"
}

PLAY RECAP ********************************************************
localhost : ok=1  changed=0  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0
```

`ok=1` and `failed=0` mean the secret was fetched.

The warning `provided hosts list is empty, only localhost is available` is normal for this test.

> This test prints the secret value in the terminal. Use a test secret only. In a real playbook do not print secrets (see "Using it in your own playbook").

## Lookup syntax

```text
lookup('pasm', '<BOX>:<SECRET>', '<format>')
lookup('pasm', '<BOX1>:<SECRET1>', '<BOX2>:<SECRET2>', '<format>')
```

- Each secret is written as `box:secret`. You can list more than one.
- The plugin splits each term on `:`, so a box or secret name that contains `:` does not work.
- The last argument is the output format: `yaml` or `json`. If you leave it out, the format is `yaml`.

| Secret type | Example YAML output |
|---|---|
| Password, text, file | `box1:`<br>`  secret1: secret-value` |
| Key-value | `box1:`<br>`  secret1:`<br>`    username: password` |

File type secrets are decoded from Base64 by the plugin. Other types are returned as stored.

## Using it in your own playbook

The lookup returns text in the chosen format. Convert it to a variable and hide it from logs:

```yaml
---
- name: Use a Vault secret
  hosts: all
  gather_facts: false
  vars:
    vault_data: "{{ lookup('pasm', '<BOX>:<SECRET>', 'json') | from_json }}"
  tasks:
    - name: Use the secret without printing it
      ansible.builtin.command: /path/to/your-command
      environment:
        MY_SECRET: "{{ vault_data['<BOX>']['<SECRET>'] }}"
      no_log: true
```

Tips:

- Use `no_log: true` on any task that handles a secret.
- The lookup runs on the Ansible machine, so the environment variables from Step 4 must be set there.
- Run `ansible-playbook` in the same terminal where you did Step 4.
- The plugin logs in to the Vault again on every lookup, so fetch several secrets in one `lookup` call where you can.

## Troubleshooting

| Error message | Likely cause | What to do |
|---|---|---|
| `ansible: command not found` | Ansible not installed on this machine | Do Step 1 on the machine that runs the playbook |
| `the playbook: ... could not be found` | Wrong path or file not created | Check the path and re-run the `cat > ... <<'EOF'` command |
| `lookup plugin (pasm) not found` | `pasm.py` is missing or `ANSIBLE_LOOKUP_PLUGINS` is not set | Do Step 3, check `ls -l /root/ansible/lookup_plugins/`, and re-export the variable |
| `Environment variable for PASM_URL not set` (or USERNAME, PASSWORD, CACERT_PATH) | Variables not set in this terminal | Repeat Step 4 in the same terminal |
| `login failed ... please check login creds` (401) | Wrong user name or password | Check the credentials. Retype the password |
| `login failed ... please check login URL` (400 or 405) | Wrong login URL or UUID | Use `https://<VAULT_IP>/vault/1.0/Login/<VAULT_UUID>/`, not the browser page URL |
| `login failed ... server error` (500) | Vault server problem | Check that the Vault is healthy, then try again |
| `JSON decode` or `Expecting value` error | The URL is wrong, or something other than the Vault answered (for example a proxy page) | Check `PASM_URL` and that the Vault is reachable over HTTPS |
| `certificate verify failed` or SSL error | Wrong or missing CA file | Check `PASM_CACERT_PATH` points to the right PEM file on the Linux machine |
| `Bad request for Secret ...` with `Checkout disallowed - access denied` in the message (vault 400) | The user is not in a policy that includes this secret | Fix the policy (Step 5): add your user, select the resource, save |
| `Secret or Box not found` (404) | Wrong box or secret name | Use the exact names shown in the Vault box page |
| `Secret or Box not found` with `Secondary Approval has been enabled. Please raise a request to checkout the secret.` at the end (vault 404) | The secret needs approval and no request was raised | Raise a checkout request in the Vault and wait for approval, then run again |
| `Authorization failed please check access policy` (401) | Policy does not cover the secret, or the approval is not granted yet | Check the access policy for the user and the approval status |
| `Checkout disallowed or Secret Rotation is in progress` (409) | Secret is checked out or rotating | Wait and try again, or check the secret status in the Vault |
| Playbook hangs | The plugin sets no request timeout; the Vault or the network is not answering | Check network access to the Vault, then stop and rerun the playbook |
| `timed out waiting for input: auto-logout` | SSH session timed out | Reconnect and repeat Step 4 |

## Security notes

- Never put the Vault password in a playbook, in a file in source control, or in chat. Type it at the prompt.
- If a password or secret is shown in a terminal log, chat or ticket, treat it as exposed and change it.
- Use `no_log: true` for tasks that handle secrets.
- Give users access to only the secrets they need.

## Summary of what was done

1. Installed Ansible, `requests` and `pyyaml` on the Ansible machine.
2. Copied the Vault CA certificate to the Ansible machine.
3. Copied `pasm.py` to `/root/ansible/lookup_plugins/`.
4. Set `PASM_URL`, `PASM_USERNAME`, `PASM_PASSWORD`, `PASM_CACERT_PATH` and `ANSIBLE_LOOKUP_PLUGINS`.
5. Added the Vault user to a Vault User Role policy that includes the box and secret.
6. Ran a test playbook and got the secret with `ok=1 failed=0`.

## Expected result for the end user

After the setup, a user can write one line in a playbook, `lookup('pasm', '<BOX>:<SECRET>', 'yaml')`, and Ansible returns the secret from the Vault. No secret is stored in the playbook. Access is controlled in the Vault by the access policy.
