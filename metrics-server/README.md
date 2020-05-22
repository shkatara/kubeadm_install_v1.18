
*DO ONLY WHEN THE CLUSTER IS INSTALLED*

Once the cluster is installed, to set the metrics up and running, 

1. git clone https://github.com/kubernetes-sigs/metrics-server

2. cd metrics-server/manifests/base

3. kubectl create -f rbac.yml

###IMPORTANT###

4. Change the listening arguments of the metrics-server. Unde the args of metrics-server container: \
      args: \
        - --kubelet-insecure-tls \
        - --cert-dir=/tmp \
        - --secure-port=4443 \
        - --metric-resolution=10s \
        - --kubelet-preferred-address-types=InternalIP \ 
      ports: \
      . \
      . \
      . 


5. After the changes are made, create the deployment

kubectl create -f deployment.yml 


6. kubectl create -f service.yml

