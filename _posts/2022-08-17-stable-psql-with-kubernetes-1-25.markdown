---
layout: post
title: Stable PSQL with Kubernetes 1.25
date: 2022-08-17T23:49:03+02:00
tags: [kuberntes, postgresql, psql]
---

For some days now I had to deal with postgresql in kubernetes. Dealing with it is meant verbatim in this case because it almost drove me crazy.

## The goal?

Run a posgres DB, connect to it and execution some actions/

## The problem?

Simply put I spun up a postgresql cluster (via helm) and port-forwarded to one of the pods to run some actions against it. That's it. I should then have been able to use my IDE of choice and manipulate the tables to my liking. I did a "test connection" and got a green light. Fine! Save and go! Well, no. Connection reset!

## Cause and effect?

After some deep-diving (into the night), it turned out that I could connect via psql and also connect to a given DB directly (providing the -dbname parameter to psql) to run some SQL statements. But whenever I changed to another DB by with `\c`, the connection got reset. From within my IDE I did not even get the chance to select a DB!

## Solution 1

The above happens with SSL connections (the default) only and a work-around is to disable SSL, for example like this:
    
    PGSSLMODE=disable psql -h <host> -p <port> -U <user>

Soon, after I opened (yet another) issue on github, it got picked up by the community and we already have a [PR](https://github.com/kubernetes/kubernetes/pull/111860) that hopefully will make its way into the upcoming 1.25 release.

## Solution 2

If you do not like disabling SSL or simply don't have the luxury to disable security means like SSL, you can spin up an [ambassador](https://www.weave.works/blog/kubernetes-patterns-the-ambassador-pattern) container (or pod) that will server as an intermediary.

    apiVersion: v1
    kind: Pod
    metadata:
    name: postgres-ambassador
    labels:
        name: postgres-ambassador
    spec:
    containers:
    - name: postgres-ambassador
        image: alpine/socat
        command: ["socat", "-dd", "tcp4-listen:5432,fork,reuseaddr", "tcp4:<postgres-service>:5432"]
        resources:
        ports:
        - containerPort: 5432

Replace `<postgres-host>` with the service or pod you actually want to connect to (by name or IP).[^1]

In the example above one could then connect to the ambassador pod in place of the actual postgres pod. This way a connection reset will be caught by socat and the connection will stay alive, even with SSL enabled.

## Final words

I love community work. It's so much fun to see something like kubernetes evolving and being able to participate, too. I did not do a PR but watching an initial problem picked up by the community to find a solution simple makes me fell "WOA!". But doing a PR myself is even more fun! ;-)

So, like I mentiond before. Hopefully this PR will make its way into the 1.25 release. Until then, use the work-around above.


[^1]: https://stackoverflow.com/questions/70762591/kubectl-port-forwarding-to-remote-postgres-port
