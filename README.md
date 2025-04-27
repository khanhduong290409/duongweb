plugins {
    id 'java-library'
    id 'maven-publish'
    id "org.hidetake.ssh" version "2.11.2"
    id 'war'

}
apply plugin: 'org.hidetake.ssh'
remotes {
    host {
        host = '192.168.153.130'
        user = 'root'
        password = 'centos'
    }
}


group = 'com.project'
version = '1.0-SNAPSHOT'

sourceCompatibility = '11'
targetCompatibility = '11'

repositories {
    mavenCentral()
}

dependencies {
    // Servlet API (provided)
    providedCompile 'jakarta.servlet:jakarta.servlet-api:6.0.0'

    // HTTP Components
    implementation 'org.apache.httpcomponents:fluent-hc:4.5.13'

    // JUnit 5
    testImplementation "org.junit.jupiter:junit-jupiter-api:5.9.2"
    testRuntimeOnly "org.junit.jupiter:junit-jupiter-engine:5.9.2"

    // MySQL Connector
    implementation 'mysql:mysql-connector-java:8.0.30'

    // JSTL and Servlet (old javax)
    implementation 'javax.servlet:javax.servlet-api:4.0.1'
    implementation 'javax.servlet:jstl:1.2'

    // JDBI
    implementation 'org.jdbi:jdbi3-core:3.41.3'
    implementation 'org.jdbi:jdbi3-sqlobject:3.30.0'

    // Jakarta Mail
    implementation 'com.sun.mail:jakarta.mail:1.6.3'

    // Jackson (modern)
    implementation "com.fasterxml.jackson.core:jackson-core:2.15.2"
    implementation "com.fasterxml.jackson.core:jackson-databind:2.15.2"
    implementation "com.fasterxml.jackson.core:jackson-annotations:2.15.2"
    implementation 'com.fasterxml.jackson.datatype:jackson-datatype-jsr310:2.13.5'

    // Jackson (ASL)
    implementation "org.codehaus.jackson:jackson-core-asl:1.9.13"
    implementation "org.codehaus.jackson:jackson-mapper-asl:1.9.13"

    // JSON
    implementation 'org.json:json:20200518'

    // Google APIs
    implementation 'com.google.http-client:google-http-client-gson:1.42.3'
    implementation 'com.google.api-client:google-api-client:2.7.2'
    implementation 'com.google.oauth-client:google-oauth-client:1.34.1'

    // Google Authenticator (TOTP)
    implementation 'com.warrenstrange:googleauth:1.5.0'

    // ZXing (QR Code)
    implementation 'com.google.zxing:core:3.5.1'
    implementation 'com.google.zxing:javase:3.5.1'

    // BCrypt
    implementation 'org.mindrot:jbcrypt:0.4'

    // Lombok
    compileOnly 'org.projectlombok:lombok:1.18.36'
    annotationProcessor 'org.projectlombok:lombok:1.18.36'

    // JetBrains annotations
    implementation 'org.jetbrains:annotations:23.0.0'

    // Facebook4j
    implementation 'org.facebook4j:facebook4j-core:2.4.12'
}

test {
    useJUnitPlatform()
}


ssh.settings {
    knownHosts = allowAnyHosts
}
task docker_app_start {
    doLast {
        println 'begin docker_app_start'
        ssh.run {
            session(remotes.host) {
                execute 'sudo docker stop tomcat9', ignoreError: true
                execute 'sudo docker run -it --rm -d ' +
                        '--name tomcat9 ' +
                        '-v /usr/deploy:/usr/local/tomcat/webapps ' +
                        '-p 80:8080 ' +
                        'tomcat:9.0&'
            }
        }
    }
}
task docker_upload_file_to_server {
    doLast {
        println 'begin docker_upload_file_to_server'
        ssh.run {
            session(remotes.host) {
                remove '/usr/deploy/Project_Web-demo.war'
                remove '/usr/deploy/Project_Web-demo/'
                put from: "${project.projectDir}/build/libs/Project_Web-1.0-SNAPSHOT.war",
                        into: "/usr/deploy/Project_Web-demo.war"
            }
        }
    }
}

task docker_deploy {
    docker_deploy.dependsOn docker_app_start
    docker_deploy.dependsOn docker_upload_file_to_server
    docker_deploy.dependsOn build
    tasks.getByName('docker_app_start').mustRunAfter docker_upload_file_to_server
}
tasks.withType(JavaCompile) {
    options.encoding = 'UTF-8'
}
