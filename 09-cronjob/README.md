# Running a background process: `cronjob`

## Introduction

In this section you will learn how to run background tasks using crons & jobs.

## `CronJob`

Let's start with crons. Crons are like the cron in linux a time-based job scheduler.

```yml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: simple-cronjob
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: Never
          containers:
          - name: pi
            image: perl
            command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(100)"]
```

* `spec`:
  * `schedule`: when to start this cron, in the same format as the linux crons. `*/2 * * * *` means every 2 minutes
  * `jobTemplate`: the template of the container(s) to start
    * `command`: the command to run, here compute the first 100 digits of π.

Let's apply it:

```sh
$ kubectl apply -f 09-cronjob/01-simple-cronjob.yml
cronjob.batch "simple-cronjob" created
```

Wait a bit and access the logs of the pod created by the cron.

## `Job`

If you need to run a one time job, you can use the `Job` in Kubernetes. In fact the `CronJob` will start a `Job` for you at the scheduled interval.

```yml
apiVersion: batch/v1
kind: Job
metadata:
  name: simple-job
spec:
  template:
    spec:
      containers:
      - name: c
        image: busybox
        command: ["sh", "-c", "echo Processing item $ITEM && sleep 5"]
      restartPolicy: Never
```

This manifest is fairly close to a `CronJob`.

Apply it and see what is happening. Does it restarts?

```sh
$ kubectl apply -f 09-cronjob/02-simple-job.yml
job.batch "simple-job" created
```

If you have a long running background process - like a consumer of a queue - you can use a `deployment` without a `service`.

## Exercises

1. Transform the `simple-job` to a cron job running every 2 minutes
2. Transform the `simple-cronjob` in a deployment of `1` replica, and compute the 1_000 first digits of π. What is happening when the container finishes?

## Clean up

```sh
kubectl delete deployment,rs,service,cronjob,pod --all
```

## Links

* https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/
