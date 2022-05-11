---
title: "Typed Java shell for Kubernetes extensions at fingertips"
date: 2021-05-11T10:46:23+01:00
draft: false
---

Working with Java vibrant tooling ecosystem on Kubernetes nowadays is a blast and you should know about it!

![demo](/img/demo.gif)

Wait, wait, what's going on???

Let's go through the BOM of this demo first:

## :globe_with_meridians: Kubernetes extensions

On Kubernetes, the [Operator pattern](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/) has quickly become a
popular way to provide "Kubernetes native" experience for various services and applications.

The contract for "talking" to Operators is based on [Custom Resource Definitions](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/) (CRD) and [Custom Resources](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) (CR)
which are usually shipped as plain and agnostic `yaml` files.

I work on the [Keycloak](https://www.keycloak.org/) project and we will use its CRD for this post.

```
https://raw.githubusercontent.com/keycloak/keycloak-k8s-resources/18.0.0/kubernetes/keycloaks.k8s.keycloak.org-v1.yml
```

Try out your own!

## :fire: Fabric8 kubernetes-client

In the Java world, there is a top-notch [kubernetes-client](https://github.com/fabric8io/kubernetes-client) that enables seamless interaction with a Kubernetes cluster.

In particular, the `fabric8 kubernetes-client` provides a [fully typed API for resources](https://github.com/fabric8io/kubernetes-client/blob/master/doc/CHEATSHEET.md#resource-typed-api)
that gives Java developers a huge kick for safety and productivity.

__Gimme more!__

Starting from the upcoming version there is a new module called [java-generator](https://github.com/fabric8io/kubernetes-client/tree/master/java-generator) that takes CRDs as input spits output plain Java files that are directly usable from the previously mentioned `resources typed API`.

## :exclamation: Jbang

[jbang](https://www.jbang.dev/) is an awesome tool for running Java programs with ease.
Not just that, it already evolved into a pretty comprehensive tool for managing/packaging/using Java programs from the command line with minimal to no setup.

## :repeat: JShell

[jshell](https://en.wikipedia.org/wiki/JShell) is a Read-Evaluate-Print Loop (REPL) for Java that can be used for learning and prototyping Java code.
Here we are (ab)using it to be our interactive shell.

# :dango: All together

Glue all of this with a little bit of `bash` and you have a [little shell script](https://github.com/andreaTP/playing-with-jbang/blob/main/k8s-shell.sh) that can be executed as a one-liner.
The script takes one argument being the URL of the (`yaml` or `json`) CRD we want to interact with, e.g.:

```bash
bash <(curl -sL https://raw.githubusercontent.com/andreaTP/playing-with-jbang/main/k8s-shell.sh) \
  https://raw.githubusercontent.com/keycloak/keycloak-k8s-resources/18.0.0/kubernetes/keycloaks.k8s.keycloak.org-v1.yml
```

The script will, in a `temp` directory:

 - download the CRD
 - generate the Java code for the CRD using the latest version (`-SNAPSHOT`) of the `java-generator` invoked using `jbang`
 - generate a little `.java` file containing a basic dependencies/sources configuration for the following steps
 - produce a single `jar` containing everything needed using `jbang export`
 - start a `jshell` session using `jbang` interactive mode

:airplane: now enjoy the fun of experimenting with Java code on the fly!
Quickly sort out the `import`s:

```java
import io.fabric8.kubernetes.client.*;
import org.keycloak.k8s.v2alpha1.*;
```

Create a client for Kubernetes:

```java
var client = new DefaultKubernetesClient();
```

You can already do pretty nifty things on your cluster with (**typed!**) commands like:

```java
client.pods().list().getItems().stream().map(p -> p.getMetadata().getName()).collect(Collectors.joining(", "));
```

Create a client for the Keycloak CRD:

```java
var kcClient = client.resources(Keycloak.class);
```

Play with your `keycloak` CRs in the cluster using, again, a **fully typed** API!

```java
var kc = kcClient.list().getItems().get(0);
kc.getSpec().getInstances()
```

# :v: Conclusions

Java ecosystem and tooling are growing, maturing, and improving at an incredible pace making it a first-tier choice for developing on top of Kubernetes as of today.
In this post, we explored how to lively interact with a Kubernetes cluster and custom CRDs through a **fully typed**, generated on-the-fly, Java native API.

:runner: **Go and try it at home!** :runner:

---
Bear in mind that we are using some bleeding edge tech here but you can already play and take advantage of it as a developer. :innocent:
