# Exercises for contrail-multi-cloud

### Exercise 1: Check connectivity between two pods running on a different compute node

1. Login to the contrail controller node (which is also k8s master node) using ssh command given at the end of stdout of deploy.sh

2. Change to root user

```bash
sudo -i
```

3. Creating two pods, one on compute of SanFrancisco VNET and another on compute of LosAngeles VNET

```bash
kubectl apply -f https://raw.githubusercontent.com/madhukar32/contrail-multi-cloud-workshop/master/exercise/ubuntu/pod-compute-contrail-SF.yaml

kubectl apply -f https://raw.githubusercontent.com/madhukar32/contrail-multi-cloud-workshop/master/exercise/ubuntu/pod-compute-contrail-NY.yaml
```

4. Check if pods are up and running

```bash
kubectl get pods -o wide
```

5. Login to one pod and ping another pod

```bash
kubectl exec -t ubuntu-compute-SF ping <IP of the other pod>
```

6. Delete all the resources created

```bash
kubectl delete pod/ubuntu-compute-SF pod/ubuntu-compute-NY
```
### Exercise 2: Create a three tier application for Contrail-Security demo

Note: Below exercise expects that foxyproxy is already setup. Use profile created for froxyproxy to login to contrail webui. Wiki to setup foxyproxy is [here](https://github.com/qarham/cfm-vagrant/blob/master/docs/FoxyProxy-Chrome-Setup.md)

1. Login to the contrail controller node (which is also k8s master node) using ssh command given at the end of stdout of deploy.sh

2. Change to root user

```bash
sudo -i
```

3. Run below commands to bring up three tier app

```bash
# creating pods
kubectl apply -f https://raw.githubusercontent.com/fashaikh7/Contrail-Security/master/3tierApp/web-tier/web-tier-1.yaml
kubectl apply -f https://raw.githubusercontent.com/fashaikh7/Contrail-Security/master/3tierApp/app-tier/app-tier-1.yaml
kubectl apply -f https://raw.githubusercontent.com/fashaikh7/Contrail-Security/master/3tierApp/database-tier/database-tier-1.yaml

#creating kubernetes network policies
kubectl apply -f https://raw.githubusercontent.com/fashaikh7/Contrail-Security/master/policy-web-to-app.yml

https://raw.githubusercontent.com/fashaikh7/Contrail-Security/master/policy-app-to-db.yml
```

4. Verifying status of pods and networkpolicies

```bash
kubectl get pods -o wide
kubectl get networkpolicies
```

5. Get service IP of app svc. In this case `10.99.21.13` is app svc

```bash
root@controller-contrail-TX:~# kubectl get svc | grep app
app-svc-1        ClusterIP   10.99.21.13    <none>        3000/TCP   9h
```

6. To verify working of the app, run below commands. Replace `10.99.21.13`, with your service IP

```bash
kubectl exec -t web-pod-1 --  curl http://10.99.21.13:3000
```
If you get an html page output then it means that three tier is working fine


7. Copy public keys from the remote host to your laptop

```bash
scp -r root@<remote_ip>:~/<userid>/contrail-multi-cloud/keys ~/Downloads/
```

8. Check if ssh-agent is up and running

```bash
ssh-add -l
Could not open a connection to your authentication agent.
```
Above output means that ssh-agent is not running in that case run the below command

```bash
eval `ssh-agent -s`
```

9. Add keys copied from remote server to ssh-agent. where contrail-multicloud-key-4686 is my private key, every setup will have a different key

```bash
ssh-add ~/Downloads/keys/contrail-multicloud-key-4686
```

10. Login to the the vrouter gateway node using the public IP. Stdout of ./deploy.sh should have public_ip details for vrouter gateway node

```bash
ssh root@<public-gw-ip> -D 1080
```
