# Ansible Security Immersion Day Lab Outline
## Initial Setup
1. Fork https://github.com/mysidlabs/sid-ansible-security
1. Create local siduser.pem file:
    1. Copy raw contents of: https://github.com/{{ YOUR ACCOUNT GOES HERE }}/sid-ansible-security/blob/master/lab-key/siduser-sec-sid.pem and paste into a file in your working ssh directory.
1. Connect to your jump host:
    *       ssh -i siduser.pem ec2-user@siduser###.jump.mysidlabs.com
1. Clone your forked repo.
    *       git clone https://github.com/dmatthews-sirius/sid-ansible-security.git

## Lab 1 - Execution Environments
1. Review project structure
1. Review execution environments
    1. Review ee project files
    1. Review [`podman`](https://https://podman.io/) vs. `docker`
    1. Review [`ansible-buider`](https://www.ansible.com/blog/introduction-to-ansible-builder)
    1. Build sec-sid-mitigation-ee
        *       ansible-builder build --tag sec_sid_ee
    1. Use `ansible-navigator` to inspect new image

## Lab 2 - Execute Plays with `ansible-navigator`
1. Review `ansible-navigator.yml`
    * Make sure execution-environment is set to:
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
1. Go back to ssh terminal and execute the curl script:
    ```bash
    ./simulate-amp-securex-POST-request
    ```



