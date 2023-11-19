# Document: SDS project
## How to set up your Kubernetes cluster
### Master (VM ubuntu)
#### Master 1
1. Disable the Uncomplicated Firewall (UFW)  
	``ufw disable``
2. Install k3s script and initialize the cluster for the server (master) node:  
  ``curl -sfL https://get.k3s.io/ | K3S_TOKEN=regsds sh -s - server --cluster-init --tls-san=192.168.0.105 ``
3. Retrieve the K3S_TOKEN for the agent (to be used in the worker node setup in Step 3):  
``sudo cat /var/lib/rancher/k3s/server/node-token``
4. Identify the IP address of your VM Ubuntu using ``ifconfig`` (to be used in the worker node setup in Step 3).
5. Copy the Kubernetes configuration file from the master node to each worker node. This file is typically located at ~/.kube/config. You can use the Secure Copy (SCP) command to achieve this. Replace [worker-node-ip] with the actual IP address of each worker node:  
``scp ~/.kube/config username@[worker-node-ip]:~/.kube/config``
6. Assign role for each worker node with following command  
``sudo k3s kubectl label node <NODE_NAME> node-role.kubernetes.io/worker=worker``
The result of all node should look like this
![image](https://github.com/RegSDS/regsds-gitops/assets/77723734/27e554ee-b2bc-469b-8ced-70a2397ae02b)

#### Master 2 
1. To be another master, run this following command  
``curl -sfL https://get.k3s.io/ | K3S_TOKEN=<TOKEN> sh -s - server --server https://<MASTER1_IP>:<PORT>``  
The result from this command ``sudo k3s kubectl get node``  should display this  
![image](https://github.com/RegSDS/regsds-gitops/assets/77723734/6e01ee98-261f-43c3-856e-41408f8a7ffb)


### Worker node
1. Access the bootline file using the following command:  
``sudo nano /cmd/bootline.txt``
2. Add the following line at the end of the /boot/cmdline.txt file:  
``cgroup_enable=memory cgroup_memory=1``   
3. Install k3s script for the worker node by running the following command. Replace <YOUR_IP> and <PORT> with the IP address and port of your master node, and <YOUR_TOKEN> with the token obtained from the master node in Step 3 of the master setup:  
``curl -sfL https://get.k3s.io/ | K3S_URL=https://<YOUR_IP>:<PORT>/ K3S_TOKEN=<YOUR_TOKEN> sh - ``
![image](https://github.com/RegSDS/regsds-gitops/assets/88878365/93d7e617-255a-4c69-9692-97f11c5c356a)

4. Wait for the configuration file at ./.kube/config to be copied from the master node (Step 5 of the master setup).  
5. Check if the configuration file has appeared by running the following command:  
``k3s kubectl config view``  
The result should display the Kubernetes configuration.
![image](https://github.com/RegSDS/regsds-gitops/assets/88878365/9b110ae6-964e-45c8-b698-5770db0e70fb)

6. Check if the worker node is connected to the master node by running the following command:  
``sudo k3s kubectl --kubeconfig=./.kube/config get node``  
the result should be similar to this
![image](https://github.com/RegSDS/regsds-gitops/assets/77723734/38a9736c-dcb9-4d94-b97a-42df7c31635f)


## How to deploy your application
1. Clone the Kubernetes configuration repository using the following command  
`git clone https://github.com/RegSDS/regsds-gitops`
2. Apply the Kubernetes configuration by running the following command  
`sudo k3s kubectl apply -f <DEPLOYMENT_FILE_PATH> .`
3. To check the status and location of each pod, use the following command  
`sudo k3s kubectl get pods -o wide`

