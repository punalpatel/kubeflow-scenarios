```
cd ~/kubeflow-model/examples/github_issue_summarization/notebooks
docker run -v $(pwd)/my_model:/my_model seldonio/core-python-wrapper:0.7 /my_model IssueSummarization 0.1 gcr.io --base-image=python:3.6 --image-name=gcr-repository-name/issue-summarization

cd build
./build_image.sh
```
## Deploy Trained Model

`cd ~/my-kubeflow/`{{execute}}

# Gives cluster-admin role to the default service account in the ${NAMESPACE}
`kubectl create clusterrolebinding seldon-admin --clusterrole=cluster-admin --serviceaccount=${NAMESPACE}:default`{{execute}}

# Install the kubeflow/seldon package
`ks pkg install kubeflow/seldon`{{execute}}


# Generate the seldon component and deploy it
```
ks generate seldon seldon --name=seldon --namespace=${NAMESPACE}
ks apply default -c seldon
```{{execute}}

```
ks generate seldon-serve-simple issue-summarization-model-serving \
  --name=issue-summarization \
  --image=katacoda/issue-summarization:0.1-zuowang \
  --namespace=${NAMESPACE} \
  --replicas=2
ks apply default -c issue-summarization-model-serving
```{{execute}}

`kubectl get pods -n ${NAMESPACE}`{{execute}}

# Query

```
curl -X POST -H 'Content-Type: application/json' -d '{"data":{"ndarray":[["issue overview add a new property to disable detection of image stream files those ended with -is.yml from target directory. expected behaviour by default cube should not process image stream files if user does not set it. current behaviour cube always try to execute -is.yml files which can cause some problems in most of cases, for example if you are using kuberentes instead of openshift or if you use together fabric8 maven plugin with cube"]]}}' http://[[HOST_IP]]:30080/seldon/issue-summarization/api/v0.1/predictions
```{{execute}}