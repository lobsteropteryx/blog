+++
date = "2018-06-16T00:00:00+00:00"
draft = false
title = "Deploying from a Git Tag with Jenkins Pipelines"
+++

In a previous [post](/post/building-git-tags-with-jenkins) I outlined a workflow to create and trigger a pipeline job in Jenkins whenever a git tag is pushed.  A common step in this type of workflow is to deploy to a staging environment once all the build and test steps are successful.

One way to accomplish this is by using parameters--a typical (declarative) job definition might look like:

```groovy
pipeline {
    agent { label 'myLabel' }
    parameters {
        choice(choices: 'staging\nproduction', description: 'Which environment?', name: 'ENVIRONMENT')
    }
    environment {
        GIT_TAG = sh(returnStdout: true, script: 'git describe --always').trim()
    }
    stages {
        stage("Checkout") {
            steps {
                checkout scm
	    }
        }
        stage("Test") {
            steps {
                sh('make test')
	    }
        }
        stage("Deploy to Staging") {
            when {
                buildingTag()
                environment name: 'ENVIRONMENT', value: 'staging'
            }
            steps {
                sh('make deploy')
	    }
        }
    }
}
```

The above job will run a test suite for every push, and deploy to a staging environment if the commit is a tag.  The environment is based on a `choice` parameter, and the default value is the first item in the newline-separated list (in this case, `staging`).

## Dealing with Null Parameters on Initial Run

Since we want our tag job to run as soon as it's created, we have to deal with a minor issue:  Our `ENVIRONMENT` parameter may not be set on the initial run (see issues [JENKINS-40574](https://issues.jenkins-ci.org/browse/JENKINS-40574) and [JENKINS-41929](https://issues.jenkins-ci.org/browse/JENKINS-41929)).  

In the case of deploying to staging, we can test for a null `ENVIRONMENT`, and set it in an     environment block:

```groovy
        stage("Deploy to Staging") {
            when {
                buildingTag()
                anyOf {
                    environment name: 'ENVIRONMENT', value: 'staging'
                    environment name: 'ENVIRONMENT', value: null
                }
            }
            environment {
                ENVIRONMENT = 'staging'
            }
            steps {
                sh('make deploy')
	    }
	}
```

## Promoting to Production
Once the changes have been evaluated in the staging environment, the same tag can be pushed to production by adding another stage:

```groovy
        stage("Deploy to Production") {
            when {
                buildingTag()
                environment name: 'ENVIRONMENT', value: 'production'
            }
            steps {
                sh('make deploy')
	    }
	}
```

In this stage we don't need to check for a null `ENVIRONMENT`, since the first run of the job will always go to staging.

This setup makes sense if you have a QA team doing exploratory testing in the staging environment, and you want a manually triggered deploy to production.  You can deploy to production by running the existing tag job from Jenkins, and setting the `ENVIRONMENT` parameter to `production` in the dropdown:

![production deploy](/images/jenkins-deployment.png)

## Automatic Promotion
If you want to evaluate your changes in the staging environment automatically via a smoketest script, and push to production on a successful test, you don't need a parameter at all; you can set the `ENVIRONMENT` for each stage in the pipeline:


```groovy
pipeline {
    agent { label 'myLabel' }
    environment {
        GIT_TAG = sh(returnStdout: true, script: 'git describe --always').trim()
    }
    stages {
        stage("Checkout") {
            steps {
                checkout scm
            }
        }
        stage("Test") {
            steps {
                sh('make test')
	    }
        }
	stage("Deploy to Staging") {
	    when {
		buildingTag()
	    }
	    environment {
		ENVIRONMENT = 'staging'
	    }
	    steps {
		sh('make deploy')
	    }
	}
	stage("Smoketest") {
	    steps {
                sh('make smoketest')
	    }
        }
	stage("Deploy to Production") {
	    when {
	        buildingTag()
            }
            environment {
	        ENVIRONMENT = 'production'
            }
            steps {
	        sh('make deploy')
            }
        }
    }
}
```

This approach forgoes any exploratory testing, but can make sense for simple systems.

Another option is to combine the two approaches.  If there are existing manual tests that are repetitive (i.e., making the same requests against URLs), a smoketest script can still offload work from the QA team; in this case, you might automatically deploy to staging, run your test scripts, and then perform an additional deployment to a QA environment for manual testing.  The final step would be a manual promotion to production, as outlined in the previous section.
