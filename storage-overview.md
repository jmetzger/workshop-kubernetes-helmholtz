# Storage Overview 

## Overview 

```
bind-mount  # not recommended 
volumes
tmpfs 
```

## Holzauge sei wachsam, geht nicht in Kubernetes. 

```
stored only on one node
Does not work well in cluster
```

## Alternative for cluster 

```
glusterfs
cephfs 
nfs 

# Stichwort
ReadWriteMany 
```
