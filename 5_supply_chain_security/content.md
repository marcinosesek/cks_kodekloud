# Minimize Base image footprint

1. Kubesec - static analysis of user workload
    1. Used to scan k8s manifests 
1. Trivy - scan docker imagres
    1. To scan docker image:
        ```
        trivy image --secerity CRITICAL,HIGH nginx:1.18.0
        ```
    1. To scane docker tar files use
        ```
        trivy image --input archive.tar --output /root/out --format json
        ```

