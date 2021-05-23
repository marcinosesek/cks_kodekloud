# Falco

1. We can install Falco with packege manager or as DaemonSet
1. It will allow to teted suspecious system calls
    ```
    journalctl -fu falco
    ```
1. Falco implements multiple rules by default
    1. There are located in `rules.yaml`
        ```
        - rule: Detect Shell inside a container
          desc: Alert if a shell such bash is open inside the container
          condition: container.id != host and proc.name = bash
          output: Bash Opened (user=%user.name container=%container.id)
          priority: WARNING
        - list: linux_shells
          items: [bash, zsh, ksh, sh, csh]
        - macro: container
          condition: container.id != host
        ```
    1. `https://falco.org/docs/rules/supported-fields/`

1. Configuration files:
    1. `/etc/falco/falco.yaml` - has all configuration options
        ```
        ...
        rules_files:
          - /etc/falco/falco_rules.yaml
          - /etc/falco/falco_rules.local.yaml
          - /etc/falco/k8s_audit_rules.yaml
          - /etc/falco/rules.d
        ...
        json_output: false
        log_stderr: true
        log_syslog: true
        log_level: info

        priority: debug

        stdout_output:
          enabled: true
        ```
    1. `/etc/falco/falco_rules.yaml` - default rules/list/macro definitions
    1. To apply rules we need to `Hot Reload`
        1. Restart Falco process - `kill -9` or restart service

# Ensure immutablity of Containers at Runtime
1. Make sure that:
    * readOnlyRootFilesystem is not set to false
    * Privileged is not set to true
    * runAsUser is not set to 0
1. To allow write data in container we can use emptyDir volue and mount it in desired location.  

1. Audit logs
    1. When we send some request to K8s api server the following request are send
        1. `RequestReceived`
        1. `RequestStarted`
        1. `RequestComplete`
        1. `Panic`
    1. Requests can generate events for each type.
    1. We should not allow to generete all events
    1. To define audit policy we need to create `Policy` resource
        ```
        apiVersion: audit.k8s.io/v1
        kind: Policy
        omitStages: ['RequestReceived']
        rules:
          - namespace: ["prod-namespace"]
            verb: ["delete"]
            resources: 
            - groups: " "
              resources: ["pods"]
              resourceNames: ["webapp-pod"]
            level: None or Metadata or Reqiuest or RequestResponse
        ```
    1. Auditing is disabled in K8s by default
    1. To enable auditing we need to add to k8s api server manifest or service and restart api server:
        ```
        - --audit-log-path=/var/log/k8s-audit.log
        - --audit-policy-file=/etc/kubernetes/audit-policy.yaml
        - --audit-log-maxage=10
        - --audit-log-maxbackup=5
        - --audit-log-maxsize=5
        ```