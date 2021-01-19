# 1.Lay down a plan for CI/CD. As there will be tests pertaining to each microservice, how do you plan to run them in tandem in the deployment pipeline. 
        We will set up a CI/CD pipeline using Jenkins for microservices running in Kubernetes on AWS EKS.
        In a typical microservices-based architecture, each microservice can be developed in a different programming language or framework. All the build instructions for a particular microservice stay in Dockerfile placed at the root of each repository.
        Jenkins pipeline having the following stages.
        1. Source
        2. Build and Test
        3. Artifact
        4. Deploy
        5. Notify

        Creating a multibranch Pipeline
        1. It wil take a clone of the git repository 
        2. There MVN/Gradle build command will and it iwll build the source code and exicute the test cases
        3. Once the above step is successthen i will start to build the docker image for our source code and push that image to a specific container registery(docker hub, ecr acr etc.) 
        4. Then we will run the sonar scaner for our source code to check  the code civerage and if the code coverge it grater then our crateria then i will move to next step else mark the job as failed    
        5. Now the next step is exiting in that jenkins will deploy the docker image on kuberntes cluster by using deployment files that is present in git repo.

# 2.Prepare a deployment strategy that can be adapted for Staging, UAT and Production. Note that staging setup needs to be less in terms of operating cost. 

        Blue/Green Deployment
            A Blue/Green deployment is a way of accomplishing a zero-downtime upgrade to an existing application. The “Blue” version is the currently running copy of the application and the “Green” version is the new version. Once the green version is ready, traffic is rerouted to the new version.
            The user will experience no downtime and will seamlessly switch between blue and green versions of the application.
            Blue/Green With Kubernetes
            In Kubernetes, this is slightly different because our blue and green versions are actually a set of containers and the load balancer is built into Kubernetes. It’s possible to do blue/green deployments in lots of ways with Kubernetes. we’ll use deployments and services to get the job done. The same idea will work for replica sets.
            Flow
            Start with already deployed containers (Deployment) and service.
            Deploy new deployment
            Issue a health check
            If health check passes, update load balancer and remove old deployment
            If the health check fails, stop and send alerts

# 3. Prepare a plan to see logs in all the above three environments. 

    There are several log data types that we can collect in Kubernetes.

    ###Application Logs
    First and foremost are the logs from the applications that run on Kubernetes. The data stored in these logs consists of the information that our applications output as they run. Typically, this data is written to stdout inside the container where the application runs.

    ###Kubernetes Cluster Logs
    Several of the components that form Kubernetes itself generate their own logs:

    1. Kube-apiserver
    2. Kube-scheduler
    3. Etcd
    4. Kube-proxy
    5. Kubelet
    These logs are usually stored in files under the /var/log directory of the server on which the service runs. For most services, that server is the Kubernetes master node. Kubelet, however, runs on worker nodes.

    ###Kubernetes Events
    Kubernetes keeps track of what it calls “events,” which can be normal changes to the state of an object in a cluster (such as a container being created or starting) or errors (such as the exhaustion of resources).

    Events provide only limited context and visibility. They tell we that something happened, but not much about why it happened.

    ###Kubernetes Audit Logs
    Kubernetes can be configured to log requests to the Kube-apiserver. These include requests made by humans (such as requesting a list of running pods) and Kubernetes resources (such as a container requesting access to storage).

    Audit logs record who or what issued the request, what the request was for, and the result. If we need to troubleshoot a problem related to an API request, audit logs provide a great deal of visibility.

    ###Viewing Application Logs
    There are two main ways to interact with application log data. The first is to run a command like

    **kubectl logs pod-name**

    The kubectl method is useful for a quick look at log data. Suppose we want to store logs persistently and analyze them systematically. In that case, better served by using an external logging tool like ELK, Prometheus etc.

    There are multiple ways of viewing cluster logs. We can simply log into the server that hosts the log we want to view (as noted above, that’s the Kubernetes master node server in most cases) and open the individual log files directly. 
    The most user-friendly solution is to use an external logging tool like ELK, Prometheus etc.

    ###Viewing Events
    We can view Kubernetes event data through kubectl with a command like

    **kubectl get events**

