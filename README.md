# k8s-resources

This contains manifest files that we submit to APIServer

NODE IP address - Publicly routable external IP Address of ay worker node or k8s master node
Node Port - We get node port by exposing any pod (eg: nginx pod) using node port service.
POD IP address - Every pod inside a k8s cluster have unique IP address
PORT - Ports uniquely identifies the containers inside a pod which shares the same namespace i.e ipaddress

Inter pod communication: How two pods communicates each other inside cluster. All pod ip addresses are fully routable on the pod network inside k8s cluster

Pod network plugin - Flannel. When you configure this every pod inside cluster gets it's own ip address.

Intra pod networking: Containers within the pod communicate each other using shared localhost interface

kubectl create -f nginx-pod.yaml / kubectl apply -f nginx-pod.yaml
kubectl get pods -o wide - to check the ip address
kubectl get pod nginx-pod -o yaml - to check pod configuration in yaml format
kubectl describe pod nginx-pod
kubectl delete pod nginx-pod

cat <<EOF > /usr/share/nginx/html/test.html
<!DOCTYPE html>
<html>
<head>
<title>Testing..</title>
</head>
<body>
<h1>Hello, Mubernetes...!</h1>
<h2>Congratulations you passed :)</h2>
</body>
</html>
EOF

When a service is exposed on NodePort - it means that it is accecible from external network.

kubectl expose pod nginx-pod --type=NodePort --port=80
kubectl get svc
NAME                    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes              ClusterIP   10.244.0.1     <none>        443/TCP        108d
nginx-pod               NodePort    10.244.19.9    <none>        80:32072/TCP   5s

kubectl describe svc nginx-pod - this would give the nodeport on which the service is exposed(here 32072)
Name:                     nginx-pod
Namespace:                default
Labels:                   app=nginx
                          tier=dev
Annotations:              <none>
Selector:                 app=nginx,tier=dev
Type:                     NodePort
IP:                       10.244.19.9
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  32072/TCP
Endpoints:                10.244.64.124:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>

To access this you need to use <NodeIP>:<NodePort> - We now know NodePort. What is NodeIP? NodeIP is an external IP address of either master node ore any worker node. Here I'm accessing it from my VMs ip address as I'm using single node k8s cluster. 10.211.1.130:32072 -> this exposes default nginx page. 10.211.1.130:32072/test.html would give us the page we've deployed inside nginx server.

The web page we could access externally from internet by exposing the pod using nodeport service.

How to access the test.html webpage internally from the k8s cluster. For that we need PodIP and port number.

kubectl describe svc nginx-pod - this would give us the PodIP address (Endpoints-10.244.64.124:80)
Name:                     nginx-pod
Namespace:                default
Labels:                   app=nginx
                          tier=dev
Annotations:              <none>
Selector:                 app=nginx,tier=dev
Type:                     NodePort
IP:                       10.244.19.9
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  32072/TCP
Endpoints:                10.244.64.124:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:

curl http://10.244.64.124:80/test.html - This will be accecible from master and all the worker nodes

The web page we could access internally from k8s cluster nodes by exposing the pod using nodeport service.
The web page we created accecible externally from internet, internally from k8s cluster

kubectl delete svc nginx-pod
kubectl delete pod nginx-pod
