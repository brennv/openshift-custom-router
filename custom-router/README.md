# OpenShift Enterprise 3.1.1.6 custom router image example

For customizing the haproxy router we'll be referring to the docs here:
https://docs.openshift.com/enterprise/3.1/install_config/install/deploy_router.html#deploying-a-customized-haproxy-router

Confirm the current version of ose-haproxy-router:

# rpm -qi ose-haproxy-router

Noting your current ose-haproxy-router version, you can obtain a new haproxy-config.template file from the latest router image by running:

# docker run --rm --interactive=true --tty --entrypoint=cat \
    registry.access.redhat.com/openshift3/ose-haproxy-router:v3.0.2.0 haproxy-config.template

Save this content to a file for use as the basis of your customized template.

For an example template modification please refer to:
https://github.com/brennv/openshift-custom-router/tree/master/custom-router

This custom haproxy template configuration is an example of how to customize the default v3.1.1.6 haproxy router configuration, doubling the default check intervals (haproxy var "inter") from 5000ms to 10000ms.

For more on check intervals see the [haproxy docs on check intervals](https://www.haproxy.com/doc/aloha/7.0/haproxy/healthchecks.html#check-interval)

For this example, our starting-point OpenShift v3.1.1.6 haproxy template is shown here: `conf/haproxy-config.template-v3.1.1.6`

The modified haproxy template with changes to "inter" on lines 217, 227 and 239 is shown here: `conf/haproxy-config.template`

Create a repo following this example with changes to the newly obtained haproxy-config.template

Make sure to edit git uri in the `custom-router/custom-router.yaml` so that the source refers to it's new location.

```
source:
  contextDir: custom-router
  git:
    uri: https://github.com/brennv/openshift-custom-router      <--- Change
  type: Git
```

Build a new image referencing a path to the revised `custom-router.yaml`.

```
oc project default
oc new-app -f openshift-custom-router/custom-router/custom-router.yaml
```

Next we'll edit the router deployment configuration so that the router refers to our new image.

```
oc edit dc router
```

In `oc edit dc router`, replace:

```
image: registry.access.redhat.com/openshift3/ose-haproxy-router:v3.1.1.6
```

with:

```
image: <your docker registry IP like 172.30.x.x>:5000/default/custom-router:latest
```

To reflect our new changes:

```
oc start-build custom-router
oc deploy router --latest
```

Note: the router deployment config doesn't have ImageChange trigger by default, so we need to deploy it manually.
