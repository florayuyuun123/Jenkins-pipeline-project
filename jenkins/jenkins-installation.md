# Jenkins Installation

## Install Jenkins standalone

Server Type: Ubuntu

[Click Here](https://www.jenkins.io/doc/book/installing/linux/#debianubuntu) for installation guidance

Become root:

```python
sudo -i
```

## Update and Upgrade the server

```python
apt update
```

```python
apt upgrade -y
```

## Create a Jenkins user

```python
adduser jenkins
```

Disable password and add `jenkins user` to the Sudoers group:

```python
echo "jenkins ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/jenkins
```

Switch to jenkins user:

```python
su - jenkins
```

## Install Java

```python
sudo apt update
```

```python
sudo apt install openjdk-17-jre -y
```

Check Java version:

```python
java -version
```

## Install Jenkins | Get neccessary plugins

```python
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian/jenkins.io-2023.key
```

```python
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
```

Update the server:

```python
sudo apt update
```

Install Jenkins:

```python
sudo apt install jenkins -y
```

Check Jenkins status:

```python
sudo systemctl status jenkins
```

If jenkins is not running then enable it:

```python
sudo systemctl enable jenkins
```

Now start it:

```python
sudo systemctl start jenkins
```

Then check the status again:

```python
sudo systemctl status jenkins
```

## Open Jenkins on the web

- Default username: admin
- Jenkins default port: 8080
- Get your instance IP from the console
- Then user IP:portNumber to access jenkins on the web.

## Get the Administrator Password to Unlock Jenkins

```python
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

### How to change Jenkins default portNumber

Locate the Jenkins configuration file:

```python
cd /etc/default/
```

```python
sudo nano jenkins
```

Or use the direct path:

```python
sudo nano /etc/default/jenkins
```
  
Look for the line that specifies the `HTTP_PORT` variable.

For example, it may look like `HTTP_PORT=8080`.

Change the port number to the desired value.

Make sure it is a valid and available port.

Add a new line below `HTTP_PORT=-1` to set your desired port using the `--httpPort` option.

This is the new line:

```python
HTTP_PORT=-1
JENKINS_ARGS="--httpPort=8081"
```

Save the changes to the configuration file and exit the text editor.

Restart the Jenkins service to apply the changes.

```python
sudo systemctl restart jenkins
```

Edit the `jenkins.service` file:

`sudo nano /lib/systemd/system/jenkins.service`

Change the port on the file:

`[Service]`

`Environment="JENKINS_PORT=8081"`

Reload the `daemon` and restart jenkins service:

```python
sudo systemctl daemon-reload
```

```python
sudo service jenkins restart
```

## Troubleshooting

- If jenkins fails to integrate with git under `Source Code Management`
- Do the following to resolve this:

On your terminal, edit the `Jenkins DNS Configuration` file using the command:

```python
sudo nano /etc/resolv.conf
```

Add the following line to use Google DNS:

```python
nameserver 8.8.8.8
```

Save the file and restart Jenkins service:

```python
sudo systemctl restart jenkins
```

Go to Jenkins and try the integration again

**If you are using UFW (Uncomplicated Firewall) on Ubuntu, you can allow incoming:**

connections to the port with the following command:

```python
sudo ufw allow 8081/tcp
```
