# Selecting a Node-Pool

**Warning: Do not run `~/go/src/k8s-deployments/do.sh` directly**

Make sure to navigate to a specific directory. For example: `$ cd ~/go/src/k8s-deployments/microservices/configctl`

You can check available node-pools by executing

```bash
$ gcloud container node-pools list --format="table[box](name:label='Node Pool')"
```

To change the node-pool this deployment be scheduled to, execute the following

```bash
$ ./do.sh change_nodepool node-pool-db1
```

To confirm if the node pool has been changed, open any `.yaml` using this command

```bash
$ vi +/"cloud.google.com\/gke-nodepool: .*" filename.yaml
```

It should look something like this

```yaml
...
nodeSelector:
  cloud.google.com/gke-nodepool: pool-db1
...
```