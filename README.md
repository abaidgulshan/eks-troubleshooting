# eks-troubleshooting
## EKS 1.22 upgrade dashboard  error 
*  ```ingresses:Unknown error (404) the server could not find the requested resource (get ingresses.extensions)```
*  Solution : `kubectl edit -n kubernetes-dashboard deployment kubernetes-dashboard`
*  change it to `v2.5.0`
## EKS 1.24 Metric servics error (ServiceUnavailable): the server is currently unable to handle the request (get pods.metrics.k8s.io)
* EKS addon metric service deploy through terraform and giving error `error (ServiceUnavailable): the server is currently unable to handle the request (get pods.metrics.k8s.io)`
* Solution: As we are using AWS EKS Module and in module resouce add this in Node security rule
```    # Allows Control Plane Nodes to talk to Worker nodes on all ports. Added this to simplify the example and further avoid issues with Add-ons communication with Control plane.
    # This can be restricted further to specific port based on the requirement for each Add-on e.g., metrics-server 4443, spark-operator 8080, karpenter 8443 etc.
    # Change this according to your security requirements if needed
    ingress_cluster_to_node_all_traffic = {
      description                   = "Cluster API to Nodegroup all traffic"
      protocol                      = "-1"
      from_port                     = 0
      to_port                       = 0
      type                          = "ingress"
      source_cluster_security_group = true
    }
```
* reference: https://github.com/kubernetes-sigs/metrics-server/issues/1024#issuecomment-1129914389
