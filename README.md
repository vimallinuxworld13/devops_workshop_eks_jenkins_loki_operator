# DevOps Masterclass

**EKS Kubernetes** to deploy **Jenkins CI/CD** tool using Automated way **Operator**, & manage Jenkins **Pipeline as Code** by Jenkins Kubernetes **CRD** & Application jobs launches in **dynamic provisioning node** our kubernetes deployment pods and monitoring of infrastructure log by **Grafana Loki & Promtail** & Metrics by **Prometheus Monitoring Tool**

## Prerequisites

1. AWS cloud: [https://aws.amazon.com/](https://aws.amazon.com/)

2. New User / Account : IAM user -> power : policy : admin access

3. Password (key) access / secret key

4. Download : [https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

5. Login in CLI: `aws configure`

6. *eksctl tool* : install kubernetes [https://github.com/eksctl-io/eksctl/releases/tag/v0.161.0](https://github.com/eksctl-io/eksctl/releases/tag/v0.161.0)

7. `cd C:\Program Files\Kubernetes`

8. Install kubernetes over AWS cloud
    - `cd C:\Program Files\Kubernetes`
    - `eksctl create cluster --name vimal12345 --region ap-southeast-1`
9. Install kubectl to connect to kubernetes [https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/)

10. Connect check : `kubectl get nodes`

## DevOps Workshop

- `eksctl create cluster --name mycluster1 --region=us-east-1`
- `eksctl get  cluster --name mycluster1 --region=us-east-1`
- `kubectl get nodes`

```powershell
kubectl create -f \
https://raw.githubusercontent.com/operator-framework/operator-lifecycle-manager/master/deploy/upstream/quickstart/crds.yaml
```

```powershell
kubectl create -f \
https://raw.githubusercontent.com/operator-framework/operator-lifecycle-manager/master/deploy/upstream/quickstart/olm.yaml
```

- `kubectl get catalogsources -n olm`
- `kubectl get packagemanifests -l catalog=operatorhubio-catalog`

[https://github.com/helm/helm/releases](https://github.com/helm/helm/releases)

- `kubectl create namespace lwns`
- `./helm.exe  repo add jenkins https://raw.githubusercontent.com/jenkinsci/kubernetes-operator/master/chart`
- `./helm.exe install my-jenkins-operator jenkins/jenkins-operator -n lwns --set jenkins.enabled=false`

- `kubectl --namespace lwns get pods -w`
- `vim jenkins_instance.yaml`

```yml
apiVersion: jenkins.io/v1alpha2
kind: Jenkins
metadata:
  name: example
  namespace: lwns
spec:
  configurationAsCode:
    configurations: []
    secret:
      name: ""
  groovyScripts:
    configurations: []
    secret:
      name: ""
  jenkinsAPISettings:
    authorizationStrategy: createUser
  master:
    disableCSRFProtection: false
    containers:
      - name: jenkins-master
        image: jenkins/jenkins:lts
        imagePullPolicy: Always
        livenessProbe:
          failureThreshold: 12
          httpGet:
            path: /login
            port: http
            scheme: HTTP
          initialDelaySeconds: 100
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 5
        readinessProbe:
          failureThreshold: 10
          httpGet:
            path: /login
            port: http
            scheme: HTTP
          initialDelaySeconds: 80
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        resources:
          limits:
            cpu: 1500m
            memory: 3Gi
          requests:
            cpu: "1"
            memory: 500Mi
  seedJobs:
    - id: jenkins-operator
      targets: "cicd/jobs/*.jenkins"
      description: "LW Jenkins Operator repository"
      repositoryBranch: master
      repositoryUrl: https://github.com/jenkinsci/kubernetes-operator.git
```

- `kubectl create -f jenkins_instance.yaml`
- `kubectl --namespace lwns get pods -w`
- `kubectl --namespace lwns get secret jenkins-operator-credentials-example -o 'jsonpath={.data.user}' | base64 -d`
- `kubectl --namespace lwns get secret jenkins-operator-credentials-example -o 'jsonpath={.data.password}' | base64 -d`
- `kubectl --namespace lwns port-forward jenkins-example 8080:8080`
- `./helm.exe  repo add grafana https://grafana.github.io/helm-charts`
- `./helm.exe  repo update`
- `./helm.exe upgrade --install loki grafana/loki-stack  --set grafana.enabled=true,prometheus.enabled=true,prometheus.alertmanager.persistentVolume.enabled=false,prometheus.server.persistentVolume.enabled=false`
- `kubectl patch svc loki-grafana -p '{"spec": {"type": "LoadBalancer"}}'`
- `kubectl get svc loki-grafana -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'`
- `kubectl get secret loki-grafana -o go-template='{{range $k,$v := .data}}{{printf "%s: " $k}}{{if not $v}}{{$v}}{{else}}{{$v | base64decode}}{{end}}{{"\n"}}{{end}}'`

- Loki log explorer and give this LogQL query in Grafana:
  - `{app="loki-medium-logs",namespace="default"}`
  - `{namespace="lwns"}`

Clean up:

- `helm delete loki`
- `kubectl delete deploy loki-medium-logs`

[https://grafana.com/grafana/dashboards/15141-kubernetes-service-logs/](https://grafana.com/grafana/dashboards/15141-kubernetes-service-logs/)
[https://grafana.com/grafana/dashboards/315-kubernetes-cluster-monitoring-via-prometheus/](https://grafana.com/grafana/dashboards/315-kubernetes-cluster-monitoring-via-prometheus/)
