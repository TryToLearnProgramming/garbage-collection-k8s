# Kuberentes configuration files to remove unused Docker Images from K8s nodes automatically.
Best Use Cases: Deploying the manifest files through pipeline along with other deployments or as pre-configured hook.

**K8s-garbage-collection**

Kubernetes Job manifest files to prune old unused docker images that are left from pervious revisions of deployments/replica-sets.

# Advantages
There are many existing solutions that delete unused docker images from kubernetes nodes but most of them need to have deployments/jobs/cronjobs runnning continuously or is only deployed when the nodes already uses a certain high percentage of disk space. This can cause a lot of unecessary resource usage and increase in costs specially in cloud deployments.

This deployment has certain advantages over the existing solutions:
> 1. Can be run manually whenever needed just by kubectl apply.
> 2. No special privileges needed.
> 3. All garbage-collection resources gets removed after the images are deleted mith a maximum limit of 5 minutes. This helps in not putting extra burden on the k8s nodes.
> 4. No manual configurations needs to be done (save minor one detailed below).
> 5. Optimized for deploying through CI/CD pipelines and pull based tools like ArgoCD with pre-configured hooks.

----------------------------------------------------------------------------------------------------------------------------------

**kubernetes-docker Branch**

For kubernetes version <= 1.22 which runs docker engine as the container runtime.

> The manifest file garbage-collection.yaml mounts the unix socket of the docker daemon in the pods ran through the k8s job. Docker prune command is used to remove all unused (tagged & dangling) images.

----------------------------------------------------------------------------------------------------------------------------------

**kubernetes-containerd Branch**

For kubernetes version >= 1.23 which runs containerd as the container runtime.

> The manifest file garbage-collection.yaml mounts the unix socket of the containerd daemon in the pods ran through the k8s job. crictl command-line tool is used to remove all unused (tagged & dangling) images.

----------------------------------------------------------------------------------------------------------------------------------

**Pod Anti-affinity & PriorityClass in k8s Job**

The kubernetes job needs to run in all the nodes to clean all the nodes of unused images. This is done through :-
1. Pod Anti-affinity in kubernetes job
2. High priorityClass for the deployment pods

Both configurations are used to ensure the pods for the job run in all the nodes even if the no of pods is saturated for a particular node.

----------------------------------------------------------------------------------------------------------------------------------

**What changes need to be made in the files ?**

1. Change Kubernetes job spec of:
      completions: <value>
      parallelism: <values>
   Replace <value> to the number of nodes your Cluster is running. You can also put values higher than the current number of nodes to take account of cluster autoscaling.

----------------------------------------------------------------------------------------------------------------------------------

# How to run ?

Run the mainifest files situated inside manifests dir as:

kubectl apply -f manifests/

----------------------------------------------------------------------------------------------------------------------------------

**Why not use Daemonset instead of Jobs ?**

 Daemonset has the advantage of deploying pods in all the nodes and might have made the deployment simpler, but the biggest dis-advantage is that daemonset is persistent.
 Therefore one has to delete the daemonset deployment manually after the docker images are removed. This can be troublesome to configure when running through:
 > when using CI/CD pipelines it will need manual intervention fer deleting the daemonsets
 > when using tools like ArgoCD the manifest files for daemonsets need to be removed from the source path once the job is done
 
 Since using daemonsets instead of k8s jobs needs manual support, kubernetes jobs is preferred as the deployment and cleaning process is completely automated.

----------------------------------------------------------------------------------------------------------------------------------


