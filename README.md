# k3s_kubevip_via_k3sup

vagrant-libvirt setup for a k3s cluster without servicelb, so kube-vip can manually be installed

This is loosely based on what Adrian Goins showed in his wonderful [Youtube video on HA k3s](https://cncn.io/2021/03/ha-k3s-with-kube-vip-and-metallb/).

This setup creates three k3s server VMs and one VM as k3s agent (aka worker) and prepares a k3sup installation script via Ansible. This script will install k3s WITHOUT `servicelb`, so you can install kube-vip manually (see below).

Default OS is openSUSE Leap 15.2, but that can be changed in the Vagrantfile. Same holds true for the sizing of the machines.

## Vagrant

1. You need vagrant obviously. And Ansible. And k3sup.
2. Fetch the box, per default this is `opensuse/Leap-15.2.x86_64`, using `vagrant box add opensuse/Leap-15.2.x86_64`.
3. Make sure the git submodules are fully working by issuing `git submodule init && git submodule update`
4. Run `vagrant up`
5. Inspect and run the `k3sup_installation.sh` script
6. Run `kubectl --kubeconfig kubeconfig_k3s_kubevip_via_k3sup.yaml get nodes` and you should see your server and agent nodes.
7. Install kube-vip (see below)
8. Party!

## kube-vip installation

Adrian Goins has a very nice description on what to do [here](https://gitlab.com/monachus/channel/-/tree/master/resources/2021.02-ha-k3s-kube-vip-metallb). Only make sure to use the latest version of kube-vip (3.4 as of this writing) and not the 3.2 that Adrian used.

The steps to install kube-vip are:
1. Select a suitable IP to be used as virtual IP
2. Install the first server node
3. Install the kube-vip-rbac yaml file (via kubectl or by putting it into `/var/lib/rancher/k3s/server/manifests/` on the first k3s server)
4. Create the actual kube-vip.yaml file and apply it (via kubectl or by putting it into `/var/lib/rancher/k3s/server/manifests/` on the first k3s server)
5. Modify the kubeconfig that k3sup created, point it to the new virtual IP and make sure it works
6. Join other server or agent nodes (pointing them at the new virtual IP to register)

After you run `vagrant up`, Ansible will prepare the file `k3sup_installation.sh` containing installation commands for k3sup. Those contain the proper IP addresses of all your nodes. *BUT* Ansible cannot freely decide which IP to use a virtual IP. So you need to replace `KUBE_VIP_GOES_HERE` with an actual (free and reserved!) IP from your libvirt network.

The command to install the first server looks like this:
```bash
k3sup install \
--host=192.168.121.20 \
--user=vagrant \
--ssh-key ./ssh_key_k3s_kubevip_via_k3sup \
--local-path=kubeconfig_k3s_kubevip_via_k3sup.yaml \
--context k3s_kubevip_via_k3sup \
--tls-san KUBE_VIP_GOES_HERE \
--cluster \
--k3s-extra-args="--disable servicelb --node-taint node-role.kubernetes.io/master=true:NoSchedule"
```

## Cleaning up

When tearing down the the vagrant VMs using `vagrant destroy`, the files created by Ansible are not being removed.

You can do this (using Ansible...) by running `ansible-playbook ansible/playbook-DELETE_everything_to_start_from_scratch.yml`.

This will DELETE all files without asking for confirmation!

## Creating additional agent nodes

You can modify the Vagrantfile to create additional agent nodes by tweaking two lines.

1. Setting the number of agents (in this example to `2`):

```
  ###################################################################################
  # define number of agents
  W = 2
```

2. Adding the additional agent nodes to the `ansible_groups` line:
```
      ansible.groups = {
        "k3sservers"  => [ "k3sserver1", "k3sserver2", "k3sserver3" ],
        "k3sagents"   => [ "k3sagent1", "k3sagent2" ]
      }
```

## Creating additional server nodes

You can modify the Vagrantfile to create additional server nodes by tweaking two lines similar to adjusting the agent number described above.
