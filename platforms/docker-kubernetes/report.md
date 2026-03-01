# Docker & Kubernetes Assessment Report

> [!TIP]
> Use this document to explain your design choices, optimisations and any challenges you faced.

## Dockerfile

Initially, I used the docker guide.
However, this wasn't incredibly useful, so I used then used the pnpm docker guide - since this project uses pnpm.

As dictated by the guide, I used a multi-stage dockerfile with seperate deps/build stages.

This was fairly straightforward, until we get to the crux of this specific problem, which is communicating over localhost.
Communicating over 127.0.0.1 is fine on a non-contanerised application, however, when we containerise an application, that
process is searching in it's container, and thus, is not able to find our host.

I fond this stack overflow question which seemed to help; where I used --network="host" in the docker run command.
https://stackoverflow.com/questions/24319662/from-inside-of-a-docker-container-how-do-i-connect-to-the-localhost-of-the-mach

After doing this, I could read data from 2026.

I used bruno for this simple GET request, but a curl request would have worked also (I am just lazy).
After doing this, I could read data from 2026, and got a response.

Funny to note, the 2025 data was unscrapable, and so the example in the github didn't work. One needed to use 2026.
However, the 404 request helped me out here.

Example of logs
```Shell
~/Dev/academic-calendar-api master* 3m 49s ❯ docker run --network="host" -p 3000:3000 academic-calendar-api:latest          06:49:47 PM
WARNING: Published ports are discarded when using host network mode
! Corepack is about to download https://registry.npmjs.org/pnpm/-/pnpm-10.20.0.tgz

> calendar@1.0.0 start /app
> node dist/server.js

{"level":30,"time":1772351442334,"pid":20,"hostname":"entrana","msg":"Server listening at http://[::1]:3000"}
{"level":30,"time":1772351442335,"pid":20,"hostname":"entrana","msg":"Server listening at http://127.0.0.1:3000"}
{"level":30,"time":1772351442335,"pid":20,"hostname":"entrana","msg":"Server listening at http://[::1]:3000"}
{"level":30,"time":1772351443757,"pid":20,"hostname":"entrana","reqId":"req-1","req":{"method":"GET","url":"/v1/dates/2025","host":"0.0.0.0:3000","remoteAddress":"127.0.0.1","remotePort":39904},"msg":"incoming request"}
{"level":30,"time":1772351443761,"pid":20,"hostname":"entrana","reqId":"req-1","res":{"statusCode":404},"responseTime":3.37149600032717,"msg":"request completed"}
{"level":30,"time":1772351466245,"pid":20,"hostname":"entrana","reqId":"req-2","req":{"method":"GET","url":"/v1/dates/2026","host":"0.0.0.0:3000","remoteAddress":"127.0.0.1","remotePort":56928},"msg":"incoming request"}
{"level":30,"time":1772351466246,"pid":20,"hostname":"entrana","reqId":"req-2","res":{"statusCode":200},"responseTime":0.9740799991413951,"msg":"request completed"}
{"level":30,"time":1772351955077,"pid":20,"hostname":"entrana","reqId":"req-3","req":{"method":"GET","url":"/v1/dates/2025","host":"0.0.0.0:3000","remoteAddress":"127.0.0.1","remotePort":54928},"msg":"incoming request"}
{"level":30,"time":1772351955078,"pid":20,"hostname":"entrana","reqId":"req-3","res":{"statusCode":404},"responseTime":0.9192869998514652,"msg":"request completed"}
{"level":30,"time":1772351956914,"pid":20,"hostname":"entrana","reqId":"req-4","req":{"method":"GET","url":"/v1/dates/2026","host":"0.0.0.0:3000","remoteAddress":"127.0.0.1","remotePort":54928},"msg":"incoming request"}
{"level":30,"time":1772351956915,"pid":20,"hostname":"entrana","reqId":"req-4","res":{"statusCode":200},"responseTime":0.78074000030756,"msg":"request completed"}
```

Also, I added a package for type annotations because all the red text was highly distressing and I got scared.

### Forked repository

`https://github.com/rfquin/academic-calendar-api`

## Kubernetes

I did this question with various resources, but mostly 
- https://www.navidrome.org/docs/installation/docker/  
- https://kubernetes.io/docs/tasks/configure-pod-container/translate-compose-kubernetes/
- https://minikube.sigs.k8s.io/docs/handbook/deploying/
- https://kubernetes.io/docs/reference/kubectl/
- https://spacelift.io/blog/kubernetes-cheat-sheet 

First, I was reading through the minikube docs and tried this command:
```bash
kubectl create deployment navidrome --image=deluan/navidrome:latest
minikube service navidrome
```

And this did in fact work!
However, this is not what the guide wanted us to do, so I went back, used the dockercompose file in the navidrome docs,
translated that to kube yaml files, then proceeded to spend an hour trying to figure out how to apply these files to kube.

I knew I could create something with kubectl, but how the hell do you apply these files instead of using a generated config file?
I then found the cheat sheet, and used ``kubectl apply -f .`` (or the files individually) and we got this working.

I still need to setup the volumes, but the task was just to get navidrome setup on localhost.
