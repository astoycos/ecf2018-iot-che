# EclipseCon France 2018 – IoT cloud development with Che

This repository contains the resources for my talk "IoT cloud development with Che"
at [EclipseCon France 2018](https://www.eclipsecon.org/france2018/).

**Note**: The setup is targeted to a node with a specific hostname. So it will contain
some specific IP addresses and hostnames which need to be replaced when replicating
the setup. 

Clone this repository with `--recursive` or `--recursive-submodules`
in order to fetch all submodules during the initial clone.

The actual setup is covered in: [INSTALL.md](INSTALL.md "Installation Guide")

## Talk & slides

You can find the actual talk here, including a link to the slides:

<https://www.eclipsecon.org/france2018/session/iot-cloud-development-che>

## See also

This project uses a bunch of other repositories:

  * [eclipse/hono](https://github.com/eclipse/hono): The Eclipse Hono™ project – IoT cloud messaging platform
  * [eclipse/che](https://github.com/eclipse/che): The Eclipse Che™ project – Next-generation Eclipse IDE
  * [openshift/openshift-ansible](https://github.com/openshift/openshift-ansible): OpenShift advanced installer
  * [EnMasseProject/enmasse](https://github.com/EnMasseProject/enmasse): EnMasse project – Scalable messaging for Kubernetes and OpenShift
  * [strimzi/strimzi](https://github.com/strimzi/strimzi): Apache Kafka running on Kubernetes and OpenShift
  * [ctron/hono-demo-1](https://github.com/ctron/hono-demo-1): Used as MQTT based example data producer for Hono.
  * [ctron/hono-example-bridge](https://github.com/ctron/hono-example-bridge): Consuming telemetry data from Hono, forwarding to Kafka.
  * [ctron/hono-example-demo-gauge](https://github.com/ctron/hono-example-demo-gauge): A simple typescript frontend application, showing a gauge of the most recent value.
