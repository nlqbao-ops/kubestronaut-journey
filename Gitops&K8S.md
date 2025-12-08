eventual consistency 

cicd 

gitops operator 

- git clone repo - fetch the latest version of config 
- discover manifests - examine clone repo for k8s manifest 
- kubectl apply - apply all the discovered manifest to the cluster 

example of cronjob to clone repo 
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: gitops-cron
  namespace: gitops
spec:
  schedule: "*/5 * * * *"                            #A
  concurrencyPolicy: Forbid                          #B
  jobTemplate:                                       #D
    spec:
      backoffLimit: 0                                #C
      template:
        spec:
          restartPolicy: Never                       #E
          serviceAccountName: gitops-serviceaccount  #F
          containers:
          - name: gitops-operator
            image: gitopsbook/example-operator:v1.0  #G
            command: [sh, -e, -c]                    #H
            args:
            - git clone https://github.com/example*** /tmp/example &&
              find /tmp/example -name '*.yaml' -exec kubectl apply -f {} \;
```

gitops CI pipeline 
- build your app and run unit testing 
- publish a new container image to container registry
- update k8s manifests in git to reflect the new image


sample gitops ci sh

```sh

export VERSION=$(git rev-parse HEAD | cut -c1-7)
make build
make test

export NEW_IMAGE="gitopsbook/sample-app:${VERSION}"
docker build -t ${NEW_IMAGE} .
docker push ${NEW_IMAGE}

git clone http://github.com/gitopsbook/sample-app-deployment.git
cd sample-app-deployment

kubectl patch \
  --local \
  -o yaml \
  -f deployment.yaml \
  -p "spec:
        template:
          spec:
            containers:
            - name: sample-app
              image: ${NEW_IMAGE}" \
  > /tmp/newdeployment.yaml
mv /tmp/newdeployment.yaml deployment.yaml

git commit deployment.yaml -m "Update sample-app image to ${NEW_IMAGE}"
git push

```

image tags 

trap of latest tag

- container tags should not be reused: k8s will not deploy new version to the cluster when tags are reused
- unique version enables tracebility 
- enable rollback 

