# install-ibm-cloud-private-3.1.2-ce-ubuntu18
Install IBM Cloud Private 3.1.2 Community Edition on Ubuntu 18

# Installing IBM Cloud Private (CE) on Ubuntu 18

```console
$ docker pull ibmcom/icp-inception:3.1.2
```
```console
$ sudo mkdir /opt/ibm-cloud-private-3.1.2 && cd /opt/ibm-cloud-private-3.1.2

$ sudo docker run -e LICENSE=accept \
   -v "$(pwd)":/data ibmcom/icp-inception:3.1.2 cp -r cluster /data
```

## Edit config.yaml
```yaml
default_admin_user: admin
default_admin_password: admin
password_rules:
 - '(.*)'
ansible_user: demo
ansible_ssh_pass: passw0rd
ansible_become: true
ansible_become_password: "{{ ansible_ssh_pass }}"
ansible_ssh_common_args: "-oPubkeyAuthentication=no"
ansible_python_interpreter: /usr/bin/python3
```

## Edit hosts file

## Install
```console
$ sudo docker run --net=host -t -e LICENSE=accept \
 -v "$(pwd)":/installer/cluster ibmcom/icp-inception:3.1.2 install
 ```
 ## Login
 ```console
$ cloudctl login -a https://172.16.1.1:8443 --skip-ssl-validation -u admin -p admin -c id-mycluster-account -n default
```
## Helm init
```console
$ helm init --client-only
$ helm version --tls
```
## Connection script
```console
$ tee connecticp.sh <<EOF
cloudctl login -a https://172.16.1.1:8443 --skip-ssl-validation -u admin -p admin -c id-mycluster-account -n default
EOF

$ sudo chmod +x connecticp.sh

$ ./connecticp.sh
```
## Jenkins (test deploy)
```console
$ helm repo add ibm-charts https://raw.githubusercontent.com/IBM/charts/master/repo/stable/
$ helm repo update
```
```console
$ sudo mkdir -p /mnt/data
$ sudo chmod 777 -R /mnt/data
```
```yaml
$ cat <<EOF | kubectl create -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-release-jenkins
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  hostPath:
    path: /mnt/data/my-release-jenkins
EOF
```
```console
$ helm install --name my-jenkins-release ibm-charts/ibm-jenkins-dev --tls
```
```console
$ kubectl get pods
```
Verify pods running 1/1
```console
$ export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services my-jenkins-release-ibm-j)
$ echo http://172.16.1.1:$NODE_PORT/login
```
Access the resulting URL via the browser: admin/admin

Verify persistent data by browsing cd /mnt/data

## Delete Jenkins
```console
$ helm delete --purge my-jenkins-release --tls
$ kubectl delete pv my-release-jenkins
$ cd /mnt/data
$ sudo rm -r my-release-jenkins
```
