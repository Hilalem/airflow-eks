# airflow-eks

https://github.com/apache/airflow/blob/main/chart/values.yaml
https://github.com/airflow-helm/charts/blob/main/charts/airflow/values.yaml
https://airflow.apache.org/docs/helm-chart/stable/manage-dags-files.html



Official Helm Chart version
1.11.0 

Apache Airflow version
2.7.1

airflow needs to store data in volumes, dynamic volumes as soon as the airflow needs volume k8s cluster will create for us.
That said in order to create that volume with EBS, we need to install the corresponding driver
https://github.com/kubernetes-sigs/aws-ebs-csi-driver/blob/master/docs/install.md 

Helm chart for AWS EBS CSI Driver
Add the aws-ebs-csi-driver Helm repository.
helm repo add aws-ebs-csi-driver https://kubernetes-sigs.github.io/aws-ebs-csi-driver
helm repo update
Install the latest release of the driver.
helm upgrade --install aws-ebs-csi-driver \
    --namespace kube-system \
    aws-ebs-csi-driver/aws-ebs-csi-driver
 
helm repo:
in order to deploy airflow we use helm chart and we should inform flux where to find this helm chart:
so we  create a repository object. The goal of the repo object is to define where the helm chart of airflow is. flux will check in 1m intervals if there is a new airflow helm chart
in url .. airflow apache.org. if we push that file flux will create the coreesponding object in k8s cluster
helm release object is used to deploy airflow based on its helmchart in cluster
    - helm release is an instance of the chart in cluster
    release name is the release for helm, name is helmrelease for flux (terraform)
    airflow version : version of the airflow to be deployed
    
    

    
    
    flux logs --follow --level=error --all-namespaces
    flux delete helmrelease airflow -n dev
    flux get all
    
    
    
   dags:
      persistence:
        enabled: False
      gitSync:
        enabled: True
        repo: git@github.com:Hilalem/airflow-private-dags.git
        branch: master
        maxFailures: 0
        subPath: ""
        wait: 60
        sshKeySecret: airflow-ssh-secret
    extraSecrets:
      airflow-ssh-secret:
        data: |
          gitSshKey: 'LS0tLS1CRUdJTiBPUEVOU1NIIFBSSVZBVEUgS0VZLS0tLS0KYjNCbGJuTnphQzFyWlhrdGRqRUFBQUFBQkc1dmJtVUFBQUFFYm05dVpRQUFBQUFBQUFBQkFBQUNGd0FBQUFkemMyZ3RjbgpOaEFBQUFBd0VBQVFBQUFnRUF0bkxldzZNRDNnN1JhRkdQMkdGcllNT1VKQytXVGNCcS9adEh2UVo4dStmVUFYV1R1ZEFMCmhnN0V0VlBWM0xBU2pvSlM4Q3c5MnFVU05MaTdVWWVGK1VsSEhaNnRVSTQ4K0xpREQ3RU1uMzhORkVTNDgvUUZuRGdOSUYKSWVmaklyVHRWcHJuS2QyN05ITkpWR2JXem5pTmNHMWhwVWJnbzZSZmExS3N6YnNlSGVNMWNkUXdIQXhTckdjWmhmMCtXWgoxc1hiYUt1Qk5LRlVUemc5TmFsQ2IyQTdyNmJyNU9nV2pMNTMzOVlFVTBKai9oaEhLY0dBYWNzQXIxR2s0NW5MTFJIMG1TCkZ6alVNY1A4RzJESm9WaEhPb25RRmVZL3NMRkhTK2NDM1ZLUEtVVGw4a1VncW82UWw5Y2NTRWo2VTdUaUpMN2s1ZUFJWFAKQmlzZnpXTE1JeWR3OE5yWkxIYjV1RmZrNjRLZURtQ2g4UWFPWkRkVkhnK3UyQTE4MFBMNXZJY3VTVzhaYUY1WHJYOFhtYQpzRWo2cU5sL2cyaFFCbjRkYnRub01mODd3RjFrem16ZEwvUzUxRXBtU2RKbVdpSXorb2dWWDB3bXB5Qlc0ZDRuYUNYamZsCkF1dEZINUpxK1g5MHNMcVA5bklWOUZEakVTTm9MRmVjc0xmN1VnLzdtMExqV3ZJd2FlZ043eHg0YXUzTDNSdngreGRKYzcKeDcxZUlGd3NzVDQyOGgwWGNNeFdHNFFid1BMVGg5UTl6NGR6cmZFYktvK01mQmZSc0psY0NLb0dRU2xFd242S3Rzcys4dgo1ejByRkl1azJRdW9hUHBkS01YQVBwUUcrM1Rzc1pOdXlvR05aelQ2MUgxeUtKWVlRMFRsTFFJeFVLU2N6WW8xRzArRUhkCnNBQUFkdzNja09ldDNKRG5vQUFBQUhjM05vTFhKellRQUFBZ0VBdG5MZXc2TUQzZzdSYUZHUDJHRnJZTU9VSkMrV1RjQnEKL1p0SHZRWjh1K2ZVQVhXVHVkQUxoZzdFdFZQVjNMQVNqb0pTOEN3OTJxVVNOTGk3VVllRitVbEhIWjZ0VUk0OCtMaURENwpFTW4zOE5GRVM0OC9RRm5EZ05JRkllZmpJclR0VnBybktkMjdOSE5KVkdiV3puaU5jRzFocFViZ282UmZhMUtzemJzZUhlCk0xY2RRd0hBeFNyR2NaaGYwK1daMXNYYmFLdUJOS0ZVVHpnOU5hbENiMkE3cjZicjVPZ1dqTDUzMzlZRVUwSmovaGhIS2MKR0FhY3NBcjFHazQ1bkxMUkgwbVNGempVTWNQOEcyREpvVmhIT29uUUZlWS9zTEZIUytjQzNWS1BLVVRsOGtVZ3FvNlFsOQpjY1NFajZVN1RpSkw3azVlQUlYUEJpc2Z6V0xNSXlkdzhOclpMSGI1dUZmazY0S2VEbUNoOFFhT1pEZFZIZyt1MkExODBQCkw1dkljdVNXOFphRjVYclg4WG1hc0VqNnFObC9nMmhRQm40ZGJ0bm9NZjg3d0Yxa3ptemRML1M1MUVwbVNkSm1XaUl6K28KZ1ZYMHdtcHlCVzRkNG5hQ1hqZmxBdXRGSDVKcStYOTBzTHFQOW5JVjlGRGpFU05vTEZlY3NMZjdVZy83bTBMald2SXdhZQpnTjd4eDRhdTNMM1J2eCt4ZEpjN3g3MWVJRndzc1Q0MjhoMFhjTXhXRzRRYndQTFRoOVE5ejRkenJmRWJLbytNZkJmUnNKCmxjQ0tvR1FTbEV3bjZLdHNzKzh2NXowckZJdWsyUXVvYVBwZEtNWEFQcFFHKzNUc3NaTnV5b0dOWnpUNjFIMXlLSllZUTAKVGxMUUl4VUtTY3pZbzFHMCtFSGRzQUFBQURBUUFCQUFBQ0FEZlhvdDBvV1BldmUySzlqQlNEaE5VaUo0YUgxaTVJRmJjMwp2dFpaVlBaQ2Q3NVdtWGVHK08vNE56YjB5UUY2RnNQdG1hc1BMNE5yZ000SU9MVTBrTW9ESnJRbkxBNzY2aWlLZVBybGl1CktLaFp0TmlYcUpsdW9Bc2V2UmxxbXplMVB1dUNEL3pkYy80U3gwQUJGZ0F1SDhSb2hqbUxIeGlYSitsMmJaT3VrUUkrMTgKM1dUejlXZWp0d2R2eHV6WldxUEUvV25sREcvcWJSNnVMUFh1cjNuZGI0ZE14T2pVTElxNWhZRktSUnFpdUc0K1NoRzZ1eQo0bVJ6aGgrZUQ3NHBjUjV0YXp5V1ZVUkNJempTQUFUVk96L3NqSGNYZVNrQXFiek9vUzdTcFNUUmhMd0pjbjBoazVvVjZmCjhwN3d5TjBhWERrOVhVSTFzbTdGNWo3NzNWVGs0VDl0YjNtT0k3TWFGQWJXanhWSjRRWGIwKzZQMU5IeStlMmZPcUtDL3cKejdUQks3Vm8vV0ZXcm5uV3dKd0Z2V05xUnM5dTdPTmJpc2I4WVdVbnV5b2dnU0psNkRDUDd5NStoVGE2TFhDbFNzMDY5OAp5eTNWZjA5RDJydHVXaVhwS0pxTlloSVFnMStNaGNOSFRRWGxWbm5TVFhmRnc2dXhMOUxCbGQzMkpxeUpHTEM3eERpMHNaCjFqdmNUeVp1aGhIZEVLSWlhdGJOZldQakozUk9YbUhKQit4ajNhRjVWM3BwZmhBbGlvUldGUEM2aWt4WUk0amNPMUFha1oKSVl6SlJJck9FUUNsTENwWGFLMDZGMFo1bFVLZVQ5cDJ4QUtJSTJKa2lyOU5YcG5HRnNITTRLcUtMckNHMnJBcUpCeCtPTwpGOUMvcU5PN0x0YlhqQ1Z6K0JBQUFCQVFDMm1QbnpkRFhnZHEzWmcwbzFLOGlUcU0rVCtNWnRqSUZJdXcvbUYyMkdjU1pkCktIRnlUT2k5ekdEemw3bXJIUm9rMnBTRkR0Qkg5ZlN5MW5rcUY0Z3JOL1hGWkJYNytORzMwY1BxakNLcXZ6RE9KclFZZjEKR0owUkk5NnZ4SS9mQ0t1amZGN21rcmtMRzNxWjd6MERtaEJKWEVEZWNweHpqOEd4anVvNzFCOXJNTHduQ2YrUHE3bCtGVAp2bVUvb0UzOWRyVlVXcGczZFluWEt4TkZuZG9zWUF6UVN6VzlvNVllUFR5R0JPemlVMGtVcVNMc3ZtR1FxdkcybXVKTTVpCm54b3Q3RTJmdEs0R01HTTNrU1g1VmEzcFpBTE4yV2RNQlE2MitrY2lRVGkycG5VUGJPcjhIQ0IzSllvSW9XU3g0K0tZOW0KSFJQb2dNRXN3Q3ZDTXI1b0FBQUJBUURhQVF4bzJtQzIrNjIzMUNic3lBV2VtTXF2QTdGM3pxcnYzV1J3bkh2b0xMcHJ4NApGUFRXb2F6a1VCV3JPYmxBc3k0Q3pNWWQwaGxNa212OGd6TUJoQkFod1JyT05CaXhjcDVlY0ZlMFNLZG8vNm90S2lRbDJ5CjFvRi9tdnRZRGg2OXc3NmlsWXN5eWFvQUtEWTRTNUZ4VTVpaUloWHdYOGdmTXIxeVlkb2FLZ28yTmxDSWRwQTJjVExZOHgKTkJVQU1WdVNCNkhUR2l2OWJ5dkJYekppMWluNDNiV3B4NVNOa0M1eGV6TW1QL0xhMEF3ZWdDcExzRVVET3VrY0tQenRwWQpqdmZvL3lpR0dGcTQ3Rzdzbm9kdDJ5V1BpMldnSHVqeWllekdqN21XQVN6cjFNKzRnQkhjbzhSYW1vaUFYTXJhbEpSNUovClU0OEVWblhESVc0QjhiQUFBQkFRRFdQMmQrUDVjcFIwdFlrYlQrdDlQaVRzalZpTTN2V29aRk9lZGd5Ykh4cldWclVaSUIKcjh5SWtLaEc2QnFKY1VuWHU4Zm9VREsvblNvTEFNUkFsdDErMk1LcU9rQ2VtdjFvYS8zdHNEUms5cEJvcWEvNDJQZE1CTgo4aEhUVnd3RWhlVnZseUYzd2twbE5KL0laanlXMUJieDY3Z3pLZmp1TlpGRE5sd0s0cFpWdk8wNkxkdjhRZEhrSnF3MWlRCm9CeSszWU1HemU4Z0NFWG5zWnBMT0EvOEtzaC9pNmtaL0YvM0hmVHVRb3QvMWRKdGdpRTRlNEZxZHE2VXZjbHVDUm9Na2IKWmRITDZ3VFJmSTZsNHVLUzBsTVlVd1AzWW5tOTdBc2ZFUW5wc0tNc2dFc2pEMEZLM2QzcURoTnkxRG9velg4NC9lM3NMeQpNdDNCMHFSdlFDaEJBQUFBTldWak1pMTFjMlZ5UUdsd0xURTNNaTB6TVMwek15MHlNalV1WlhVdGJtOXlkR2d0TVM1amIyCjF3ZFhSbExtbHVkR1Z5Ym1Gc0FRSURCQVU9Ci0tLS0tRU5EIE9QRU5TU0ggUFJJVkFURSBLRVktLS0tLQo='
    
    
        Ingress:
    1)IAM folder contains iam policies, roles, role bindings
    airflow-rbac-role-alb-ingress.yml contains cluster role, cluster role binding and service account used by ingress controller to interact with resources in the eks cluster
    
   2) creating the IAM policy will be used by the ingress controller to interact with aws services
    # Create an IAM policy ALBIngressControllerIAMPolicy to alllow ALB makes API calls on your behalf Copy the Policy.Arn value
           "Arn": "arn:aws:iam::339712758698:policy/ALBIngressControllerIAMPolicy",

aws iam create-policy \
    --policy-name ALBIngressControllerIAMPolicy \
    --policy-document file://airflow-materials-aws/section-7/iam/iam-alb-policy.json
    
    arn:aws:iam::339712758698:policy/ALBIngressControllerIAMPolicy    
3)# Create a service account and an IAM role for the pod running AWS ALB Ingress controller
eksctl create iamserviceaccount \
       --cluster=airflow \
       --namespace=kube-system \
       --name=alb-ingress-controller \
       --attach-policy-arn=$PolicyARN \
       --override-existing-serviceaccounts \
       --approve
    