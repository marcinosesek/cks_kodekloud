# Manage K8s Secrets

1. We can inject secrets to the pod from environment variables 
1. If we will mount secrets each secret entry will be mounted as a file

# Container Sandboxing

1. Containers on host share the same kernel!!!
1. Containers can run syscalls to the same kernel
1. To restrict syscalls usage we can use for example seccomp, apparmor

1. gVisor - Allows additional layer of izolation between container and host kernel - provided by google - like custom kernel
    1. Sentry - independenty intercept and respond for container syscalls
    1. Gofer - file proxy to access system files
1. Kata Containers
    1. It provides for each container light Weight Virtual Machine. Each container will work on it's own Kernel
    1. It will work on cloud that supports nested virtualization. And performance can be small
1. Container Runtime
    1. runC is default runtime. It's OCI compatible
    1. Kata container uses kata-runtime runtime
    1. gVisor use Runsc runtime
    1. To run container with specific runtime:
        ```
        docker run --runtime kata -d nginx
        docker run --runtime runsc -d nginx
        ```
1. Use runtime configuration in K8s
    1. Create `RuntimeClass` resource
        ```
        apiVersion: node.k8s.io/v1beta1
        kind: RuntimeClass
        metadata:
            name: gvisor
        handler: runsc - need to be set to right runtime
        ```
    1. In pod we can specify runtimeClassName field:
        ```
        ...
        spec:
            runtimeClassName: gvisor
        ```
    1. Right now we can check on host if container process is not visible:
        ```
        pgrep -a nginx
        ```
    1. And runsc will run on node
        ```
        pgrep -a runsc
        ```

1. Mutual TLS to secure pod to pod communications.
    1. Istio and linkerd implements MTlS - service mesh
    1. Istio sidecar container decrypt data
    1. Types of encryptions in istion:
        1. Permisive/Oportunistic
        1. Enforced/Strict
