# WIPP Deployment on a single-node cluster - Complete installation

WIPP relies on Kubernetes (k8s) to run. If you have Kubernetes cluster available or know how to create one, just use the command below to deploy WIPP applications:

```
kubectl apply -f wipp-single-node.yaml
```

## Installation

If you do not have Kubernetes cluster or don't know how to create one, follow the instructions below for popular operating systems to get the working local WIPP installation on your computer:

- [macOS (with Docker Desktop)](#macOS-(with-Docker-Desktop))
- [macOS (with Multipass+microk8s)](#macOS-(with-Multipass+microk8s))
- [Linux (Multipass+microk8s)](#Linux-(Multipass+microk8s))
- [Windows 10 (Multipass+microk8s)](#Windows-10-(Multipass+microk8s))

### Volume size considerations

Kubernetes require to specify the volume size before deploying apps. `wipp-single-node.yaml` contains some reasonable defaults for testing WIPP on a single computer/laptop, which you might want to change to better suit your case. Below is the list of volumes with default size and reference to config, so you can change them before deploying.

| Volume name                                         | Purpose                                    | Default size |
|-----------------------------------------------------|--------------------------------------------|--------------|
| [`mongo-pv-claim`](wipp-single-node.yaml#L154)     | MongoDB database for WIPP                  | 1Gi          |
| [`wipp-pv-claim`](wipp-single-node.yaml#L554)      | WIPP storage for images and data           | 20Gi         |
| [`notebooks-pv-claim`](wipp-single-node.yaml#L989) | Shared storage for all Notebook users      | 5Gi          |
| [`claim-{username}`](wipp-single-node.yaml#L532)   | Individual storage for each Notebook users | 1Gi          |

### macOS (with Docker Desktop)

1. Download and install Docker Desktop. Follow instructions here: https://docs.docker.com/docker-for-mac/install/ You will need to create free Docker ID to get access to .dmg download
2. Once installed, click on the Docker logo and choose **Preferences** → **Advanced**. Depending on your Mac configuration, choose the appropriate amount of CPU, RAM and disk available for WIPP (and all Docker containers).
3. After that go to **Preferences** → **Kubernetes** → **Enable Kubernetes**. Wait for the *Kubernetes is running* status and the green indicator in the bottom right corner of the window.
4. Find the ip address of your Mac on the local network: click **Network** <img src="https://icon-library.net/images/mac-wifi-icon/mac-wifi-icon-23.jpg" width="10"> → **Open Network Preferences …** and copy the IP address from this line: *Wi-Fi is connected to <WIFI_NAME> and has the IP address x.x.x.x*.
5. Replace all occurences of `localhost` in `wipp-single-node.yaml` to the IP address from previous step: `x.x.x.x`.
6. Deploy WIPP. Open the terminal and run:

```
kubectl apply -f wipp-single-node.yaml
```
7. In browser, access the apps at the following addresses:
   * WIPP: x.x.x.x:32001 
   * Argo: x.x.x.x:32002
   * Notebooks: x.x.x.x:32003
   * Plots: x.x.x.x:32004


### macOS (with Multipass+microk8s)

1. Download and install Multipass for Mac: https://multipass.run/#install Multipass creates on-demand Linux VMs and provides an easy way to run Kubernetes using microk8s. 
2. Once installed, open the terminal and create VM:

```
multipass launch --name wipp --cpus 4 --mem 8G --disk 100G ubuntu
```

Depending on your Mac configuration, choose the appropriate amount of CPU, RAM and disk available for WIPP.
3. Install and start microk8s:
```
multipass exec wipp -- sudo apt update
multipass exec wipp -- sudo apt install docker.io
multipass exec wipp -- sudo snap install microk8s --classic
multipass exec wipp -- sudo iptables -P FORWARD ACCEPT
multipass exec wipp -- sudo usermod -a -G microk8s multipass
multipass exec wipp -- /snap/bin/microk8s.start
multipass exec wipp -- /snap/bin/microk8s.enable rbac
multipass exec wipp -- /snap/bin/microk8s.enable dns
multipass exec wipp -- /snap/bin/microk8s.enable storage
```
4. Find the IP of Multipass VM:
```
multipass info wipp | grep IP
> IPv4:           x.x.x.x
```
5. Replace all occurences of `localhost` in `wipp-single-node.yaml` to the IP address from previous step: `x.x.x.x`.
6. Copy the Kubernetes config and deploy WIPP (install `kubectl` if not present: https://kubernetes.io/docs/tasks/tools/install-kubectl/)
```
multipass exec wipp -- /snap/bin/microk8s.config > kubeconfig
kubectl --kubeconfig=kubeconfig apply -f wipp-single-node.yaml
```
7. In browser, access the apps at the following addresses:
   * WIPP: x.x.x.x:32001 
   * Argo: x.x.x.x:32002
   * Notebooks: x.x.x.x:32003
   * Plots: x.x.x.x:32004


### Linux (Multipass+microk8s)

1. Install Multipass from Snap Store:
```
sudo snap install multipass --classic --beta
```
2. Once installed, open the terminal and create VM:
```
multipass launch --name wipp --cpus 4 --mem 8G --disk 100G ubuntu
```
3. Install and start microk8s:
```
multipass exec wipp -- sudo apt update
multipass exec wipp -- sudo apt install docker.io
multipass exec wipp -- sudo snap install microk8s --classic
multipass exec wipp -- sudo iptables -P FORWARD ACCEPT
multipass exec wipp -- sudo usermod -a -G microk8s multipass
multipass exec wipp -- /snap/bin/microk8s.start
multipass exec wipp -- /snap/bin/microk8s.enable rbac
multipass exec wipp -- /snap/bin/microk8s.enable dns
multipass exec wipp -- /snap/bin/microk8s.enable storage
```
4. Find the IP of Multipass VM:
```
multipass info wipp | grep IP
> IPv4:           x.x.x.x
```
5. Replace all occurences of `localhost` in `wipp-single-node.yaml` to the IP address from previous step: `x.x.x.x`.
6. Copy the Kubernetes config and deploy WIPP:
```
multipass exec wipp -- /snap/bin/microk8s.config > kubeconfig
kubectl --kubeconfig=kubeconfig apply -f wipp-single-node.yaml
```
7. In browser, access the apps at the following addresses:
   * WIPP: x.x.x.x:32001 
   * Argo: x.x.x.x:32002
   * Notebooks: x.x.x.x:32003
   * Plots: x.x.x.x:32004


### Windows 10 (Multipass+microk8s)
Make sure you have Windows 10 Pro, Enterprise or Education; Windows 10 Home is not supported.

1. Download and install Multipass for Windows: https://multipass.run/#install Multipass creates on-demand Linux VMs and provides an easy way to run Kubernetes using microk8s. 
2. Enable Hyper-V on Windows: https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v
3. Create Multipass VM:
```
multipass launch --name wipp --cpus 4 --mem 8G --disk 100G ubuntu
```
Depending on your PC configuration, choose the appropriate amount of CPU, RAM and disk available for WIPP.
4. Install and start microk8s:
```
multipass exec wipp -- sudo apt update
multipass exec wipp -- sudo apt install docker.io
multipass exec wipp -- sudo snap install microk8s --classic
multipass exec wipp -- sudo iptables -P FORWARD ACCEPT
multipass exec wipp -- sudo usermod -a -G microk8s multipass
multipass exec wipp -- /snap/bin/microk8s.start
multipass exec wipp -- /snap/bin/microk8s.enable rbac
multipass exec wipp -- /snap/bin/microk8s.enable dns
multipass exec wipp -- /snap/bin/microk8s.enable storage
```
5. Print the Kubernetes config:
```
multipass exec wipp -- /snap/bin/microk8s.config
```
Copy the output of the command to `kubeconfig` file.
6. Find the IP of Multipass VM:
```
multipass info wipp
```
Copy the IP address `x.x.x.x`.

7. Replace all occurences of `localhost` in `wipp-single-node.yaml` to the IP address from previous step: `x.x.x.x`.

8. Deploy WIPP (install `kubectl` if not present: https://kubernetes.io/docs/tasks/tools/install-kubectl/):
```
.\kubectl.exe --kubeconfig=kubeconfig apply -f wipp-single-node.yaml
```
9. In browser, access the apps at the following addresses:
   * WIPP: x.x.x.x:32001 
   * Argo: x.x.x.x:32002
   * Notebooks: x.x.x.x:32003
   * Plots: x.x.x.x:32004

## Teardown

There are couple of important considerations to keep in mind. If you perform the full teardown, all the data generated or uploaded in any of the apps (WIPP, Notebooks, Plots) **will be lost**. It is possible to do a partial teardown when only the apps are deleted but all the data persist, so you can reinstall WIPP again in the future and continue to use it in the state you left it in. If you would like to delete WIPP from your computer, follow the instructions below:

- [when using Docker for Mac](#Teardown-in-Docker-Desktop)
- [when using Multipass+microk8s](#Teardown-in-Multipass)

#### Teardown in Docker Desktop

1. Delete all resources created by WIPP (deployments, services, storage, etc.):
```
kubectl delete -f wipp-single-node.yaml
```
2. (OPTIONAL) Turn of Kubernetes cluster in Docker settings. In  Docker menu go to **Preferences** → **Kubernetes** and uncheck **Enable Kubernetes**.
3. (OPTIONAL) For complete teardown, you can delete Docker Desktop

#### Teardown in Multipass

1. Delete all resources created by WIPP (deployments, services, storage, etc.):
```
kubectl --kubeconfig=kubeconfig delete -f wipp-microk8s.yaml
```
2. (OPTIONAL) Delete Multipass VM:
```
multipass stop wipp
multipass delete wipp
multipass purge
```
3. (OPTIONAL) Delete Multipass from your computer