# workflow-integration
Example on how to integrate Jenkins Workflow with external applications

# Running Docker container
docker run -p 8080:8080 -p 8180:8180 -it uday/workflow-integration

# Instructions

## REST URL Format

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

## Workflow Triggering

### No input required
In this scenario, the workflow doesn't require any input from the user.

To Proceed

```
curl -X POST http://192.168.59.103:8080/job/workflow-integration-noinput/1/input/ApprovalAppnameDeployment/proceedEmpty
```

To Abort

```
curl -X POST http://192.168.59.103:8080/job/workflow-integration-noinput/1/input/ApprovalAppnameDeployment/abort
```

### Input required
In this scenario, the workflow requires input from the user and the data is passed to the InputStep as a combination of key/value pairs and JSON object as shown below.

```
name:comments
value:Approved
json:{"parameter": {"name": "comments", "value": "Approved"}}
proceed:Approve
```

The JSON object above need to be url encoded. The POST request would look something like this

```
name=comments&value=Approved&json=%7B%22parameter%22%3A+%7B%22name%22%3A+%22comments%22%2C+%22value%22%3A+%22Approved%22%7D%7D&proceed=Approve
```

To Proceed

```
curl -d "name=comments&value=Approved&proceed=Approve" --data-urlencode json='{"parameter":{"name":"comments","value":"Approved"}}' http://192.168.59.103:8080/job/workflow-integration/6/input/ApproveDeployment/submit
```

To Abort

```
curl -X POST http://192.168.59.103:8080/job/workflow-integration/1/input/ApprovalAppnameDeployment/submit
```
