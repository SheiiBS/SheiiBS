---
layout: post_en
title: Build your Pipeline in Jenkins 2.0 as Code for iOS9 and XCode7
permalink: /en/blog/build-pipeline-jenkins2-as-code-with-ios9-xcode7/
translate_es: /blog/construye-pipeline-con-jenkins2-como-codigo-para-ios9-xcode/
lang: en
sidebar: yes
category: [articulo]
tags: [jenkins, xcode, ios9]
image: /images/banners/jenkins-og.jpeg
excerpt: <strong><em>Automatizar las pruebas en proyectos para iOS 9 es posible!!!</em></strong> Delegar la ejecución de casos de pruebas a máquinas de mayor rendimiento <strong><em>simplifica el proceso de desarrollo</em></strong>.
---

<img src="{{ site.baseurl }}/images/banners/jenkins2-pipeline.jpg" title="Jenkins, XCode 7 y iOS 9" name="Jenkins, XCode 7 y iOS 9" />

### Introduction

Now you can define **Continuous Integration** and **Continuous Delivery** (CI/CD) process as code with **Jenkins 2.0** for your projects in **iOS 9**. Activities like **build**, **test**, **code coverage**, **check style**, **reports** and **notifications** can be described in only one file.

### What is the idea?

One of the **DevOps** goals are build process of **CI/CD** with the characteristics that can be written ones and runned always.

We you can write your process you avoid the human error and can track all changes over the time. You can learn from your errors and improve your next steps.

**Jenkins** support this philosophy of work when include the `Jenkinsfile` file along with <a target="_blank" href="https://jenkins.io/doc/book/pipeline/">Pipeline modules</a>. The `Jenkinsfile` file is used to describe all step needed in your workflow. The site <a target="_blank" href="https://jenkins.io/solutions/pipeline/">Jenkins.io</a> have a lot of information related to this topic but now, we are going to become dirty our hands with a real example.

### Time Table: An example project

Time Table is an example to show how can we model our **CI/CD** process for **iOS9** projects.

### Source Code

The source code can be <a target="_blank" href="https://github.com/mmorejon/time-table">cloned or downloaded from GitHub</a> to test it.

## Setting Up Jenkinsfile

