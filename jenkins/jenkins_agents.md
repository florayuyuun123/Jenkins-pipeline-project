# Setting up an Agent

Note:

- To run a `pipeline job`, you need to setup an `agent`.
- Install same `java version` on both the `master node` and `agent node`
- Never have the `number of executors` exceed the number of `CPUs` or `vCPUs` on the agent
- In the case of using `docker` as an agent, you can run the container in the same machine that `jenkins` is running.

## Setting up Docker as agent

### On Jenkins

- Got to `Manage Jenkins`, then `Nodes` and open `+New Node`.
- Inout your `Node name` and select `Permanent Agent` under `Type` then click `Create`.

**Configuring the Agent:**

- Name: `agent1`
- Description: Input your description
- Number of executors: `1`
- Remote root directory: `/home/jenkins/agent`
- Labels: `agent1`
- Usage: `Use this node as much as possible`
- Launch method: `Launch agent by connecting it to the controller`
- Availability: `Keep this agent online as much as possible`
- Now click on `Save`

**Make Agent Online:**

- Go to `Manage Jenkins` and then select `Security`
- Scroll down to `Agents`
- Under `TCP port for inbound agents` select `Fixed`, and set port to default `JNLP port` which is `50000`

### On the command line where you want your node running

Become jenkins:

```python
sudo su - jenkins
```

Pull the docker image:

```python
docker pull jenkins/inbound-agent:latest
```

Run the docker container:

```python
docker run -d --name jenkins-agent \
  -e JENKINS_URL=http://your-jenkins-server/ \
  -e JENKINS_SECRET=your-agent-secret \
  -e JENKINS_AGENT_NAME=your-agent-name \
  jenkins/inbound-agent:latest
```

- Replace `http://your-jenkins-server/` with the URL of your Jenkins server.
- Replace `your-agent-secret` with the secret token from the Jenkins agent configuration.
- Replace `your-agent-name` with the desired name for your agent.

Also make sure to run the commands given given to you when you created the agent in `Jenkins`, to make the agent come online. The credentials needed to run the container are found in these commands.

Also, make sure to put off your master node (Buil-In Node) for this can be a security issue.

- Go to `Dashboard` and then `Nodes`, then `Built-In Node` and finally `Configure`
- Number of executors: `0`
- Labels: `any strange name`
- Usage: `Only build jobs with label expressions matching this node`
- Now click `Save`

## Setting up a VM agent

### Pre-requisite

Install Java in VM where the agent will run, as Jenkins requires Java to run.

```python
sudo apt update
sudo apt install openjdk-11-jdk
java -version
```

- Install Git, Maven, and Docker
- Add user to the Docker group so that the user can run docker commands:

```python
sudo usermod -aG docker $USER
```

### Create a directory for the Jenkins agent

```python
mkdir -p ~/jenkins
cd ~/jenkins
```

- See above for additional steps like:
- Configuring agent on Jenkisn
- Make agent online

## Troubleshooting Jenkins Agent

**Problem:**

- If agent fails to download the jar file with the following command:
  
```python
curl -sO http://18.206.56.228:8080/jnlpJars/agent.jar
```

**Solution:**

- Jenkins is surely to be running on an instance, and the said instance was stopped and is now restarted.
- When an instance is restarted after being on a stopped state, then the instance IP will change.
- Replace the public IP in the above command with the new Jenkins IP
- Also replace the IP on the command to start the agent

```python
java -jar agent.jar -url http://18.206.56.228:8080/ -secret 5953e88ad01424a0f9fe98634c1b57c343db4cf5dba8c0c0c37099dd2d1a5e6c -name agent1 -workDir "/home/vin-24/jenkins/agent1"
```

**Problem:**

- Jenkins pipeline failed to build with the error code:

```python
+ mvn clean package
/home/kinsjen/agent3/workspace/init-app@tmp/durable-d6fd8a7e/script.sh.copy: 1: mvn: not found
script returned exit code 127
```

**Solution:**

- Get the direct `/path/to/maven/bin/` on your agent server with the following command:

```python
readlink -f $(which mvn)
```

Replace the path with your `build step` on your `Jenkinsfile`:

- `sh 'mvn clean package'`  // Simple Maven build
- `sh '/opt/apache-maven-3.9.6/bin/mvn clean package'`  // Simple Maven build
