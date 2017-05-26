---
layout: post
title: Pipeline en Jenkins 2.0 como Código para iOS 10 y XCode 8.
permalink: /blog/construye-pipeline-con-jenkins2-como-codigo-para-ios10-xcode8/
translate_en: /en/blog/build-pipeline-jenkins2-as-code-with-ios10-xcode8/
lang: es
sidebar: yes
promo: yes
category: [articulo]
tags: [jenkins, xcode, ios10]
image: /images/banners/jenkins2-pipeline.jpg
excerpt: Ya puedes definir tus procesos de Integración Continua y Despliegue Continuo <strong>(CI/CD)</strong> como si fuera código fuente con <strong>Jenkins 2.0</strong> para tus proyectos de <strong>iOS 10</strong>. Compilar, realizar pruebas, analizar cobertura y estilo de código, reportes y notificaciones son actividades descritas en un solo fichero.
---

<img src="{{ site.baseurl }}/images/banners/jenkins2-pipeline.jpg" title="Jenkins 2, XCode 8 y iOS 10" name="Jenkins, XCode 8 y iOS 10" />

### Introducción

Ya puedes definir tus procesos de Integración Continua y Despliegue Continuo (CI/CD) como si fuera código fuente con Jenkins 2.0 para tus proyectos en iOS 10. Compilar, realizar pruebas, analizar cobertura y estilo de código, reportes y notificaciones son actividades descritas en un solo fichero.

### Explicar cómo funciona

Una de las metas de DevOps es poder definir nuestros procesos de Integración Continua y Despliegue Continuo de manera tal que puedan ser documentados, descritos y modificados fácilmente.

Jenkins a apoyado esta filosofía de trabajo al incluir el fichero `Jenkinsfile` junto al grupo de módulos <a target="_blank" href="https://jenkins.io/doc/book/pipeline/">Pipeline</a>. El fichero `Jenkinsfile` se utiliza para describir la secuencia de pasos que va a realizar el sistema a través de los módulos Pipeline. La información existente en el sitio <a target="_blank" href="https://jenkins.io/solutions/pipeline/">Jenkins.io</a> puede ser consultada para profundizar los detalles.

### Time Table: Un proyecto de ejemplo

Time Table es un proyecto de ejemplo para mostrar cómo podemos estructurar nuestro proceso de CI/CD para un proyecto en iOS 10.

### Código fuente

El código fuente puede ser <a target="_blank" href="https://github.com/mmorejon/time-table">descargado o clonado desde Github</a>.

### Entorno

La configuración del entorno donde fue desarrollado el artículo es la siguiente:

```
### Sistema de integración continua ###
             SO: Ubuntu 14.04
        Jenkins: 2.19.3

### Nodo de Jenkins ###
             SO: Mac OS 10.12.2
          XCode: 8.1

### Proyecto iOS 10.1 ###
       Lengueje: Swift 3
```

## Configurando Jenkinsfile

