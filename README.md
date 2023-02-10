# eks-troubleshooting
## EKS 1.22 upgrade dashboard  error 
*  ```ingresses:Unknown error (404) the server could not find the requested resource (get ingresses.extensions)```
*  Solution : `kubectl edit -n kubernetes-dashboard deployment kubernetes-dashboard`
*  change it to `v2.5.0`