The following lines will show what do you need to include in your iOS9 project to setting up the pipeline. First of all, create a new file with the name `Jenkinsfile` in the project root and after add the code behind to `Jenkinsfile` archive. It is simple, right?

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
        sh 'xcodebuild -scheme "TimeTable" -configuration "Debug" build test -destination "platform=iOS Simulator,name=iPhone 6,OS=9.3" -enableCodeCoverage YES | /usr/local/bin/xcpretty -r junit'

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
}
```
<br>

## Understanding Jenkinsfile  

Specify the node

```
node('iOS Node') {  
    ......
}
```

The Jenkins node must have installed **Mac OS** with **XCode7**.
<br><br>

Task definitions

Secuencial tasks: checkout code, build and test.

```
stage('Checkout/Build/Test') {
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

Jenkins group tasks in `stages`. This tasks can be runned as secuencial or parallel process depends on the case. The `Jenkinsfile` file show both examples.
<br><br>

Checkout source code

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
<br><br>

Build and test

```
// Build and Test
sh 'xcodebuild -scheme "TimeTable" -configuration "Debug" build test -destination "platform=iOS Simulator,name=iPhone 6,OS=9.3" -enableCodeCoverage YES | /usr/local/bin/xcpretty -r junit'
```

The project is compiled using `xcodebuild` tool. Parameters like `scheme`, `configuration` and `destination` must be setting up depending of the project information.

During the tests execution `xcpretty` transform the tests result into a standart **JUnit** file to be consulted. The file is generated in the following location: `build/reports/junit.xml`.

**You must have installed** <a target="_blank" href="https://github.com/supermarin/xcpretty">Xcpretty</a> to work with tests.
<br><br>

Publish test results

```
// Publish test restults.
step([$class: 'JUnitResultArchiver', allowEmptyResults: true, testResults: 'build/reports/junit.xml'])
```

The <a target="_blank" href="https://wiki.jenkins-ci.org/display/JENKINS/JUnit+Plugin">Plugin JUnit</a> show the tests result in a tables.

**You must have installed** <a target="_blank" href="https://wiki.jenkins-ci.org/display/JENKINS/JUnit+Plugin">Plugin JUnit</a> to publish tests reports.
<br><br>

Code Coverage

```
// Generate Code Coverage report
sh '/usr/local/bin/slather coverage --jenkins --html --scheme TimeTable TimeTable.xcodeproj/'
```

<a target="_blank" href="https://github.com/SlatherOrg/slather">Slather</a> generate the code coverage report. Slater can be configured to show the report in `html` format and saved in the following location: `./html/index.html`.

**You must have installed** <a target="_blank" href="https://github.com/SlatherOrg/slather">Slather</a> to generate code coverage reports.
<br><br>

Publish code coverage report

```
// Publish coverage results
publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'html', reportFiles: 'index.html', reportName: 'Coverage Report'])
```

The <a target="_blank" href="https://wiki.jenkins-ci.org/display/JENKINS/HTML+Publisher+Plugin">Plugin HTML Publisher</a> is used to publish the code coverage reports.

**You must have installed** <a target="_blank" href="https://wiki.jenkins-ci.org/display/JENKINS/HTML+Publisher+Plugin">Plugin HTML Publisher</a> tu publish `index.html` file generated by Slather.
<br><br>

Generate checkstyle report

```
// Generate Checkstyle report
sh '/usr/local/bin/swiftlint lint --reporter checkstyle > checkstyle.xml || true'
```

<a target="_blank" href="https://github.com/realm/SwiftLint">SwiftLint</a> is used to evaluate the source code. The report is generated in `checkstyle` and stored in the `checkstyle.xml` file under the project root folder.

**You must have installed** <a target="_blank" href="https://github.com/realm/SwiftLint">SwiftLint</a> to generate checkstyle reports.
<br><br>

Chechstyle publisher report

```
// Publish checkstyle result
step([$class: 'CheckStylePublisher', canComputeNew: false, defaultEncoding: '', healthy: '', pattern: 'checkstyle.xml', unHealthy: ''])
```

The <a target="_blank" href="https://wiki.jenkins-ci.org/display/JENKINS/Checkstyle+Plugin">Checkstyle Plugin</a> is used to publish the reports generated by SwiftLint.

**You must have installed** <a target="_blank" href="https://wiki.jenkins-ci.org/display/JENKINS/Checkstyle+Plugin">Checkstyle Plugin</a> to show SwiftLint reports.
<br><br>

## Setting up Jenkins job

Create new Job

Create a new Jenkins job with the name `time-table` and selecting **Pipeline** option. After do click in **OK** button.
<br><br>

Setting Up Pipeline

The **Pipeline** configuration must be the same like the following image:

```
      Definition: Pipeline script from SCM
             SCM: Git
    Repositories: https://github.com/mmorejon/time-table.git
Branch Specifier: master
     Script Path: Jenkinsfile
```

<img src="{{ site.baseurl }}/images/jenkins2-pipeline-ios9-xcode/jenkins-pipeline-configuration.jpg" title="Jenkins 2.0 Pipeline Configuration" name="Jenkins 2.0 Pipeline Configuration" />

<br><br>

## Build Job

Run `time-table` job twice and see the results. The results are like this images, right?

Jenkins traditional UI

<img src="{{ site.baseurl }}/images/jenkins2-pipeline-ios9-xcode/jenkins2-pipeline-stage-view.jpg" title="Jenkins 2.0 Pipeline Configuration" name="Jenkins 2.0 Pipeline Configuration" />

<img src="{{ site.baseurl }}/images/jenkins2-pipeline-ios9-xcode/jenkins2-pipeline-test-result.jpg" title="Jenkins 2.0 Pipeline Configuration" name="Jenkins 2.0 Pipeline Configuration" />

<img src="{{ site.baseurl }}/images/jenkins2-pipeline-ios9-xcode/jenkins2-pipeline-checkstyle.jpg" title="Jenkins 2.0 Pipeline Configuration" name="Jenkins 2.0 Pipeline Configuration" />

<br><br>

Jenkins Blue Ocean UI

<img src="{{ site.baseurl }}/images/jenkins2-pipeline-ios9-xcode/jenkins2-pipeline-blue-ocean.jpg" title="Jenkins 2.0 Pipeline Configuration" name="Jenkins 2.0 Pipeline Configuration" />
