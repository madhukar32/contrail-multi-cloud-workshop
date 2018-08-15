# Exercises for contrail-multi-cloud

### Exercise 1: Check connectivity between two pods running on a different compute node

1. Get ssh command

```bash
cd /root/multicloud/; wget https://gist.githubusercontent.com/madhukar32/cffa63fd55ec693819dce397dd28f979/raw/a38672a9da120de0624c724a99e0b48d9136bf7f/get_ssh_command.sh -O - | bash
```

2. Login to the contrail controller node (which is also k8s master node)

3. Change to root user

```bash
sudo -i
```

4. Creating two pods, one on compute of SanFrancisco VNET and another on compute of LosAngeles VNET

```bash
kubectl apply -f https://raw.githubusercontent.com/madhukar32/contrail-multi-cloud-workshop/master/exercise/ubuntu/pod-compute-contrail-SF.yaml

kubectl apply -f https://raw.githubusercontent.com/madhukar32/contrail-multi-cloud-workshop/master/exercise/ubuntu/pod-compute-contrail-NY.yaml
```

5. Check if pods are up and running

```bash
kubectl get pods -o wide
```

6. Login to one pod and ping another pod

```bash
kubectl exec -t ubuntu-compute-sf ping <IP of ny pod>
```

7. Delete all the resources created

```bash
kubectl delete pod/ubuntu-compute-sf pod/ubuntu-compute-ny
```
### Exercise 2: Create a three tier application for Contrail-Security demo (Bonus)

Note: Below exercise expects that foxyproxy is already setup. Use profile created for froxyproxy to login to contrail webui. Wiki to setup foxyproxy is [here](https://github.com/qarham/cfm-vagrant/blob/master/docs/FoxyProxy-Chrome-Setup.md)



1. Run below commands to bring up three tier app

```bash
# creating pods
kubectl apply -f https://raw.githubusercontent.com/fashaikh7/Contrail-Security/master/web-pod.yaml
kubectl apply -f https://raw.githubusercontent.com/fashaikh7/Contrail-Security/master/app-pod.yaml
kubectl apply -f https://raw.githubusercontent.com/fashaikh7/Contrail-Security/master/db-pod.yaml

#creating kubernetes network policies
kubectl apply -f https://raw.githubusercontent.com/fashaikh7/Contrail-Security/master/policy-web-to-app.yml

kubectl apply -f https://raw.githubusercontent.com/fashaikh7/Contrail-Security/master/policy-app-to-db.yml
```

2. Verifying status of pods and networkpolicies

```bash
kubectl get pods -o wide
kubectl get networkpolicies
```

3. Generate traffic from web to app and app to db

```bash
kubectl exec -t web --  ping <ip of app pod>
kubectl exec -t app -- ping <ip of db pod>
```

4. Checkout the workshop doc to copy keys from your remote-server to your laptop

5. Check if ssh-agent is up and running

```bash
ssh-add -l
Could not open a connection to your authentication agent.
```

6. Above output means that ssh-agent is not running in that case run the below command

```bash
eval `ssh-agent -s`
```

7. Add keys copied from remote server to ssh-agent. In the below example, contrail-multicloud-key-4686 is my private key. Every setup will have a different key.

```bash
ssh-add keys/contrail-multicloud-key-4686
```

8. Getting to contrail controller webui

```bash
ssh -L localhost:10000:<controller-contrail-sf>:8143 ubuntu@<public-vrouter-gw>
```
9. Login into your contrail web-ui by entering the following on the chrome browser:

https://localhost:8143       admin/contrail123

10. Enable Session Export Rate

Contrail security visualization relies on session statistics which by default are disabled. The knob can be enabled by the following step:

* Go to Configure> Forwarding Options> Edit
* Set "Session Export Rate/secs" to "-1"

![alt text][export_rate]

[export_rate]: https://github.com/fashaikh7/Images/raw/master/SetSessionExportRate.png "Export Rate"

11. Acknowledged all the alarms by clicking on Alarms > Select > Check > Confirm - Hit Cancel to exit out

![alt text][alarms]

12. Go to Monitor > Security > Traffic Groups > all projects (drop down)

![alt text][dashboard]

13. Click on web to app arch and validate traffic direction and policy - double to click the arch to view session and local/remote IPs

![alt text][traffic]

14. Click on app to db arch and validate traffic direction and policy - double to click the arch to view session and local/remote IPs

![alt text][app-to-db]

[alarms]: https://github.com/fashaikh7/Images/blob/master/Alarms.png
[dashboard]: https://github.com/fashaikh7/Images/blob/master/Dashboard.png
[traffic]: https://github.com/fashaikh7/Images/blob/master/Traffic-web-to-app.png
[app-to-db]: https://github.com/fashaikh7/Images/blob/master/App-to-db.png
