---
layout: post
title: Cluster clusterfuck
date: 2023-02-24T23:46:03+01:00
tags: [kubernetes, linux, k8s, clsuter, calico, ingress]
---

If you have gained some experience in Kubernetes, then you definitely know that debugging distributed systems is .... a beast.

So, here are some short picks that I came across administrating my own cluster where all nodes are virtual machines running via KVM/QEMU.

## Time management

Make sure all your nodes have a correct time. Chrony is a good choice for that. That is even more true, when your nodes are virtual machines, and you have snapshots that your can revert back to. After such a revert, time will be a mess. So, make sure your time is accurate, to the millisecond!

## Overlay networks

In my cluster, I use project Calico. It works fine, but it seems it has some problems after reverting the cluster back to a previous state. Even after a time synchronization, I still - especially with Rook Ceph - encountered some weired scenarios where some Pods could not communicate correctly. So, I always run `kubectl -n calico-system delete pods -l k8s-app=calico-node`. This will delete all Calico Node-Pods. After a minute they should have been recreated. Solved a lot of mysterious issues for me!

## Ingress

So, time was right, network refreshed and still, I faced some issues with e.g. the Jaeger operator. It complained there was no route to ingress-nginx-controller-admission service. I then killed the ingress controller Pod. After it got recreated, the Jaeger operator found the route to ingress-nginx-controller-admission service but now complained about an invalid X509 certificate (signed by unknown authority).

This can be [fixed](https://fabianlee.org/2022/01/29/nginx-ingress-nginx-controller-admission-error-x509-certificate-signed-by-unknown-authority/) by taking the CA data from the nginx admission secret, and patching it into the nginx validating web hook. [^1][^2]

    # adjust to your namespace
    ns=default
    
    CA=$(kubectl -n $ns get secret ingress-nginx-admission -ojsonpath='{.data.ca}')
    
    kubectl patch validatingwebhookconfigurations ingress-nginx-admission -n $ns --type='json' -p='[{"op": "add", "path": "/webhooks/0/clientConfig/caBundle", "value":"'$CA'"}]'

[^1]: https://fabianlee.org/2022/01/29/nginx-ingress-nginx-controller-admission-error-x509-certificate-signed-by-unknown-authority/
[^2]: https://github.com/kubernetes/ingress-nginx/issues/5968#issuecomment-849772666


## Conclusion

After any reset or revert to a previous state, I do all of the above. That is, I force a time synchronization on every node, refresh the Calico network by deleting all Calico Pods with labels equal to `app=calico-node`, I delete the Ingress controller Pod and patch it afterwards.
