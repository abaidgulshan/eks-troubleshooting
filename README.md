#  ✴️ AWS EKS Troubleshooting ✴️
## EKS 1.22 upgrade dashboard  error 
*  ❌ **Error**:```ingresses:Unknown error (404) the server could not find the requested resource (get ingresses.extensions)```
*  🎯 **Solution** : `kubectl edit -n kubernetes-dashboard deployment kubernetes-dashboard`
*  change it to `v2.5.0`
## EKS 1.24 Metric servics error (ServiceUnavailable): the server is currently unable to handle the request (get pods.metrics.k8s.io)
* ❌ **Error**:EKS addon metric service deploy through terraform and giving error `error (ServiceUnavailable): the server is currently unable to handle the request (get pods.metrics.k8s.io)`
* 🎯 **Solution**: As we are using AWS EKS Module and in module resouce add this in Node security rule
``` node_security_group_additional_rules = {
    # Extend node-to-node security group rules. Recommended and required for the Add-ons
    ingress_self_all = {
      description = "Node to node all ports/protocols"
      protocol    = "-1"
      from_port   = 0
      to_port     = 0
      type        = "ingress"
      self        = true
    }
    #Recommended outbound traffic for Node groups
    egress_all = {
      description      = "Node all egress"
      protocol         = "-1"
      from_port        = 0
      to_port          = 0
      type             = "egress"
      cidr_blocks      = ["0.0.0.0/0"]
      ipv6_cidr_blocks = ["::/0"]
    }
    # Allows Control Plane Nodes to talk to Worker nodes on all ports. Added this to simplify the example and further avoid issues with Add-ons communication with Control plane.
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
## EKS ALB S3 Logs Access Denied for bucket: Please check S3bucket permission
* ❌ **Error**:EKS ALB S3 logs access Denied error when enable 
* 🎯 **Solution**: We need to make sure S3 bucket policy has `AWS ELB ACCOUNT ID` which is `127311923021` for `us-east-1`
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "s3_write_alb_logs",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::127311923021:root"
            },
            "Action": "s3:*",
            "Resource": "arn:aws:s3:::S3BucketName/*"
        }
    ]
}
```
![Screenshot (6)](https://github.com/abaidgulshan/eks-troubleshooting/assets/7329596/5f1ae82e-9ecf-43ca-896b-e66d7ee49eed)
## EKS Remove bulk jobs in namespace
* 🎯 **Solution**: `kubectl delete jobs -n monitoring "kubectl get jobs -n monitoring -o custom-columns=:.metadata.name" `
* 🙌🏼 **reference**: https://stackoverflow.com/questions/43675231/kubernetes-delete-all-jobs-in-bulk

## EKS Timeout Error from Ec2 instance 
* **error**: couldn't get current server API group list: Get eks.amazonaws.com/api?timeout=32s": dial tcp 10.122.65.248:443: i/o timeout
* **solution**: Try to debug the error using command `kubectl get nodes -v=10`
* **reference** : https://stackoverflow.com/questions/76841889/kubectl-error-memcache-go265-couldn-t-get-current-server-api-group-list-get

## EKS Timeout Error from Ec2 instance 
* 🤔 **Try**: After upgrading EKS version from 1.22 to 1.23 we try to sync terrafrom with newer version
* ❌ **error**: ValidationError(CatalogSource.spec): unknown field "grpcPodConfig" in com.coreos.operators.v1alpha1.CatalogSource .spec". After updating the CRDs for OLM
* 🎯 **solution**: 
    ```
     wget https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.25.0/crds.yaml
     kubectl apply -f crds.yaml --server-side=true
     kubectl get crd
     sudo apt install golang-go
     make install
     export PATH=$PATH:$(go env GOPATH)/bin
     kubectl get customresourcedefinitions.apiextensions.k8s.io catalogsources.operators.coreos.com -o yaml | tfk8s --strip -o sample.tf
     export PATH=$PATH:$(go env GOPATH)/bin
     kubectl get customresourcedefinitions.apiextensions.k8s.io catalogsources.operators.coreos.com -o yaml | tfk8s --strip -o sample.tf
     kubectl get customresourcedefinitions.apiextensions.k8s.io clusterserviceversions.operators.coreos.com -o yaml | tfk8s --strip -o sample2.tf
     kubectl get customresourcedefinitions.apiextensions.k8s.io installplans.operators.coreos.com -o yaml | tfk8s --strip -o sample3.tf
     kubectl get customresourcedefinitions.apiextensions.k8s.io olmconfigs.operators.coreos.com -o yaml | tfk8s --strip -o sample4.tf
     kubectl get customresourcedefinitions.apiextensions.k8s.io operatorconditions.operators.coreos.com -o yaml | tfk8s --strip -o sample5.tf
     kubectl get customresourcedefinitions.apiextensions.k8s.io operatorgroups.operators.coreos.com -o yaml | tfk8s --strip -o sample6.tf
     kubectl get customresourcedefinitions.apiextensions.k8s.io operators.operators.coreos.com -o yaml | tfk8s --strip -o sample7.tf
     kubectl get customresourcedefinitions.apiextensions.k8s.io subscriptions.operators.coreos.com -o yaml | tfk8s --strip -o sample8.tf
    ```
* 🙌🏼 **reference** : https://github.com/operator-framework/operator-lifecycle-manager/issues/2695
