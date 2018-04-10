+++
date = "2018-04-10T00:00:00+00:00"
draft = false
title = "Building Git Tags with Jenkins Pipelines"
+++

There seem to be a lot of questions around making Jenkins [pipelines](https://jenkins.io/doc/book/pipeline-as-code/) work with [git tags](https://git-scm.com/book/en/v2/Git-Basics-Tagging); having set this up recently, I thought I'd write a quick summary of one workable approach.

## Workflow

This post covers a specific workflow used by my team; in particular, we don't do pull requests, and we use tags to denote releases, which are automatically deployed.  We tag our releases manually, so we aren't configuring Jenkins to tag every build, but rather to watch for new tags to be pushed, and execute builds for them.

Our requirements are something like this:

1. Generate a build for every branch, and execute it on creation
1. Execute a build every time changes are pushed to its branch
1. Generate a build whenever a tag is pushed, and execute it

If your workflow is different, you can configure things differently (i.e., to build pull requests), without affecting the way tags are built.

The steps outlined here will work for a GitHub organization; there are [equivalents](https://wiki.jenkins.io/display/JENKINS/Bitbucket+Branch+Source+Plugin) for BitBucket, but I haven't used them yet.

## Plugins and Configuration

We need a couple of plugins in order to discover and build git tags:

* [GitHub Branch Source Plugin](https://github.com/jenkinsci/github-branch-source-plugin)
* [Basic Branch Build Strategies Plugin](https://github.com/jenkinsci/basic-branch-build-strategies-plugin)

Note that the GitHub Branch Source Plugin must be at least version 2.3.0 for tag discovery to work.

Once the plugins are installed, navigate to your top-level GitHub Organization job, go to `Configure`, and you can add `Tag Discovery` from the dropdown under `Behaviors`:

![discover tags](/images/jenkins-tag-discover.png)

Similarly, you can add build strategies for both branches and tags:

![build tags](/images/jenkins-tag-build.png)

The next time you scan your GitHub Organization, you should see jobs generated for each existing tag:

![tag jobs](/images/jenkins-tag-jobs.png)

When you push a new tag, its job will be generated and executed automatically.

## Controlling which Tags are Built

If you're using tags for deployment, you need to put some thought into setting up your build strategy; for example, if you add a Jenkinsfile to a repository that already has a number of tags, they will all get discovered, and jobs will be generated for them.  If you aren't careful with your build strategy, all of those jobs will be run as well!

We've set our system up to only build tags that are less than a day old--so far this has worked well.

## Executing Stages Provisionally

We typically want to run our build and test scripts on every branch, but only deploy artifacts from tags.  You can do this by wrapping the stage in a conditional (for scripted pipelines), or using a `when` block (for declarative pipelines):

```groovy
node {
    stage ("checkout") {
        checkout scm
    }
    stage("test") {
        sh('bin/run_all_tests.sh')
    }
    def tag = sh(returnStdout: true, script: "git tag --contains | head -1").trim()
    if (tag) {
        stage("deploy") {
            sh('bin/build_and_publish.sh')
        }
    }
}
```

We have seen issues getting the `TAG_NAME` environment variable to work, similar to this [issue](https://issues.jenkins-ci.org/browse/JENKINS-34520).  As a workaround, we capture the tag using a shell command.


