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