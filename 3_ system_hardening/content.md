# Limit node access

1. Only some people should have access to node
1. `/etc/passwd` - contains list of users and its shell info
1. `/etc/shadow` - contains hashed users passwords
1. `/etc/groups` - contains list of groups


# SSH hardening

1. Access to the node via root user should be disabled:
    ```
    vi /etc/ssh/sshd_config
    PremitRootLogin no

    service sshd restart
    ```

1. Disable access to the node with password
    ```
    vi /etc/ssh/sshd_config
    PasswordAuthentication no

    service sshd restart
    ```

# Restrict Kernel Modules

1. Load manually modules
    ```
    modprobe pcspkr
    ```

1. List loaded modules
    ```
    lsmod
    ```

1. To restrict load some modules update `/etc/modprobe.d/blacklist.conf`
    ```
    vi /etc/modprobe.d/blacklist.conf
    blacklist sctp
    blacklist dccp

    shutdown -r now

    lsmod | grep sctp - module should not be listed 
    lsmod | grep dccp - module should not be listed 
    ```

# Disable Open Ports

1. List open ports
    ```
    netstat -an | grep -w LISTEN
    ```

1. To check which service uses specific port:
    ```
    cat /etc/services | grep -w 53
    ```

# Minimize IAM roles

1. AWS root user can create new users
1. We can assign some permissions using IAM policies for each user
1. And deny them to rest users
1. AWS trusted Adivisor, Security Command Center Azure Advisor - can help to analyze IAM policies/Roles etc

# Restrict Network Access

1. UFW - uncomplicated firewall - frontend to iptables
1. Install UFW
    ```
    apt update
    apt-get install ufw
    systemctl start ufw
    ufw status
    ```
1. Define rules in ufw
    ```
    ufw default allow outgoing
    ufw default deny incoming
    ```
1. Allow access from some ip:
    ```
    ufw allow from 172.16.238.5 to any from port 22 proto tcp
    ufw allow from 172.16.238.5 to any from port 80 proto tcp
    ufw deny 8080
    ```
1. Enable ufw
    ```
    ufw enable
    ufw status
    ```
1. Delete rule
    ```
    ufw delete 5
    ```

# Linux Syscalls

1. Linux Kernel can be split on User and Kernel space
1. From Application/process we can call System calls
1. Tracing syscals
    ```
    which strace
    strace touch /tmp/file
    ```
1. To find process PID run: `pidof etcd`
1. We can trace process PID: `strace -p <PID>`
1. To see list of all syscals run by command: `strace -c touch /tmp/file`

# Restrict syscals using seccomp

1. Seccomp can work in 3 modes:
    1. mode 0 - disabled
    1. mode 1 - strict
    1. mode 2 - filtered

1. Docker has build in seccomp filter
1. Default seccomp config folder: `/var/lib/kubelet/seccomp`
1. To use seccomp in pod we need to: 
    1. `mkdir -p /var/lib/kubelet/seccomp/profiles`
    1. `vi /var/lib/kubelet/seccomp/profiles/audit.json`
        ```
        {
            "defaultAction": "SCMP_ACT_LOG"
        }
        ```
        ```
        securityContext:
            seccompPrefix:
                type: Localhost
                localhostProfile: profiles/audit.json
        containers:
        ...
            securityContext:
                allowPrivilegeEscalation: false
        ```
1. Logs are generated in `/var/log/syslog`
1. Syscalls numbers can be found in `/usr/inclusde/asm/unistd_64.h`
1. tracee container also can be run to check which syscalls was called

# AppArmor

1. Linux security module
1. `systemctl status apparmor`
1. To check if AppArmor is loaded check on load if 
    ```
    cat /sys/module/apparmor/parameters/enabled
    y
    ```
1. It's applied to the application via profile
    ```
    cat /sys/kernel/security/apparmor/profiles
    ```
1. To check status of AppArmor use `aa-status`
1. AppArmor profile can have one of 3 modes
    1. enforce
    1. complain
    1. unconfined
1. Create AppArmor profile
    1. `apt-get install -y apparmor-utils`
    1. `aa-genprof /root/add_data.sh`
    1. In new terminal run `/root/add_data.sh` and answere apparmor questions
1. Load profile `apparmor_parser /etc/apparmor.d/root.add_data.sh`
1. Disable profile `apparmor_parser /etc/apparmor.d/root.add_data.sh` and
    `ln -s /etc/apparmor.d/root.add_data.sh /etc/apparmor.d/disable/`
1. Use AppArmor in Kubernetes
    1. Run `aa-status` on all nodes
    1. Add to the pod annotation
        ```
        ...
        metadata:
            annotations:
                container.apparmor.security.beta.kubernetes.io/<container_name>: localhost/<profile-name>
        ```

# Linux Capabilities

1. In Kernel 2.2 Provileged Proceess allows assign lots of Capabilities based on the functionality
1. `getcap /usr/bin/ping`, getcap <PID>
1. We can add capabilities to the container
    ```
    ...
    containers:
    - name: test
    ...
      securityContext:
        capabilities:
          add: ["SYS_TIME"]
          drop: ["CHOWN"]
    ```

# Admission Controllers

1. When we try to access K8s api server we need to:
    * authentication - for example with key/certs
    * authorization - for example with RBAC
    * admission controllers - allows for example change requestes 
        * AlwaysPullImages
        * DefaultSorageClass
        * NamespaceExists
        * NamespaceAutoProvision - disabled by default
        * ..
1. To see list of enabled Admissions Controllers run:
    ```
    kube-apiserver -h | grep enable-admission-plugins
    ```
1. To enable AdmissionController update enable-admission-plugin in k8s manifest or service file definition
1. To disable AdmissionController update disable-admission-plugin in k8s manifest or service file definition
1. Mutating Admission Controllers can change request before object is created
1. Mutating admission controllers are called before validating admission controllers

# Pod Security Policy

1. To enable PSP we need to enable it in admission controller section in k8s manifest file/service definition file

# OPA - Load Policy

1. OPA starts work when user is Authenticated then it define what user can do
1. To install OPA we can:
    1. Donwload binary
    1. Make it executable
    1. Run OPA:
        ```
        ./opa run -s
        ```
1. Load Policy written in Rego language
    1. Example policy
        ```
        vi example.rego
        package httpapi.authz

        # HTTP API request
        import input

        default allow = false

        allow {
            input.path == "home"
            input.user == "john"
        }
        ```
        ```
        curl -X PUT --data-binary @example.rego http://localhost:8181/v1/policies/example1
        ```
    1. List policies
        ```
        curl http://localhost:8181/v1/policies
        ```
1. ValidatingAdmissionWebhook
    1. kube-mgmt service is deployed with OPA and is used to
        1. replicate K8s resources to OPA
        1. Load policies into OPA via K8s
        1. It's deployed as K8s sidecar