A continuación le mostramos el código que usted necesita incluir en su proyecto para configurar su pipeline completamente. Cree un fichero en la raíz de su proyecto llamado 
`Jenkinsfile` y `copie - pegue` el código que aparece a continuación. Si esto es lo que usted necesitaba, aquí va:

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
}
```


## Comprendiendo Jenkinsfile  

Veamos paso a paso lo que significa cada sección del fichero `Jenkinsfile`.

### Especificar el nodo

```
node('iOS Node') {	
	......
}
```
El nodo de Jenkins tiene que tener sistema operativo Mac OS 10 con XCode 8.

### Definición de trabajos

Tareas realizadas de manera secuencia: obtener código, compilarlo y reliazarle pruebas.

```
stage('Checkout/Build/Test') {
	......
}
```

Tareas a realizase en paralelo: análisis de pruebas y estilo de código.

```
stage('Analytics') {

	parallel Coverage: {
		......
	}, Checkstyle: {
		......
	}, failFast: true|false

}
```

Jenkins agrupa las tareas en `stage`. Las tareas pueden ser ejecutadas de manera secuencial o en paralelo según sea el caso. El fichero `Jenkinsfile` muestra ambos ejemplos.

### Obtener código fuente

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

El Pipeline SCM Step Plugin es el encargado de obtener el código fuente del repositorio de GitHub.

### Compilar y hacer pruebas

```
// Build and Test
sh 'xcodebuild -scheme "TimeTable" -configuration "Debug" build test -destination "platform=iOS Simulator,name=iPhone 6,OS=10.1" -enableCodeCoverage YES | /usr/local/bin/xcpretty -r junit'
```

Para compilar el proyecto se utiliza la herramienta `xcodebuild`. El resto de los parámetros son ajustado según los datos del proyecto.

Durante la ejecución de los test `xcpretty` se responsabiliza de transformarlos a un fichero estándar de JUnit para ser consultados posteriormente. El fichero se genera en la dirección `build/reports/junit.xml`.

Necesita tener instalado <a target="_blank" href="https://github.com/supermarin/xcpretty">Xcpretty</a> para poder obtener los reportes de pruebas.

### Publicar resultados de las pruebas

```
// Publish test restults.
step([$class: 'JUnitResultArchiver', allowEmptyResults: true, testResults: 'build/reports/junit.xml'])
```

El <a target="_blank" href="https://wiki.jenkins-ci.org/display/JENKINS/JUnit+Plugin">Plugin JUnit</a> es el responsable de mostrar los resultados de las pruebas.

Necesita tener instalado el <a target="_blank" href="https://wiki.jenkins-ci.org/display/JENKINS/JUnit+Plugin">Plugin JUnit</a> para poder publicar los reportes de pruebas.

### Cobertura de código

```
// Generate Code Coverage report
sh '/usr/local/bin/slather coverage --jenkins --html --scheme TimeTable TimeTable.xcodeproj/'
```

Para generar el reporte de cobertura del código se utiliza <a target="_blank" href="https://github.com/SlatherOrg/slather">Slather</a>. Slather puede ser parametrizado para obtener el reporte en formato `html` en la ubicación `./html/index.html`

Necesita tener instalado <a target="_blank" href="https://github.com/SlatherOrg/slather">Slather</a> para poder generar este reporte.

### Publicar reporte de cobertura de código

```
// Publish coverage results
publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'html', reportFiles: 'index.html', reportName: 'Coverage Report'])
```

El <a target="_blank" href="https://wiki.jenkins-ci.org/display/JENKINS/HTML+Publisher+Plugin">Plugin HTML Publisher</a> se utiliza para publicar el reporte de cobertura del código.

Necesita tener instalado el <a target="_blank" href="https://wiki.jenkins-ci.org/display/JENKINS/HTML+Publisher+Plugin">Plugin HTML Publisher</a> para utilizar esta funcionalidad.

### Generar reporte de estilo de código

```
// Generate Checkstyle report
sh '/usr/local/bin/swiftlint lint --reporter checkstyle > checkstyle.xml || true'
```

<a target="_blank" href="https://github.com/realm/SwiftLint">SwiftLint</a> se ha utilizado para evaluar el estilo de código. El reporte es generado utilizando el formato `checkstyle` y se almacena el el fichero `checkstyle.xml`.

Necesita tener instalado <a target="_blank" href="https://github.com/realm/SwiftLint">SwiftLint</a> para utilizar esta funcionalidad.

### Publicar reporte de estilo de código

```
// Publish checkstyle result
step([$class: 'CheckStylePublisher', canComputeNew: false, defaultEncoding: '', healthy: '', pattern: 'checkstyle.xml', unHealthy: ''])
```

El <a target="_blank" href="https://wiki.jenkins-ci.org/display/JENKINS/Checkstyle+Plugin">Plugin Checkstyle</a> se ha utilizado para mostrar los reportes de estilo de código. Se utiliza el fichero `checkstyle.xml` generado por <a target="_blank" href="https://github.com/realm/SwiftLint">SwiftLint</a>.

### Enviar notificación de Slack

```
// Send slack notification
slackSend channel: '#my-team', message: 'Time Table - Successfully', teamDomain: 'my-team', token: 'my-token'
```

El <a target="_blank" href="https://wiki.jenkins-ci.org/display/JENKINS/Slack+Plugin">Plugin Slack Notification</a> se ha utilizado para enviar notificaciones al canal del equipo. El plugin debe ser configurado según los datos de la cuenta de Slack que utilice el equipo. Los valores que deben ser modificados son: `channel`, `message`, `teamDomain` y `token`.

Necesita tener instalado el <a target="_blank" href="https://wiki.jenkins-ci.org/display/JENKINS/Slack+Plugin">Plugin Slack Notification</a> para enviar las notificaciones.

## Configurar el trabajo en Jenkins

**Crear nuevo trabajo**

Crea un nuevo trabajo en Jenkins. Coloque el nombre de `time-table`, seleccione la opción **Pipeline** y de clic en el botón **OK**.

**Configure los datos del Pipeline**

Los datos relacionados con el módulo Pipeline deben quedar como muestra la imagen.

```
      Definition: Pipeline script from SCM
             SCM: Git
    Repositories: https://github.com/mmorejon/time-table.git
Branch Specifier: master
     Script Path: Jenkinsfile
```

<img src="{{ site.baseurl }}/images/jenkins2-pipeline-ios10-xcode8/jenkins-pipeline-configuration.jpg" title="Jenkins 2.0 Configuración Pipeline" name="Jenkins 2.0 Configuración Pipeline" />

## Ejecutar el trabajo en Jenkins

Ejecute el trabajo en Jenkins y podrá obtener resultados similares a las imágenes que se muestran a continuación.

**_Vista tradicional de Jenkins_**

<img src="{{ site.baseurl }}/images/jenkins2-pipeline-ios10-xcode8/jenkins2-pipeline-stage-view.jpg" title="Jenkins 2.0 Vista de Estado" name="Jenkins 2.0 Vista de Estado" />

<img src="{{ site.baseurl }}/images/jenkins2-pipeline-ios10-xcode8/jenkins2-pipeline-test-result.jpg" title="Jenkins 2.0 Resultados de las Pruebas" name="Jenkins 2.0 Resultados de las Pruebas" />

<img src="{{ site.baseurl }}/images/jenkins2-pipeline-ios10-xcode8/jenkins2-pipeline-checkstyle.jpg" title="Jenkins 2.0 Estilo de Código" name="Jenkins 2.0 Estilo de Código" />

**_Vista Blue Ocean UI de Jenkins_**

<img src="{{ site.baseurl }}/images/jenkins2-pipeline-ios10-xcode8/jenkins2-pipeline-blue-ocean.jpg" title="Jenkins 2.0 Vista de Blue Ocean" name="Jenkins 2.0 Vista de Blue Ocean" />


## Conclusiones

Ahora usted sabe cómo escribir sus propios procesos de CI/CD utilizando el módulo Pipeline de Jenkins 2.0. Es su turno de construir el fichero `Jenkinsfile` que necesita su equipo.

