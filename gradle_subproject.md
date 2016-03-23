# Gradle subprojects

This is a working way of creating subprojects, which would be very nice
if it wasn't so slow.
Anyone has any ideas?

## File tree

    .
    |- settings.gradle
    |- build.gradle
    |- shared
    |   |- src
    |       |- main
    |           |- java
    |- webApp
    |   |- src
    |       |- main
    |           |- java
    |- worker
        |- src
            |- main
                |- java


## settings.gradle

    rootProject.name = 'aProject'
    include 'worker', 'shared', 'webApp'

## build.gradle

    group 'aGroup'
    version '0.1'

    subprojects {
        apply plugin: 'java'

        sourceCompatibility = 1.8

        repositories {
            mavenCentral()
        }

        // Is this what makes it slow?
        jar {
            from {
                configurations.compile.collect { it.isDirectory() ? it : zipTree(it) }
            }
        }

        task projName << {
            println project.name
        }

        dependencies {
            // To avoid circular dependencies
            if (project.name != 'shared') {
                compile project(':shared')
            }

            // Jersey + Grizzly
            compile 'org.glassfish.jersey.containers:jersey-container-grizzly2-http:2.22.2'

            // Other
            compile 'com.google.code.gson:gson:2.6.2'
            compile 'com.amazonaws:aws-java-sdk:1.10.56'
            compile 'commons-io:commons-io:2.4'

            // Test
            testCompile group: 'junit', name: 'junit', version: '4.11'
        }
    }

    project(':worker') {
        jar {
            manifest {
                attributes 'Main-Class': 'worker.WorkerMain'
            }
        }
    }

    project(':webApp') {

        // To make the subproject runnable with "gradle webApp:run"
        apply plugin: 'application'

        jar {
            manifest {
                attributes 'Main-Class': 'webApp.WebAppMain'
            }
        }
        project.mainClassName = 'webApp.WebAppMain'
    }
