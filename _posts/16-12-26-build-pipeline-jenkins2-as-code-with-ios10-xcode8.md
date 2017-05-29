---
layout: post_en
title: Pipeline in Jenkins 2.0 as Code for iOS 10 and XCode 8
permalink: /en/blog/build-pipeline-jenkins2-as-code-with-ios10-xcode8/
translate_es: /blog/construye-pipeline-con-jenkins2-como-codigo-para-ios10-xcode8/
lang: en
sidebar: yes
promo: yes
category: [articulo]
tags: [jenkins, xcode, ios10]
image: /images/banners/jenkins2-pipeline.jpg
excerpt: Now you can define Continuous Integration and Continuous Delivery <strong>(CI/CD)</strong> process as code with <strong>Jenkins 2.0</strong> for your projects in <strong>iOS 10</strong>. Activities like to build, test, code coverage, check style, reports and notifications can be described in only one file.
---

<img src="{{ site.baseurl }}/images/banners/jenkins2-pipeline.jpg" title="Jenkins, XCode 8 y iOS 10" name="Jenkins, XCode 8 y iOS 10" />

### Introduction

Now you can define Continuous Integration and Continuous Delivery (CI/CD) process as code with Jenkins 2.0 for your projects in iOS 10. Activities like to build, test, code coverage, check style, reports and notifications can be described in only one file.

### What is the idea?

One of the DevOps goals it to build process of CI/CD with the characteristics that can be written ones and run always.

When you write your process you avoid the human error and can track all changes over the time. You can learn from your errors and improve your next steps.

Jenkins support this philosophy of work when including the `Jenkinsfile` file along with <a target="_blank" href="https://jenkins.io/doc/book/pipeline/">Pipeline modules</a>. The `Jenkinsfile` file is used to describe all step needed in your workflow. The site <a target="_blank" href="https://jenkins.io/solutions/pipeline/">Jenkins.io</a> have a lot of information related to this topic but now, we are going to become dirty our hands with a real example.

### Time Table: An example project

Time Table is an example to show how can we model our CI/CD process for iOS 10 projects.

### Source Code

The source code can be <a target="_blank" href="https://github.com/mmorejon/time-table">cloned or downloaded from GitHub</a> to test it.

### Environment

The environment configuration used for this article is the following:

```
### Continuous Integration System ###
             SO: Ubuntu 14.04
        Jenkins: 2.19.3

### Jenkins Node ###
             SO: Mac OS 10.12.2
          XCode: 8.1

### Project iOS 10.1 ###
       Lengueje: Swift 3
```

## Setting Up Jenkinsfile

The following lines will show what do you need to include in your iOS 10 project to setting up the pipeline. First of all, create a new file with the name `Jenkinsfile` in the project root and after adding the code behind to `Jenkinsfile` archive. It is simple, right?

```
node('iOS Node') {

    stage('Checkout/Build/Test') {

        // Checkout files.
        checkout([
            $class: 'GitSCM',
            branches: [[name: 'master']],
            doGenerateSubmoduleConfigurations: false,
            extensions: [], submoduleCfg: [],
            userRemoteConfigs: [[
                name: 'github',
                url: 'https://github.com/mmorejon/time-table.git'
            ]]
        ])

        // Build and Test
        sh 'xcodebuild -scheme "TimeTable" -configuration "Debug" build test -destination "platform=iOS Simulator,name=iPhone 6,OS=10.1" -enableCodeCoverage YES | /usr/local/bin/xcpretty -r junit'

        // Publish test restults.
        step([$class: 'JUnitResultArchiver', allowEmptyResults: true, testResults: 'build/reports/junit.xml'])
    }

    stage('Analytics') {
        
        parallel Coverage: {
            // Generate Code Coverage report
            sh '/usr/local/bin/slather coverage --jenkins --html --scheme TimeTable TimeTable.xcodeproj/'
    
            // Publish coverage results
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'html', reportFiles: 'index.html', reportName: 'Coverage Report'])
        
            
        }, Checkstyle: {

            // Generate Checkstyle report
            sh '/usr/local/bin/swiftlint lint --reporter checkstyle > checkstyle.xml || true'
    
            // Publish checkstyle result
            step([$class: 'CheckStylePublisher', canComputeNew: false, defaultEncoding: '', healthy: '', pattern: 'checkstyle.xml', unHealthy: ''])
        }, failFast: true|false   
    }

    stage ('Notify') {
        // Send slack notification
        slackSend channel: '#my-team', message: 'Time Table - Successfully', teamDomain: 'my-team', token: 'my-token'
    }
}
```

