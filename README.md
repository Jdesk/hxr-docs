## Deploy Kubernetes Cluster and Windows Server Pods

1.  From the azure cli, your first step will be to run the following command which will build your AKS (Azure Kubernetes Service) Cluster with all the requirements to run Kubernetes on Windows Nodes (so you can run windows containers).  

```
az aks create \
    --resource-group hxr-win-k8s \
    --name hxr-win-k8s \
    --node-count 1 \
    --enable-addons monitoring \
    --kubernetes-version 1.14.0 \
    --generate-ssh-keys \
    --windows-admin-password hxr14328jkUi! \
    --windows-admin-username azureuser \
    --enable-vmss \
    --network-plugin azure
```
2.  Next you need to create a Windows based Nodepool (this is where all windows containers will be scheduled.  You can do this by running the following command from the azure cli:

```
az aks nodepool add \
    --resource-group hxr-win-k8s \
    --cluster-name hxr-win-k8s \
    --os-type Windows \
    --name npwin \
    --node-count 1 \
    --kubernetes-version 1.14.0
```
3.  You now need to login to your newly created kubernetes cluster.  You do this by running the following command from the azure cli:

```
az aks get-credentials --resource-group hxr-win-k8s --name hxr-win-k8s
```
4.  Next you need to create a file called windows-webserver.yaml (or anything you wish to call it).   The contents of the file can be seen here in this gist I created:  [windows-webserver.yaml](https://gist.github.com/Jdesk/5b75da7fc114a5cd780270b6affc444b)
5. Make sure windows-webserver.yaml is saved in your $PATH and then run the following command:
```
kubectl apply -f windows-webserver.yaml
```

Please note that you must have kubectl installed locally in order to interact with your kubernetes cluster.   

Instructions for installing kubectl on windows can be found here:  [Install Kubectl on Windows](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-binary-with-curl-on-windows)

6.  To see the status of your pod run the following command:
```
kubectl get pods
```
You will see a status of "running" once it is ready.  Also take note of the name of the pod for the next step which will be something like this:  ***win-webserver-8648d6f7b8-qd6h6***

7.  Make sure your zipped archive of oos is in your $PATH.   Then you will copy this zip file to the pod by running the following command:
```
kubectl cp oos.zip default/win-webserver-8648d6f7b8-qd6h6:oos.zip
```

8.  Now we want to go into the windows server container by running the following command which will open a powershell within our container:

```
kubectl exec -it win-webserver-8648d6f7b8-qd6h6 powershell
```

9. Once you are in the container, you should see the the following after running ```ls```
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        5/31/2019   7:49 PM                inetpub
d-----        5/31/2019   7:48 PM                oss
d-r---        5/18/2019   6:41 PM                Program Files
d-----        5/18/2019   6:39 PM                Program Files (x86)
d-r---        5/31/2019   7:58 PM                Users
d-----        5/31/2019   7:39 PM                var
d-----        5/31/2019   7:49 PM                Windows
-a----        9/15/2018   9:42 AM           5510 License.txt
-a----        5/30/2019   8:09 PM      686912774 oss.zip

Notice our oss.zip file is present.  

10.  Unzip the archive by running the following command:

```
Expand-Archive -LiteralPath C:\oss.zip -DestinationPath C:\oss
```
Now you can ```cd oss``` and you will see the oss filesystem there.   

This is where I ran into issues as I was not able to install from powershell.  Perhaps you will have a bit more luck!


