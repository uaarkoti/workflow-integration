# Workflow Integration with external apps or CLI
[Jenkins Workflow](https://wiki.jenkins-ci.org/display/JENKINS/Workflow+Plugin) allows developers to do end-to-end automation of their applications all the way from build (triggered by a code commit by a developer) to deployment into production.

End-to-end automation could involve both manual and automated tests/tasks. Jenkins Workflow supports both. Manual tasks are supported through [input step](https://github.com/jenkinsci/workflow-plugin/blob/master/TUTORIAL.md#pausing-flyweight-vs-heavyweight-executors) while there are over 1000+ plugins to support other types of automation and integration with other tools.

When a workflow reaches the `input` step, it pauses for input. To continue, the responsible owner needs to login and resume the workflow while providing necessary data, as defined in the `input` step. This can be done through Jenkins UI or from an external application as Jenkins Workflow exposes RESTful API's for the same.

In this article, you will see how to leverage the RESTful API's to integrate Jenkins Workflow in your environment.

## Setup

### RESTful URL Format

```
http://<username>:<api-token>@<hostname>:<port>/<prefix>/job/<job-name>/<build-number>/input/<input-step-id>/<action>

username      - When security is enabled on Jenkins, the username you want to use to authenticate
api-token     - Can be obtained from http://<hostname>:<port>/<prefix>/user/<username>/configure
hostname      - Host where Jenkins is running
port          - Port where Jenkins is running (default : 8080)
job-name      - Name of your workflow job
build-number  - The build number of the workflow job
input-step-id - The id attribute you provided to your input step. If the 'id' attribute is not provided, Jenkins will create a random id.
                It is recommended to enforce the 'id' that is easy to reference
action        - The value of action changes depending on whether its an empty or non-empty input. The different values of action are
                1. submit (proceed with user input)
                2. proceedEmpty (proceed without user input)
                3. abort (abort workflow)
```

username/password is required if you secured your Jenkins instance.

## Triggering
When you add an `input` step to the workflow, you can require the user to provide additional information to continue (or not). There are a variety of parameters you can request from the user like `string`, `choice`, `password`, `file` etc....

The following input requires `comments` from the user which is of type `string`

```
input id: 'ApprovalForm', message: 'Please approve', parameters: [[$class: 'StringParameterDefinition', defaultValue: '', description: '', name: 'comments']]

```

The following does not require the user to provide any data to continue. Simply press "Proceed" or "Abort".

```
input id: 'ApprovalForm', message: 'Please approve'
```

### No input required
In this scenario we are going assume that no additional data is required to resume the workflow.

To Proceed

```
curl -X POST http://192.168.59.103:8080/job/workflow-integration-noinput/1/input/ApprovalAppnameDeployment/proceedEmpty
```

To Abort

```
curl -X POST http://192.168.59.103:8080/job/workflow-integration-noinput/1/input/ApprovalAppnameDeployment/abort
```

### Input required
In this scenario, the workflow requires input from the user to continue. Once the user provides the data, it is passed to the InputStep as a combination of key/value pairs and JSON object as shown below.

```
json:{"parameter": {"name": "comments", "value": "Approved"}}
proceed:Approve
```

To Proceed

```
curl -d proceed=Approve --data-urlencode json='{"parameter":{"name":"comments","value":"Approved"}}' http://192.168.59.103:8080/job/workflow-integration/6/input/ApproveDeployment/submit
```

To Abort

```
curl -X POST http://192.168.59.103:8080/job/workflow-integration/1/input/ApprovalAppnameDeployment/abort
```

## Demo
For convenience, this entire demo is captured in a docker container so you can see the API's in action.

To run the demo, you'll need [docker installed](https://docs.docker.com/installation/).

```
docker run -p 8080:8080 -p 8180:8180 -it uday/workflow-integration
```

To access the demo navigate to `http://<docker-host-ip>:8080` and build any of the jobs. Then from your command line use the above `curl` commands to see how you can trigger the jobs to continue.


## Conclusion
As described above, Jenkins Workflow makes it very easy to integrate with external application by exposing workflow job itself as well as manual steps as RESTful endpoints.

While we used `curl` command throughout this example, one should be able to use the same RESTful API's used by the `curl` command to achieve the same level of integration from any application.
