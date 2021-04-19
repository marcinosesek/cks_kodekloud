# CIS Benchmarks

1. Run CIS-CAT to generate Security Configuration Assessment Report
    ```
    root@controlplane:~/Assessor-CLI# ./Assessor-CLI.sh -rd /var/www/html/ -html -i -nts -rp index

    --------------------------------------------------------------------------------------
    ,o88888o.    8888    d888888o.          ,o88888o.           8.    8888888888888888 
    8888    `88.  8888  .`8888:' `88.       8888    `88.        .88.         8888       
    ,88888      `8. 8888  8.`8888.   Y8     ,88888      `8.      .8888.        8888       
    888888          8888  `8.`8888.         888888              .`88888.       8888       
    888888          8888   `8.`8888.   888  888888             .8.`88888.      8888       
    888888          8888    `8.`8888.  888  888888            .8`8.`88888.     8888       
    888888          8888     `8.`8888.      888888           .8' `8.`88888.    8888       
    `88888      .8' 8888 8b   `8.`8888.     `88888      .8' .8'   `8.`88888.   8888       
    8888    ,88'  8888 `8b.  ;8.`8888       8888    ,88' .888888888.`88888.  8888       
    `888888P'    8888  `Y8888P ,88P'        `888888P'  .8'       `8.`88888. 8888       
    --------------------------------------------------------------------------------------
            Welcome to CIS-CAT Pro Assessor; built on 01/28/2021 02:03 AM
    --------------------------------------------------------------------------------------
    This is the Center for Internet Security Configuration Assessment Tool, v4.3.1
            At any time during the selection process, enter 'q!' to exit.
    --------------------------------------------------------------------------------------

    Verifying application

    Configured report output directory to '/var/www/html/'
    Attempting to load the default sessions.properties, bundled with the application.
    Started Assessment 1/1

    Loading Benchmarks/Data-Stream Collections

    Available Benchmarks/Data-Stream Collections:
    1. CIS Controls Assessment Module - Implementation Group 1 for Windows 10 v1.0.3
    2. CIS Controls Assessment Module - Implementation Group 1 for Windows Server v1.0.0
    3. CIS Google Chrome Benchmark v2.0.0
    4. CIS Microsoft Windows 10 Enterprise Release 2004 Benchmark v1.9.1
    5. CIS Ubuntu Linux 18.04 LTS Benchmark v2.0.1
    > Select Content # (max 5): 5

    Selected 'CIS Ubuntu Linux 18.04 LTS Benchmark'

    Assessment File CIS_Ubuntu_Linux_18.04_LTS_Benchmark_v2.0.1-xccdf.xml has a valid Signature.
    Profiles:
    1. Level 1 - Server
    2. Level 2 - Server
    3. Level 1 - Workstation
    4. Level 2 - Workstation
    > Select Profile # (max 4): 1

    Selected Profile 'Level 1 - Server'

    Obtaining session connection --> Local
    Connection established.  
    Selected Checklist 'CIS Ubuntu Linux 18.04 LTS Benchmark'
    Selected Profile 'Level 1 - Server'
    Starting Assessment
    ----------------------- ASSESSMENT TARGET -----------------------------------
        Hostname: controlplane
            OS Name: linux
        OS Version: 5.4.0-1040-gcp
    OS Architecture: x86_64

    Interfaces:
    Name: lo
        IP: 127.0.0.1
    MAC: 00:00:00:00:00:00
    Name: docker0
        IP: 172.12.0.1
    MAC: 02:42:93:d5:a2:57
    Name: flannel.1
        IP: 10.244.0.0
    MAC: 1e:31:6a:7f:1a:27
    Name: cni0
        IP: 10.244.0.1
    MAC: fa:f7:d1:2d:3a:31
    Name: eth0@if168682
        IP: 10.190.255.3
    MAC: 02:42:0a:be:ff:03
    Name: eth1@if168684
        IP: 172.17.0.55
    MAC: 02:42:ac:11:00:37
    -----------------------------------------------------------------------------

    Starting Assessment - Date & Time: 04-17-2021 17:57:51
    ```

1. Run kube-bench

    ```
    curl -L https://github.com/aquasecurity/kube-bench/releases/download/v0.3.1/kube-bench_0.3.1_linux_amd64.tar.gz -o kube-bench_0.3.1_linux_amd64.tar.gz

    tar -xvf kube-bench_0.3.1_linux_amd64.tar.gz
    ./kube-bench --config-dir `pwd`/cfg --config `pwd`/cfg/config.yaml
    ```

