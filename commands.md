# Pod
### Create pods from the yaml
```
kubectl create -f pod-definition.yml
```

# Service
### command to start the service
```
kubectl create -f service-definition.yml
```

### List the services
```
kubectl get services
```

### access the app
```
curl http://192.168.1.2:30008
```

## Troubleshooting:
1. Make sure that the service is pointing to correct pod endpoint
- Get the IP of the pod using the command
```
k get pod myapp-pod -o wide

NAME        READY   STATUS    RESTARTS   AGE   IP           NODE                                       NOMINATED NODE   READINESS GATES
myapp-pod   1/1     Running   0          16m   10.124.2.6   gke-cluster-1-e2-small-vms-72805f24-xml8   <none>           <none>
```

- Describe the service and check if the pod’s ip and port are correct

```
k describe service myapp-service

Name:                     myapp-service
Namespace:                default
Labels:                   <none>
Annotations:              cloud.google.com/neg: {"ingress":true}
Selector:                 app=myapp
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       34.118.232.213
IPs:                      34.118.232.213
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  30008/TCP
Endpoints:                10.124.2.6:80
Session Affinity:         None
External Traffic Policy:  Cluster
Internal Traffic Policy:  Cluster
Events:                   <none>
```

This makes sure that the service is pointing to correct pod endpoint.

2. Firewall Rules
Check if the firewall rules are correct to access the port 30008 from external systems.

```
 gcloud compute firewall-rules list
NAME: default-allow-icmp
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 65534
ALLOW: icmp
DENY: 
DISABLED: False

NAME: default-allow-internal
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 65534
ALLOW: tcp:0-65535,udp:0-65535,icmp
DENY: 
DISABLED: False

NAME: default-allow-rdp
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 65534
ALLOW: tcp:3389
DENY: 
DISABLED: False

NAME: default-allow-ssh
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 65534
ALLOW: tcp:22
DENY: 
DISABLED: False

NAME: gke-cluster-1-f400913e-all
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: ah,sctp,tcp,udp,icmp,esp
DENY: 
DISABLED: False

NAME: gke-cluster-1-f400913e-exkubelet
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: 
DENY: tcp:10255
DISABLED: False

NAME: gke-cluster-1-f400913e-inkubelet
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 999
ALLOW: tcp:10255
DENY: 
DISABLED: False

NAME: gke-cluster-1-f400913e-vms
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: icmp,tcp:1-65535,udp:1-65535
DENY: 
DISABLED: False
```

If not, add a rule
```
gcloud compute firewall-rules create allow-nginx-nodeport \
    --allow tcp:30008 \
    --source-ranges=0.0.0.0/0 \
    --description="Allow traffic to nginx NodePort service"


Creating firewall...working..Created [https://www.googleapis.com/compute/v1/projects/learn-kubenetes-479606/global/firewalls/allow-nginx-nodeport].
Creating firewall...done.                                                
NAME: allow-nginx-nodeport
NETWORK: default
DIRECTION: INGRESS
PRIORITY: 1000
ALLOW: tcp:30008
DENY: 
DISABLED: False
```

After setting the firewall, I was able to curl the nginx using the clusterIP
`curl http://35.232.19.127:30008`