## Understanding Jenkinsfile  

Specify the node

```
node('iOS Node') {  
    ......
}
```

The Jenkins node must have installed Mac OS 10 with XCode 8.

### Task definitions

Sequential tasks: checkout code, build, test and notify.

```
stage('Checkout/Build/Test') {
    ......
}

stage ('Notify') {
    ......
}
```

Parallel tasks: code coverage and check style.

```
stage('Analytics') {

    parallel Coverage: {
        ......
    }, Checkstyle: {
        ......
    }, failFast: true|false

}
```

Jenkins group tasks in `stages`. This tasks can be run as the sequential or parallel process depends on the case. The `Jenkinsfile` file show both examples.

### Checkout source code

```
// Checkout files.
checkout([
    $class: 'GitSCM',
    branches: [[name: 'master']],
    doGenerateSubmoduleConfigurations: false,
    extensions: [], submoduleCfg: [],
    userRemoteConfigs: [[
        name: 'github',
        url: 'https://github.com/mmorejon/time-table.git'
    ]]
])
```

The Pipeline SCM Step Plugin get the source code from GitHub.

### Build and test

```
// Build and Test
sh 'xcodebuild -scheme "TimeTable" -configuration "Debug" build test -destination "platform=iOS Simulator,name=iPhone 6,OS=10.1" -enableCodeCoverage YES | /usr/local/bin/xcpretty -r junit'
```

The project is compiled using `xcodebuild` tool. Parameters like `scheme`, `configuration` and `destination` must be setting up depending of the project information.

During the tests execution `xcpretty` transform the tests result into a standard JUnit file to be consulted. The file is generated in the following location: `build/reports/junit.xml`.

You must have installed <a target="_blank" href="https://github.com/supermarin/xcpretty">Xcpretty</a> to work with tests.

### Publish test results

```
// Publish test restults.
step([$class: 'JUnitResultArchiver', allowEmptyResults: true, testResults: 'build/reports/junit.xml'])
```

The <a target="_blank" href="https://wiki.jenkins-ci.org/display/JENKINS/JUnit+Plugin">Plugin JUnit</a> show the tests result in a tables.

You must have installed <a target="_blank" href="https://wiki.jenkins-ci.org/display/JENKINS/JUnit+Plugin">Plugin JUnit</a> to publish tests reports.

### Code Coverage

```
// Generate Code Coverage report
sh '/usr/local/bin/slather coverage --jenkins --html --scheme TimeTable TimeTable.xcodeproj/'
```

<a target="_blank" href="https://github.com/SlatherOrg/slather">Slather</a> generate the code coverage report. Slater can be configured to show the report in `html` format and saved in the following location: `./html/index.html`.

You must have installed <a target="_blank" href="https://github.com/SlatherOrg/slather">Slather</a> to generate code coverage reports.

### Publish code coverage report

```
// Publish coverage results
publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'html', reportFiles: 'index.html', reportName: 'Coverage Report'])
```

The <a target="_blank" href="https://wiki.jenkins-ci.org/display/JENKINS/HTML+Publisher+Plugin">Plugin HTML Publisher</a> is used to publish the code coverage reports.

You must have installed <a target="_blank" href="https://wiki.jenkins-ci.org/display/JENKINS/HTML+Publisher+Plugin">Plugin HTML Publisher</a> to publish `index.html` file generated by Slather.