1. K8s security permitives

    1. Secure hosts
        1. password based authentication disabled
        1. ssh key based authentication
    
    1. Secure K8s
        1. kube-apiserver
            1. authentication - who can access cluster
                1. Accounts:
                    1. Users like Admins Developers
                        1. We can't create users in K8s
                1. Service Accounts like bots
                    1. We can create service accounts in K8s
                    1. Can by used by applications
                    1. To create service account:
                        ```
                        kubectl create serviceaccount test-sa
                        ```
                    1. When service account is created then secret with token is created.
                    1. Token can be used as authorization bearer token when access K8s api server
                    1. secret with token can be automatically mounted into the pod
                    1. Each namespace has it's default service account create.
                    1. Secrt is mounted automatically in `/var/run/secrets/kubernetes.io/serviceaccount`
                    1. Default service account has restricted access
                    1. To disable automatic mount service account set in pod manifest
                        ```
                        automountServiceAccountToken: false
                        ```
                1. Authentication mechanisms:

                    1. Static password files
                        1. We can pass csv file in to K8s api server:
                            ```
                            -- basic-auth-file=user-details.csv
                            ```
                        1. To access K8s api server using user and password we can use curl command:
                            ```
                            curl -v -k https://master-node-ip:6444/api/v1/pods -u "user1:password123"
                            ```
                    1. Static token files
                        1. We can pass csv file in to K8s api server:
                            ```
                            -- token-auth-file=user-details.csv
                            ```
                        1. To access K8s api server using token we can use curl command:
                            ```
                            curl -v -k https://master-node-ip:6444/api/v1/pods --header "Authorization: Bearer <token>"
                            ```
                    1. Certificates
                    1. Identity services
            1. authorization - what can they do. for example RBAC
                1. Node authorization
                    1. Node Authorizor - any request that goes from system:node is processed by this authorizor
                1. ABAC - External access to to API - Attribute Base authorization control
                1. Webhook
                1. AlwaysAllow - allows all requests - it's enabled by default and configurd in K8s API server manifest
                1. AlwaysDeny - always deny requests
    1. TLS certificates
        1. All connections between K8s components need to ne secured
        1. We need to have at least one Certificate Authority (CA)
            1. HAs it's own pair of cert and key: ca.crt, ca.key
        1. Server components:
            1. Kube-api server: apiserver.crt, apiserver.key
            1. ETCD server: etcdserver.crt, etcdserver.key
            1. Kubelet server: kubelet.crt, kubelet.key
        1. Client components:
            1. admin: admin.crt, admin.key - kubectrl client use this user 
            1. Kube-scheduler: scheduler.crt, scheduler.key
            1. Kube-controller-manager: controller-manager.crt, controller-manager.key
            1. Kube-proxy: kube-proxy.crt, kube-proxy.key
            1. Kube-api server: apiserver-ercd-client.crt, apiserver-etcd-client.key
        1. Generate Certificate fo the cluster:
            1. Certificate Authority:
                '''
                openssl genrsa -out ca.key 2048 - To generate ca.key
                opensssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr - Certificate Signing Request
                openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
                ```
            1. Admin user
                ```
                openssl genrsa -out admin.key 2048
                openssl req -new -key admin.key -subj "/CN=kube-admin/O=system:masters" -out admin.csr
                openssl x509 -req -in admin.csr -signkey admin.key -out admin.crt
                ```
        1. To access K8s api server we can user generated certs:
            ```
            curl https://kube-apiserver:6443/api/v1/pods --key admin.key --cert admin.crt --cacert ca.crt
            ```
        1. View certificates
            ```
            openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
            ```
        1. Add new user to access K8s api server
            1. Controller manager is responsible for process CSR requests. It has in it's manifest definition path to the ca.crt and ca.key to sign requests
    1. Kubeconfig
        1. kubectl search config in $HOME/.kube/config
        1. It has 3 sections:
            * clusters
            * contexts - can have set specific namespace
            * users
        1. To see kubeconfig content run `kubectl config view`
        1. To change context:
            ```
            kubectl config use-context prod-user@production
            ```
    1. API groups
        1. /metrics /healthz /version /api /apis /logs
        1. Core group: /api/v1
        1. Named group: /apis
            1. API groups:
                ```
                /apps /extensions/ ...
                ```
        1. To see api groups:
            ```
            curl http://localhost:6443 -k
            {
                "paths": [
                    "/api",
                    "/api/v1",

                ]
            }
            ```

        1. To see named groups:
            ```
            curl http://localhost:6443/apis -k | grep "name"
            {
                "name": "extensions",
                "name": "apps",
                "name": "events.k8s.io",
                ...
            }
            ```
        1. Each API group has resources and each resource has verbs
        1. kubectl proxy runs proxy service locally and uses kubeconfig details to access the cluster
            ```
            kubectl proxy
            Starting to serve on 127.0.0.1:8001
            
            curl http://localhost:8001 -k
            {
                "paths": [
                    "/api",
                    "/api/v1",

                ]
            }
            ```
        1. Kube proxy enables connectivity between pods
    1. kubectl proxy and kubectl port forward
        1. kubectl can by enywhere
        1. We can start `kubectl proxy` client
            1. It uses credentials from ~/.kube/config file and forward requests to K8s api server
            1. Proxy will run only on the laptop
            1. With kubectl proxy we can also access services:
                ```
                curl http://localhost:8001/api/v1/namespaces/default/servces/nginx/proxy/
                ```
        1. To access services on K8s locally we can run:
            ```
            kubectl port-forward service/nginx 28080:80
            and 
            curl http://localhost:28080/
            ```
        
    1. Network Policies
        1. By default all pods can communicate each other

