How many ClusterRoles do you see defined in the cluster?

```bash
k get clusterrole --no-headers | wc -l
```

How many ClusterRoleBindings exist on the cluster?

```bash
 k get clusterrolebinding --no-headers | wc -l
```

What namespace is the cluster-admin clusterrole part of?

Cluster Roles are cluster wide and not part of any namespace

What user/groups are the cluster-admin role bound to?

The ClusterRoleBinding for the role is with the same name.

```bash
k describe clusterrolebinding cluster-admin
```

output:

```bash
Name:         cluster-admin
Labels:       kubernetes.io/bootstrapping=rbac-defaults
Annotations:  rbac.authorization.kubernetes.io/autoupdate: true
Role:
  Kind:  ClusterRole
  Name:  cluster-admin
Subjects:
  Kind   Name            Namespace
  ----   ----            ---------
  Group  system:masters
```

What level of permission does the cluster-admin role grant?

Inspect the cluster-admin role's privileges.

```bash
k describe clusterrole cluster-admin
```

A new user michelle joined the team. She will be focusing on the nodes in the cluster. Create the required ClusterRoles and ClusterRoleBindings so she gets access to the nodes.

```bash
k create clusterrole michelle-role --verb=create,list,get,watch --resource=nodes

k create clusterrolebinding michelle-role-binding  --clusterrole=mi
chelle-role --user=michelle
```

michelle's responsibilities are growing and now she will be responsible for storage as well. Create the required ClusterRoles and ClusterRoleBindings to allow her access to Storage.

Get the API groups and resource names from command kubectl api-resources. Use the given spec:

ClusterRole: storage-admin

Resource: persistentvolumes

Resource: storageclasses

ClusterRoleBinding: michelle-storage-admin

ClusterRoleBinding Subject: michelle

ClusterRoleBinding Role: storage-admin

```bash
k create clusterrole storage-admin --verb=list,get,create,watch --resource=storageclasses,persistentvolumes

k create clusterrolebinding michelle-storage-admin --clusterrole=storage-admin --user=michelle
```
