# Workflow Integration with external apps or CLI
Jenkins Workflow allows developers to do end-to-end automation of their applications all the way from code checkin to production deployment.

In complex environments, end-to-end automation involves manual and automated tasks. Jenkins Workflow supports both. Manual tasks as supported in Jenkins workflow through `input` step. When the workflow reached the `input` step, it pauses. To resume, the rightful owner needs to login and resume the workflow while providing necessary data, as defined in the "input" step. This can be done through Jenkins UI or from an external application as Jenkins Workflow exposes RESTful API's for the same.

In this article, you will see how to leverage the RESTful API's to integrate Jenkins Workflow in your environment.

## Demo container
For convenience, this entire demo is captured in a docker container so you can see the API's in action.

```
docker run -p 8080:8080 -p 8180:8180 -it uday/workflow-integration
```

## Setup

### REST URL Format

```
http://<username>:<password>@<hostname>:<port>/job/<job-name>/<build-number>/input/<input-step-id>/<action>

username      - When security is enabled on Jenkins, the username you want to use to authenticate
password      - When security is enabled on Jenkins, the password you want for the username
hostname      - Host where Jenkins is running
port          - Port where Jenkins is running (default : 8080)
job-name      - Name of your workflow job
build-number  - The build number of the workflow job
input-step-id - The id attribute you provided to your input step. If the 'id' attribute is not provided, Jenkins will create a random id.
                It is recommended to enforce the 'id' that is easy to reference
action        - The value of action changes depending on whether its an empty or non-empty input. The different values of action are
                1. submit (non-empty)
                2. proceedEmpty (empty & proceed)
                3. abort (empty & abort)
```

username/password is required if you secured your Jenkins instance.

## Triggering

### No input required

When an `input` step is added to the workflow, it can be added to require user to provide additional data or not.

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
name:comments
value:Approved
json:{"parameter": {"name": "comments", "value": "Approved"}}
proceed:Approve
```

To Proceed

```
curl -d "name=comments&value=Approved&proceed=Approve" --data-urlencode json='{"parameter":{"name":"comments","value":"Approved"}}' http://192.168.59.103:8080/job/workflow-integration/6/input/ApproveDeployment/submit
```

To Abort

```
curl -X POST http://192.168.59.103:8080/job/workflow-integration/1/input/ApprovalAppnameDeployment/submit
```
