# Kubernetes Info

This is a very short and very incomplete overview of Kubernetes intended to be used internally for ITC Members of the [S.P.E.L.L Projekt](https://spell-plattform.de/) of the DRK Landesverband Rheinland Pfalz e.V.. This only contains the most basic of information which should however be enough to be able to manage non complex Kubernetes Instances.

## Table of Contents

- [Kubernetes Commands Cheatsheet](./cheatsheet_commands.md) \[[PDF Version](/pdfs/cheatsheet_commands.pdf)]
- [Components and Objects](./components.md.md)
  - [ConfigMap](./yamls.md#configmap)
  - [Secrets](./yamls.md#secrets)
  - [Pods](./yamls.md#pods)
  - [Nodes](./yamls.md#nodes)
  - [Deployment](./yamls.md#deployment)
  - [Service](./yamls.md#service)
  - [Jobs](./yamls.md#jobs)
    - [CronJobs](./yamls.md#cronjobs)
  - [Control Plane](./yamls.md#control-plane)

### Additional Info

- Doing things with a CLI is nice, but can also be overbearing and it doesn't provide a nice overview. So Kubernetes has the possibility to provide a [Web Interace](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/). It's quite nice and can basically do everything explained in [Components and Objects](./components.md) without having to memorize commandline commands.

- edX has a very nice [Kubernetes Course](https://www.edx.org/course/introduction-to-kubernetes) held by professionals, which is completely free, if you don't want to be granted a certificate at the end.
