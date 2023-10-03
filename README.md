# devops_workshop_eks_jenkins_loki_operator
# https://jenkinsci.github.io/kubernetes-operator/docs/getting-started/latest/installing-the-operator/

$ eksctl create cluster --name mycluster1 --region=us-east-1
$ eksctl get  cluster --name mycluster1 --region=us-east-1
$ kubectl get nodes

$ kubectl create namespace lwns
$ ./helm.exe  repo add jenkins https://raw.githubusercontent.com/jenkinsci/kubernetes-operator/master/chart
$ ./helm.exe install my-jenkins-operator jenkins/jenkins-operator -n lwns --set jenkins.enabled=false

$ kubectl --namespace lwns get pods -w


$ vim jenkins_instance.yaml
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
        image: jenkins/jenkins:2.319.1-lts-alpine
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


$ kubectl create -f jenkins_instance.yaml
$ kubectl --namespace lwns get pods -w

$ kubectl --namespace lwns get secret jenkins-operator-credentials-example -o 'jsonpath={.data.user}' | base64 -d
$ kubectl --namespace lwns get secret jenkins-operator-credentials-example -o 'jsonpath={.data.password}' | base64 -d

$ kubectl --namespace lwns port-forward jenkins-example 8080:8080






      
 