### Generate checkstyle report

```
// Generate Checkstyle report
sh '/usr/local/bin/swiftlint lint --reporter checkstyle > checkstyle.xml || true'
```

<a target="_blank" href="https://github.com/realm/SwiftLint">SwiftLint</a> is used to evaluate the source code. The report is generated in `checkstyle` and stored in the `checkstyle.xml` file under the project root folder.

You must have installed <a target="_blank" href="https://github.com/realm/SwiftLint">SwiftLint</a> to generate checkstyle reports.

### Chechstyle publisher report

```
// Publish checkstyle result
step([$class: 'CheckStylePublisher', canComputeNew: false, defaultEncoding: '', healthy: '', pattern: 'checkstyle.xml', unHealthy: ''])
```

The <a target="_blank" href="https://wiki.jenkins-ci.org/display/JENKINS/Checkstyle+Plugin">Checkstyle Plugin</a> is used to publish the reports generated by SwiftLint.

You must have installed <a target="_blank" href="https://wiki.jenkins-ci.org/display/JENKINS/Checkstyle+Plugin">Checkstyle Plugin</a> to show SwiftLint reports.

### Send Slack notification

```
// Send slack notification
slackSend channel: '#my-team', message: 'Time Table - Successfully', teamDomain: 'my-team', token: 'my-token'
```

The <a target="_blank" href="https://wiki.jenkins-ci.org/display/JENKINS/Slack+Plugin">Slack Notification Plugin</a> is used to send notifications to channel team. The plugin must be configured according to Slack account and channel team. The values you need setup are `channel`, `message`, `teamDomain` and `token`.

You must have installed <a target="_blank" href="https://wiki.jenkins-ci.org/display/JENKINS/Slack+Plugin">Slack Notification Plugin</a> to send notifications.

## Setting up Jenkins job

**Create new Job**

Create a new Jenkins job with the name `time-table` and selecting **Pipeline** option. After do click **OK** button.

**Setting Up Pipeline**

The Pipeline configuration must be the same like the following image:

```
      Definition: Pipeline script from SCM
             SCM: Git
    Repositories: https://github.com/mmorejon/time-table.git
Branch Specifier: */master
     Script Path: Jenkinsfile
```

<img src="{{ site.baseurl }}/images/jenkins2-pipeline-ios10-xcode8/jenkins-pipeline-configuration.jpg" title="Jenkins 2.0 Pipeline Configuration" name="Jenkins 2.0 Pipeline Configuration" />

## Build Job

Run `time-table` job twice and see the results. The results are like this images, right?

**Jenkins traditional UI**

<img src="{{ site.baseurl }}/images/jenkins2-pipeline-ios10-xcode8/jenkins2-pipeline-stage-view.jpg" title="Jenkins 2.0 Stage View" name="JJenkins 2.0 Stage View" />

<img src="{{ site.baseurl }}/images/jenkins2-pipeline-ios10-xcode8/jenkins2-pipeline-test-result.jpg" title="Jenkins 2.0 Tests Results" name="Jenkins 2.0 Tests Results" />

<img src="{{ site.baseurl }}/images/jenkins2-pipeline-ios10-xcode8/jenkins2-pipeline-checkstyle.jpg" title="Jenkins 2.0 Checkstyle" name="Jenkins 2.0 Pipeline Checkstyle" />

**Jenkins Blue Ocean UI**

<img src="{{ site.baseurl }}/images/jenkins2-pipeline-ios10-xcode8/jenkins2-pipeline-blue-ocean.jpg" title="Jenkins 2.0 Pipeline Blue Ocean UI" name="Jenkins 2.0 Pipeline Blue Ocean UI" />

## Conclusion

Now you know how to write your own **CI/CD** process using **Pipeline Modules** in **Jenkins 2.0**. It's your turn to build the  `Jenkinsfile`  that needs your team.
