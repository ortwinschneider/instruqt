---
slug: 01-prepare
id: z2b4ri8gr8rr
type: challenge
title: Prepare for Exercises
notes:
- type: text
  contents: |
    [serverless-main]: https://www.openshift.com/learn/topics/serverless
    [amq-docs]: https://developers.redhat.com/products/amq/overview
    [pipelines-main]: https://www.openshift.com/learn/topics/pipelines
    [service-mesh-main]: https://www.openshift.com/learn/topics/service-mesh

    In this self-paced tutorial, you will learn the basics of how to use OpenShift Serverless, which provides a development model to remove the overhead of server provisioning and maintenance from the developer.

    In this tutorial, you will:
    * Deploy an OpenShift Serverless `service`.
    * Deploy multiple `revisions` of a service.
    * Understand the `underlying compents` of a serverless service.
    * Understand how Serverless is able to `scale-to-zero`.
    * Run different revisions of a service via `canary` and `blue-green` deployments.
    * Utilize the `knative client`.

    ## Why Serverless?

    Deploying applications as Serverless services is becoming a popular architectural style. It seems like many organizations assume that _Functions as a Service (FaaS)_ implies a serverless architecture. We think it is more accurate to say that FaaS is one of the ways to utilize serverless, although it is not the only way. This raises a super critical question for enterprises that may have applications which could be monolith or a microservice: What is the easiest path to serverless application deployment?

    The answer is a platform that can run serverless workloads, while also enabling you to have complete control of the configuration, building, and deployment. Ideally, the platform also supports deploying the applications as linux containers.

    ## OpenShift Serverless

    In this chapter we introduce you to one such platform -- [OpenShift Serverless][serverless-main].  OpenShift Serverless helps developers to deploy and run applications that will scale up or scale to zero on-demand. Applications are packaged as OCI compliant Linux containers that can be run anywhere.  This is known as `Serving`.

    ![OpenShift Serving](https://raw.githubusercontent.com/openshift-instruqt/instruqt/master/assets/developing-on-openshift/serverless/00-intro/knative-serving-diagram.png)

    Serverless has a robust way to allow for applications to be triggered by a variety of event sources, such as events from your own applications, cloud services from multiple providers, Software as a Service (SaaS) systems and Red Hat Services ([AMQ Streams][amq-docs]).  This is known as `Eventing`.

    ![OpenShift Eventing](https://raw.githubusercontent.com/openshift-instruqt/instruqt/master/assets/developing-on-openshift/serverless/00-intro/knative-eventing-diagram.png)

    OpenShift Serverless applications can be integrated with other OpenShift services, such as OpenShift [Pipelines][pipelines-main], and [Service Mesh][service-mesh-main], delivering a complete serverless application development and deployment experience.

    This tutorial will focus on the `Serving` aspect of OpenShift Serverless as the first diagram showcases.  Be on the lookout for additional tutorials to dig further into Serverless, specifically `Eventing`.

    ## The Environment

    During this scenario, you will be using a hosted OpenShift environment that is created just for you. This environment is not shared with other users of the system. Because each user completing this scenario has their own environment, we had to make some concessions to ensure the overall platform is stable and used only for this training. For that reason, your environment will only be active for a one hour period. Keep this in mind before you get started on the content. Each time you start this training, a new environment will be created on the fly.

    The OpenShift environment created for you is running version 4.7 of the OpenShift Container Platform. This deployment is a self-contained environment that provides everything you need to be successful learning the platform. This includes a preconfigured command line environment, the OpenShift web console, public URLs, and sample applications.

    > **Note:** *It is possible to skip around in this tutorial.  The only pre-requisite for each section would be the initial `Prepare for Exercises` section.*
    >
    > *For example, you could run the `Prepare for Exercises` section immediately followed by the `Scaling` section.*

    Now, let's get started!
tabs:
- title: Terminal 1
  type: terminal
  hostname: crc
- title: Visual Editor
  type: code
  hostname: crc
  path: /root
difficulty: intermediate
timelimit: 360
---
[serverless-install-script]: https://github.com/openshift-labs/learn-katacoda/blob/master/developing-on-openshift/serverless/assets/01-prepare/install-serverless.bash
[olm-docs]: https://docs.openshift.com/container-platform/latest/operators/understanding/olm/olm-understanding-olm.html
[serving-docs]: https://github.com/knative/serving-operator#the-knativeserving-custom-resource

OpenShift Serverless is an OpenShift add-on that can be installed via an operator that is available within the OpenShift OperatorHub.

Some operators are able to be installed into single namespaces within a cluster and are only able to monitor resources within that namespace.  The OpenShift Serverless operator is one that installs globally on a cluster so that it is able to monitor and manage Serverless resources for every single project and user within the cluster.

You could install the Serverless operator using the *Operators* tab within the web console, or you can use the CLI tool `oc`.  In this instance, the terminal on the side is already running through an automated CLI install.  This [script can be found here][serverless-install-script].

Since the install will take some time, let's take a moment to review the installation via the web console.

> **Note:** *These steps are for informational purposes only. **Do not** follow them in this instance as there already is an automated install running in the terminal.*

## Log in and install the operator
This section is automated, so you won't need to install the operator.  If you wanted to reproduce these results on another cluster, you'd need to authenticate as an admin to complete the following steps:

![01-login](https://raw.githubusercontent.com/openshift-instruqt/instruqt/master/assets/developing-on-openshift/serverless/01-prepare/01-login.png)

Cluster administrators can install the `OpenShift Serverless` operator via *Operator Hub*

![02-operatorhub](https://raw.githubusercontent.com/openshift-instruqt/instruqt/master/assets/developing-on-openshift/serverless/01-prepare/02-operatorhub.png)

> **Note:** *We can inspect the details of the `serverless-operator` packagemanifest within the CLI via `oc describe packagemanifest serverless-operator -n openshift-marketplace`.*
>
> **Tip:** *You can find more information on how to add operators on the [OpenShift OLM Documentation Page][olm-docs].*

Next, our scripts will install the Serverless Operator into the `openshift-operators` project using the `stable` update channel.

![03-serverlessoperator](https://raw.githubusercontent.com/openshift-instruqt/instruqt/master/assets/developing-on-openshift/serverless/01-prepare/03-serverlessoperator.png)

Open the **Installed Operators** tab and watch the **OpenShift Serverless Operator**.  The operator is installed and ready when the `Status=Succeeded`.

![05-succeeded](https://raw.githubusercontent.com/openshift-instruqt/instruqt/master/assets/developing-on-openshift/serverless/01-prepare/05-succeeded.png)

> **Note:** *We can inspect the additional api resouces that the serverless operator added to the cluster via the CLI command `oc api-resources | egrep 'Knative|KIND'`*.

Next, we need to use these new resources provided by the serverless operator to install KnativeServing.

## Install KnativeServing
As per the [Knative Serving Operator documentation][serving-docs] you must create a `KnativeServing` object to install Knative Serving using the OpenShift Serverless Operator.

> **Note:** *Remember, these steps are for informational purposes only. **Do not** follow them in this instance as there already is an automated install running in the terminal.*

First we create the `knative-serving` project.

![06-kservingproject](https://raw.githubusercontent.com/openshift-instruqt/instruqt/master/assets/developing-on-openshift/serverless/01-prepare/06-kservingproject.png)

Within the `knative-serving` project open the **Installed Operators** tab and the **OpenShift Serverless Operator**.  Then create an instance of **Knative Serving**.

![07-kservinginstance](https://raw.githubusercontent.com/openshift-instruqt/instruqt/master/assets/developing-on-openshift/serverless/01-prepare/07-kservinginstance.png)

![08-kservinginstance](https://raw.githubusercontent.com/openshift-instruqt/instruqt/master/assets/developing-on-openshift/serverless/01-prepare/08-kservinginstance.png)

Open the Knative Serving instance.  It is deployed when the **Condition** `Ready=True`.

![09-kservingready](https://raw.githubusercontent.com/openshift-instruqt/instruqt/master/assets/developing-on-openshift/serverless/01-prepare/09-kservingready.png)

OpenShift Serverless should now be installed!

## Login as a Developer and Create a Project
Before beginning we should change to the non-privileged user `developer` and create a new `project` for the tutorial.

> **Note:** *Remember, these steps are for informational purposes only. **Do not** follow them in this instance as there already is an automated install running in the terminal.*

To change to the non-privileged user in our environment we login as username: `developer`, password: `developer`

Next create a new project by executing: `oc new-project serverless-tutorial`

There we go! You are all set to kickstart your serverless journey with **OpenShift Serverless**.

Please check for the terminal output of `Serverless Tutorial Ready!` before continuing.  Once ready, click `continue` to go to the next module on how to deploy your first severless service.
