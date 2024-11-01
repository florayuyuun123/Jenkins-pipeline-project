# Jenkins Pipeline

## On your GitHub Project

- Create a file on the root directory of your project and name it `"Dockerfile"`
- Get the contents of the Dockerfile in one of the `sample_pipeline` files and paste on your Dockerfile
- Commit the changes

## On Jenkins

### Configure Agent Node

See jenkins_agent:

- devops_tools ==> jenkins ==> jenkins_agents

### Configure pipeline job

- Go to `Dashboard` and click on `+ New Item`
- Enter your job name, select `Pipeline` then click `OK`
- Select `Configure` and under `General`, add your `Description`
- `Discard old builds`, chose your `Max # of builds to keep`
- Select `Pipeline` on the left panel, under `Definition`, select `Pipeline script`
- Paste your script in the provided area, click `Save` then `Build Now`

But if you want to build from your Jenkinsfile on your project, follow the following steps:

**On your GitHub Project:**

- Create a file on the `root directory` of your project and name it `"Jenkinsfile"`
- The naming syntax is very important
- Commit the changes

**On Jenkins:**

- The configuration is same as above
- But on `Definition` under `Pipeline`, select `Pipeline script from SCM`
- Under `SCM` select `Git`
- Paste your repository URL under `Repository`
- Create and Add your credentials making sure to use `Secret text` under `Kind`
- Your `Secret` should be your `GitHub token`
- Under `Branches to build`, indicate the branch you want to build from
- Under `Script Path`, write `Jenkinsfile`
- However, if your jenkinsfile name was a custom name, then you will have to specify the direct path to the file on your project repository.
- Select `Save` then `Build now`
