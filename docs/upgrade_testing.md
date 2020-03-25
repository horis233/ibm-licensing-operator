# Testing Automatic Operator upgrades using OLM with Subscriptions

In order to test upgrades you can use your own opencloud account:
For example I will use this:

[https://quay.io/application/adamdyszy/ibm-licensing-operator-app](https://quay.io/application/adamdyszy/ibm-licensing-operator-app)

Then we do these steps:

- delete ibm-licensing-operator-app olm on opencloudio account from UI:

- push current version of olm-catalog:

I made this script to push faster:

```bash
# enter your olm-catalog directory
cd /home/adam/go/src/github.com/ibm/ibm-licensing-operator/deploy/olm-catalog

# your operator dir inside olm-catalog
export OPERATOR_DIR=ibm-licensing-operator/
# Quay namespace where olm will be pushed
# I made it as parameter so I can push to opencloudio when ready
export QUAY_NAMESPACE=$2
# you operator olm name
export PACKAGE_NAME=ibm-licensing-operator-app
# version will be used as first parameter
export PACKAGE_VERSION=$1
# you need your quay account token
export QUAY_TOKEN="basic ${your_quay_account_token}"

operator-courier push "$OPERATOR_DIR" "$QUAY_NAMESPACE" "$PACKAGE_NAME" "$PACKAGE_VERSION" "$QUAY_TOKEN"
```

I saved above script as olm-push-licensing

Now to push my version 1.0.0 I do this:

```bash
olm-push-licensing 1.0.0 adamdyszy
```

- remember to making it public on your opencloud namespace settings, for me the page was:
[https://quay.io/application/adamdyszy/ibm-licensing-operator-app?tab=settings](https://quay.io/application/adamdyszy/ibm-licensing-operator-app?tab=settings)

- now install your operator on OpenShift cluster, add some Custom Resource and make sure it works

- install your operator on non OpenShift cluster cluster too, add some Custom Resource and make sure it works

- make some changes to the code that You can detect in operator image logs

- add new operator image to your registry, for me the commands were:

```bash
# create image:
make images
# it created some image f.e. quay.io/opencloudio/ibm-licensing-operator:f6aae00d-dirty
# then push it
docker push quay.io/opencloudio/ibm-licensing-operator:f6aae00d-dirty
```

Now we want to add new olm with version 1.0.1 that replace 1.0.0, that use new created image:

- first change deploy/operator.yaml to include operator image
- then generate csv:

```bash
operator-sdk generate csv --csv-version 1.0.1 --from-version 1.0.0 --operator-name ibm-licensing-operator --update-crds
```

- if you compare new 1.0.1 csv yaml with 1.0.0 csv yaml you will see there are some things that were not copied:
- look for customresourcedefinitions field and copy needed values
- look for watch namespace it is probably overridden too if you have cluster scoped operator
- look for operator image you will probably need to add new one here to, it should have 2 occurrences
- if you want to modify upgrade scenario olm logic (what versions are upgraded to this version) do it now at replace section
- next replace currectCSV version in package.yaml:
[deploy/olm-catalog/ibm-licensing-operator/ibm-licensing-operator.package.yaml](../deploy/olm-catalog/ibm-licensing-operator/ibm-licensing-operator.package.yaml)
- if you are ready to push then push new version:
olm-push-licensing 1.0.1 adamdyszy

- example timestamp when it was pushed:
14:02Z march 24

- wait for subscription to get newer version
- done upgrade successful on OCP
- install plan creation date for upgrade on OCP:
creationTimestamp '2020-03-24T14:12:17Z'
- done upgrade successful on non OCP
- install plan creation date for upgrade on NON OCP:
creationTimestamp: "2020-03-24T14:13:17Z"

- example code changes can be seen in upgrade_scenario branch last few commits here:
[https://github.com/IBM/ibm-licensing-operator/commits/upgrade_scenario](https://github.com/IBM/ibm-licensing-operator/commits/upgrade_scenario)