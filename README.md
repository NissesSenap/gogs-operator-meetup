# Steeling with pride

All cred for writing this base operator goes to Wolfgang Kulhanek and he's team at RedHat.
https://github.com/wkulhanek

## NOTES

I use OpenShift 4 to show this demo, due to this I use oc instead of kubectl.
Think of it as kubectl but with extra features.

## Create a ansible operator

operator-sdk new gogs-operator --api-version=gpte.opentlc.com/v1alpha1 --kind=Gogs --type=ansible --generate-playbook

## How to for meetup

Get your own QUAY_ID, it's free.

I assume that you use podman but this will work just as fine with docker.

```shell
export QUAY_ID=nissessenap
export QUAY_PASSWORD='supersecretpassword123'
podman login -u ${QUAY_ID} -p ${QUAY_PASSWORD} quay.io

# Build the operator
operator-sdk build quay.io/${QUAY_ID}/gogs-operator:v0.0.4
podman push quay.io/${QUAY_ID}/gogs-operator:v0.0.4
```

Update the ./deploy/operator.yaml file to match your container.
In my case i update it with: quay.io/nissessenap/gogs-operator:<tag version>

### Deploy pre-requierments

Time to create the CRD, have a look at:
./deploy/crds/gpte.opentlc.com_gogs_crd.yaml

Take a look at the rbac rules
cat ./gogs-admin-rbac.yaml

The label rbac.authorization.k8s.io/aggregate-to-admin="true" modifies the default project admin role to automatically grant this cluster role to any project owner, who then automatically gets admin permission on the project.

```shell
oc create -f ./deploy/crds/gpte.opentlc.com_gogs_crd.yaml

oc create -f ./gogs-admin-rbac.yaml

# oc new-project is just like creating a normal namespace.
# Create namespace
oc new-project gogs-operator --display-name="Gogs"

# Create service account
oc create -f ./deploy/service_account.yaml

```

#### Role definition

Since I'm using openshift you will see routes in the role definition.
If you use kubernetes change this to your ingress.

In openshift '*' is not supported due to the risk of giving to much access, instead
you have to specify all the verbs that you want to use that's why the file looks like it does.

Create the role and the rolebinding.

```shell
oc create -f ./deploy/role.yaml
oc create -f ./deploy/role_binding.yaml
```

### Deploy the operator

oc create -f ./deploy/operator.yaml

### Deploy gogs using the operator

oc create -f deploy/crds/gpte.opentlc.com_v1alpha1_gogs_cr.yaml

### Watch output

oc logs -f gogs-operator-6fd8548bfb-dvxzf -c operator