# 4.Prepare a plan to add alerts and monitoring across all of the mentioned components. 

    Monitoring and Alerting in Kubernetes

    The system is continuously monitored, and when a predefined threshold of some metric is crossed, an alert is sent out to pre-configured recipients. Thresholds can be set to not only indicate the occurrence of an issue with the system, but rather also proactively indicate that the system might be headed towards an issue. Alerting on such thresholds may give the stakeholders adequate indication and time to monitor the system more closely and even start some mitigation activities that will lower the probability of issue occurrence. The exact thresholds for alerting will no doubt have to be thought out, as having low thresholds that are reached frequently but have little impact, will lead to very noisy alerting; i.e. We will receive frequent alerts for events that do not require immediate attention. On the other hand if the thresholds are too high, we may not receive alerts until the issue has already occurred, or it is too late to mitigate in any way. The monitoring data will of course all be there in the required fine-grained detail to allow for effective root cause analysis, but the alerts should only be triggered within the sweet-spot of the threshold range


    We can use prometheus for monitoring on a Kubernetes cluster.

    A convenient way of deploying Prometheus on Kubernetes is by using the Prometheus Operator. With Prometheus there are a few components involved at the center of which we have the Prometheus Server. Prometheus server essentially scrapes metric data that other services expose. Each Kubernetes node exposes certain Services like Node Exporter & Kubelet, which contain system level metrics. Node Exporter collects OS level metrics of a node through Docker host and Kubelet contains cadvisor which collects container metrics from the Docker Engine. For Kubernetes monitoring, Prometheus scrapes metrics from each Kubelet and Node Exporter from all nodes.
    There may be some services such as ephemeral and batch jobs that Prometheus servers cannot reliably scrape because of their ephemeral nature. For such a case we have the Prometheus Pushgateway which is able to have such jobs or services push their metrics to it, and in turn Pushgateway exposes these metrics to Prometheus for scraping.
    For visualization we have Grafana which queries Prometheus and groups the results and displays it in Dashboards.


    For alerting, Prometheus Server triggers alerts on Prometheus AlertManager based on the rules defined within AlertManager. The alert trigger sends notifications through a desired notification channel.
    Prometheus Operator creates/configures/manages Prometheus atop Kubernetes and makes Kubernetes native Prometheus configuration in the form of Customer Resource Definitions (CRDs). Two of these CRDs are for PrometheusRule and AlertManager.
    We can define multiple alerting rules using PrometheusRule which are actually Prometheus queries. When the rule query is satisfied, it fires the alert to AlertManager. Each rule can have labels. AlertManager has Routes, which can be defined using these labels and each Route can have multiple Receivers, and each receiver can send the notification to a specific app like Slack or email. We can also set a time period during which a rule is satisfied, for the alert to be triggered, e.g. we want the alert to trigger if Kubelet is down for 2 minutes.


# 5. Able to come up with a plan which can be scaled to 100K users in a cost optimized way. 

    ##Scaling options for applications 

    To handle the load for 100k users we may need to increase or decrease the number of computing resources. As the number of application instances, we need changes, the number of underlying Kubernetes nodes may also need to change. we also might need to quickly provision a large number of additional application instances.

    ###Horizontal pod autoscaler

    Kubernetes uses the horizontal pod autoscaler (HPA) to monitor the resource demand and automatically scale the number of replicas. By default, the horizontal pod autoscaler checks the Metrics API every 30 seconds for any required changes in replica count. When changes are required, the number of replicas is increased or decreased accordingly. 

    When We configure the horizontal pod autoscaler for a given deployment, we define the minimum and a maximum number of replicas that can run. we also define the metric to monitor and base any scaling decisions on, such as CPU usage.

    ###Cluster autoscaler
    
    To respond to changing pod demands, Kubernetes has a cluster autoscaler, that adjusts the number of nodes based on the requested compute resources in the node pool. By default, the cluster autoscaler checks the Metrics API server every 10 seconds for any required changes in node count. If the cluster autoscale determines that a change is required, the number of nodes in our K8S cluster is increased or decreased accordingly. 
    Cluster autoscaler is typically used alongside the horizontal pod autoscaler. When combined, the horizontal pod autoscaler increases or decreases the number of pods based on application demand, and the cluster autoscaler adjusts the number of nodes as needed to run those additional pods accordingly.
