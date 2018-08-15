# Exercises for contrail-multi-cloud

### Exercise 1: Check connectivity between two pods running on a different compute node

1. Get ssh command

```bash
cd /root/multicloud/; wget https://gist.githubusercontent.com/madhukar32/cffa63fd55ec693819dce397dd28f979/raw/4685b0c9150226482a301dd2c602c15f9d16167f/get_ssh_command.sh -O - | bash
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
kubectl exec -t ubuntu-compute-SF ping <IP of the other pod>
```

7. Delete all the resources created

```bash
kubectl delete pod/ubuntu-compute-SF pod/ubuntu-compute-NY
```
### Exercise 2: Create a three tier application for Contrail-Security demo

Note: Below exercise expects that foxyproxy is already setup. Use profile created for froxyproxy to login to contrail webui. Wiki to setup foxyproxy is [here](https://github.com/qarham/cfm-vagrant/blob/master/docs/FoxyProxy-Chrome-Setup.md)

1. Get ssh command

```bash
cd /root/multicloud/; wget https://gist.githubusercontent.com/madhukar32/cffa63fd55ec693819dce397dd28f979/raw/4685b0c9150226482a301dd2c602c15f9d16167f/get_ssh_command.sh -O - | bash
```

2. Login to the contrail controller node (which is also k8s master node)

3. Change to root user

```bash
sudo -i
```

4. Run below commands to bring up three tier app

```bash
# creating pods
kubectl apply -f https://raw.githubusercontent.com/fashaikh7/Contrail-Security/master/3tierApp/web-tier/web-tier-1.yaml
kubectl apply -f https://raw.githubusercontent.com/fashaikh7/Contrail-Security/master/3tierApp/app-tier/app-tier-1.yaml
kubectl apply -f https://raw.githubusercontent.com/fashaikh7/Contrail-Security/master/3tierApp/database-tier/database-tier-1.yaml

#creating kubernetes network policies
kubectl apply -f https://raw.githubusercontent.com/fashaikh7/Contrail-Security/master/policy-web-to-app.yml

https://raw.githubusercontent.com/fashaikh7/Contrail-Security/master/policy-app-to-db.yml
```

5. Verifying status of pods and networkpolicies

```bash
kubectl get pods -o wide
kubectl get networkpolicies
```

6. Get service IP of app svc. In this case `10.99.21.13` is app svc

```bash
root@controller-contrail-TX:~# kubectl get svc | grep app
app-svc-1        ClusterIP   10.99.21.13    <none>        3000/TCP   9h
```

7. To verify working of the app, run below commands. Replace `10.99.21.13`, with your service IP

```bash
kubectl exec -t web-pod-1 --  curl http://10.99.21.13:3000
```
If you get an html page output then it means that three tier is working fine


8. Copy public keys from the remote host to your laptop

```bash
scp -r root@<remote_ip>:~/<userid>/contrail-multi-cloud/keys ~/Downloads/
```

9. Check if ssh-agent is up and running

```bash
ssh-add -l
Could not open a connection to your authentication agent.
```
Above output means that ssh-agent is not running in that case run the below command

```bash
eval `ssh-agent -s`
```

10. Add keys copied from remote server to ssh-agent. where contrail-multicloud-key-4686 is my private key, every setup will have a different key

```bash
ssh-add ~/Downloads/keys/contrail-multicloud-key-4686
```

11. Login to the the vrouter gateway node using the public IP. Stdout of ./deploy.sh should have public_ip details for vrouter gateway node

```bash
ssh root@<public-gw-ip> -D 1080
```

12. Enable Session Export Rate

Contrail security visualization relies on session statistics which by default are disabled. The knob can be enabled by the following step:

* Go to Configure> Forwarding Options> Edit
* Set "Session Export Rate/secs" to "-1"

![alt text][export_rate]
[export_rate]: https://github.com/fashaikh7/Images/raw/master/SetSessionExportRate.png "Export Rate"

13. Validation of Traffic via Visualization

Visualization is perhaps the most important pillar of Contrail Security. This gives detailed view of all traffic (tagged or untagged) on the cluster and helps administrators identify compliant and non-compliant flows.

* Got to Monitor> Security> Traffic Groups> * Select "Kube"

![alt text][visualization]

* Single click on the traffic arc gives session type/port along with policy name

![alt text][traffic_arc_single_click]

* Double click on the traffic arc gives local IP of the application

![alt text][traffic_arc_double_click]

* One can further drill down to find remote IP

![alt text][remote_ip]

[visualization]: https://github.com/fashaikh7/Images/raw/master/Visualization-K8-1.png "Visualization"
[traffic_arc_single_click]: https://github.com/fashaikh7/Images/raw/master/Visualization-K8-2.png "Single Click on Traffic Arc"
[traffic_arc_double_click]: https://github.com/fashaikh7/Images/raw/master/Visualization-K8-3.png "Double click on traffic arc"
[remote_ip]: https://github.com/fashaikh7/Images/raw/master/Visualization-K8-4.png "Remote IP details"


14. Filtering and capturing relevant data

There is lots of good information that can be gathered by applying filters and knowing where to get application data when you need it. Everything is very intuitive but in this section we'll show some snapshots.

* Security Dashboard gives you top applications on your system along with allow/deny ACL hits and top VMI's

![alt text][vmis]

* Time range filter allows you to gather historic data

![alt text][time_range]

* Filtering for your environment (site, deployment, tier etc)

![alt text][filter]

* Getting end point statistics

![alt text][ep_stat]

[vmis]: https://github.com/fashaikh7/Images/raw/master/SecurityDashboard.png "more details"
[time_range]: https://github.com/fashaikh7/Images/raw/master/CapturingHistoricData.png "Time Range"
[filter]: https://github.com/fashaikh7/Images/raw/master/FilteringForGranularity.png "Filter"
[ep_stat]: https://github.com/fashaikh7/Images/raw/master/EndpointStatistics.png "Endpoint statistics"
