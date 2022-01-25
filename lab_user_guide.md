# Ansible Security Immersion Day Lab Outline
## Initial Setup
### Fork the lab repo and connect to you r jump box.
1. Fork https://github.com/mysidlabs/sid-ansible-security
1. Connect to your jump host:
    ```
    >>  ssh siduser###@siduser###.jump.mysidlabs.com
    Password: Ger1974!
    ```
    You should then see the following:
    ```bash
        Welcome to the CDW/Sirius Red Hat Immersion Day lab environment
        +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
        Login Succeeded!
        siduser101@jump:~$ 
    ```
1. Clone your forked repo.
    ```bash
        siduser101@jump:~$ git clone https://github.com/<YOUR GIT HUB ACCOUNT>/sid-ansible-security.git
        Cloning into 'sid-ansible-security'...
        remote: Enumerating objects: 98, done.
        remote: Counting objects: 100% (98/98), done.
        remote: Compressing objects: 100% (56/56), done.
        remote: Total 98 (delta 26), reused 90 (delta 20), pack-reused 0
        Unpacking objects: 100% (98/98), 18.73 KiB | 958.00 KiB/s, done.
        siduser101@jump:~$
    ```

### Review jump environment
Installed tools:
  * `git` - command line tool to interact with git source control repositories.
  * `podman` - a name-spaced drop in replacement for the docker cli.
  * `nano` - a simple text editor.
  * `vim` - a powerful text editor.
  * `ansible-builder` - a command line wrapper to podman to build execution environments.
  * `ansible-navigator` - a powerful replacement to the old ansible-playbook command.
  * `ansible-vault` - a command line encryption tool to protect secrets for ansible-playbooks.
  * `tree` - a command line utilitiy to show recursive directory listings in a visual form.

    We will be editing text files throught the course of this lab.  Both `nano` and `vim` are available.  If you are inexperienced with `vim`,  `nano` will be the better choice, as it behaves more like a basic text editor.

### Decrypt variables file
Note that ansible-vault is not currently supported by ansible-navigator.  We will need to decrypt are group_vars/all.yml file.

Execute the following command to decrypt the group_vars/all.yml file:
```bash
    siduser101@jump:~/sid-ansible-security$ ansible-vault decrypt group_vars/all.yml 
    [DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the controller starting with Ansible 2.12. Current 
    version: 3.6.8 (default, Mar 18 2021, 08:58:41) [GCC 8.4.1 20200928 (Red Hat 8.4.1-1)]. This feature will be removed 
    from ansible-core in version 2.12. Deprecation warnings can be disabled by setting deprecation_warnings=False in 
    ansible.cfg.
    Vault password: Ger1974!
    Decryption successful
    siduser101@jump:~/sid-ansible-security$
```


## Lab 1 - `ansible-navigator` and execution environments
1. Review project structure
```bash
    cd sid-ansible-security
```
1. Review execution environments
    1. Review ee project files
    1. Review [`podman`](https://https://podman.io/) vs. `docker`
    1. Review [`ansible-buider`](https://www.ansible.com/blog/introduction-to-ansible-builder)
    1. Build sec-sid-mitigation-ee

> :warning: **OPTIONAL** building this execution environment is optional.  It will take a couple of minutes to complete.
```bash
    ansible-builder build --tag sec_sid_ee
```
1. Use `podman` to list the local images:
```bash
    podman images
```
1. Use `ansible-navigator` to inspect new image

## Lab 2 - Execute Plays with `ansible-navigator`
1. Review `ansible-navigator.yml`

    Make sure execution-environment is set to:
```yaml
    execution-environment:
      enabled: true
      container-engine: podman
      image: sec_sid_ee:latest
      pull-policy: missing
```
1. Review `ansible-vault` and look at:
    ```
        group_vars/all.yml
        roles/paloalto/vars/main.yml
        roles/jira/vars/main.yml
    ```
1. Edit `create-secuirty-rule.yml` and modify rule_name key to:
    ```yaml
        rule_name: "siduser###- Block amp_{{ item.detection_id }}"
    ```
1. Execute the `create-security-rule.yml` with `ansible-navigator`
    ```bash
    ansible-navigator run create-security-rule.yml --extra-vars "@dev-data/amp_single_event.json" --ask-vault-pass
    ```
1. Vault Password is: `Ger1974!`
1. Instructor will show rules added to Palo Alto NGFW
1. Edit `create-soc-ticket.yml` and modify summary key to:
    ```yaml
        summary: "siduser### - AMP: {{item.event_type}}: RHAP Automated mitigation"
    ```
1. Execute the `create-soc-ticket.yml` playbook with `ansible-navigator`
    ```bash
    ansible-navigator run create-soc-ticket.yml --extra-vars "@dev-data/amp_single_event.json" --ask-vault-pass
    ```
1. Vault Password is: `Ger1974!`
1. Instructor will show tickets added to JIRA project
1. Edit `roles/paloalto/tasks/main.yml` `rule_name:` key and `roles/jira/tasks/main.yml` `summary:` key to include your siduserID.

1. Execute `amp-mitigation-play.yml` with `ansible-navigator`:
    ```bash
    ansible-navigator run amp-mitigation-play.yml --extra-vars "@dev-data/amp_single_event.json" --ask-vault-pass
    ```


## Lab 3 - Automation controller and the AC REST api
1. Login to https://rhap2.mysidlabs.com
    * user: siduser###
    * password: Ger1974!
1. Review execution environments in AC.
1. Create project for individual repos.
1. Review registering external applications.
1. Review generating oauth2 token.
1. Create `amp-mitigation-play.yml` template.
1. Create workflow template for amp-mitigation:
    * Add project refresh node.
    * Add template node.
    * save
1. Review the AC REST API: `https://rhap2.mysidlabs.com/api/v2/`
    1. Look at the `/api/v2/workflow_job_templates/` details.
    1. determine the id of your workflow.
1. Go back to ssh terminal and edit ./simulate-amp-securex-POST-request
    ```yaml
     1 curl --location --request POST 'https://rhap2.mysidlabs.com/api/v2/workflow_job_templates/{{ CHANGE TO YOUR WORKFLOW ID }}/launch/' \
    ```
1. Execute the curl script:
    ```bash
    ./simulate-amp-securex-POST-request
    ```
1. In AC go to the jobs list and view your workflow in progress.

## FINIS



