Here's a GitHub Markdown (`.md`) file containing the information you provided:

```markdown
# Creating and Using a Volume with Azure Disks in Azure Kubernetes Service (AKS)

In this guide, you will learn how to create and use a persistent volume with Azure Disks in an Azure Kubernetes Service (AKS) cluster. A persistent volume represents storage provisioned for use with Kubernetes pods. You can provision it dynamically or statically, and it can be used with one or multiple pods.

## Preconfigured Storage Classes in AKS

Each AKS cluster comes with four precreated storage classes, two of which are configured to work with Azure Disks:

1. **Default Storage Class**: This class provisions a standard SSD Azure Disk. Standard SSDs are cost-effective while providing reliable performance.

2. **Managed-CSI-Premium Storage Class**: This class provisions a premium Azure Disk. Premium disks are SSD-based, offering high performance and low latency, making them ideal for VMs running production workloads. You can also use the managed-csi storage class, which is backed by Standard SSD locally redundant storage (LRS), when using the Azure Disk CSI driver on AKS.

To view the available storage classes in your AKS cluster, use the following command:

```bash
kubectl get sc
```

## Creating a Persistent Volume Claim (PVC)

A Persistent Volume Claim (PVC) automatically provisions storage based on a storage class. You can use one of the precreated storage classes to create either a standard or premium Azure managed disk.

1. Create a file named `azure-pvc.yaml` and copy the following manifest into it. This manifest requests a 5GB disk named `azure-managed-disk` with `ReadWriteOnce` access. The `managed-csi` storage class is specified.

    ```yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
        name: azure-managed-disk
    spec:
      accessModes:
      - ReadWriteOnce
      storageClassName: managed-csi
      resources:
        requests:
          storage: 5Gi
    ```

2. Create the persistent volume claim using the following `kubectl` command and specify your `azure-pvc.yaml` file:

    ```bash
    kubectl apply -f azure-pvc.yaml
    ```

## Using the Persistent Volume

After creating the persistent volume claim, you need to verify that it has a status of `Pending`, indicating that it's ready to be used by a pod.

To check the status of the PVC, use the following command:

```bash
kubectl describe pvc azure-managed-disk
```

## Creating a Pod with the Persistent Volume

1. Create a file named `azure-pvc-disk.yaml` and copy the following manifest into it. This manifest creates a basic NGINX pod named `mypod` that uses the persistent volume claim named `azure-managed-disk` to mount the Azure Disk at the path `/mnt/azure`.

    ```yaml
    kind: Pod
    apiVersion: v1
    metadata:
      name: mypod
    spec:
      containers:
        - name: mypod
          image: mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 250m
              memory: 256Mi
          volumeMounts:
            - mountPath: "/mnt/azure"
              name: volume
      volumes:
        - name: volume
          persistentVolumeClaim:
            claimName: azure-managed-disk
    ```

2. Create the pod using the following `kubectl` command:

    ```bash
    kubectl apply -f azure-pvc-disk.yaml
    ```

Now, you have a running pod with your Azure Disk mounted in the `/mnt/azure` directory. To check the pod configuration, use the following command:

```bash
kubectl describe pod mypod
```

For more information on storage options in AKS and related topics, you can refer to the following references:

- [Storage options for applications in Azure Kubernetes Service (AKS)](https://learn.microsoft.com/en-us/azure/aks/concepts-storage)
- [How to install the Azure CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli)
- [Kubernetes Storage Classes](https://kubernetes.io/docs/concepts/storage/storage-classes)
- [Kubectl Documentation](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply)
```

You can copy and paste this content into a Markdown file in your GitHub repository.