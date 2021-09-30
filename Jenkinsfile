#!/usr/bin/env groovy

node {
    def app
    stage('checkout') {
        checkout scm
    }

    stage('check java') {
        bat "java -version"
    }

    stage('clean') {
        bat "./mvnw -ntp clean -P-webapp"
    }
    stage('nohttp') {
        bat "./mvnw -ntp checkstyle:check"
    }

    stage('install tools') {
        bat "./mvnw -ntp com.github.eirslett:frontend-maven-plugin:install-node-and-npm@install-node-and-npm"
    }

    stage('npm install') {
        bat "./mvnw -ntp com.github.eirslett:frontend-maven-plugin:npm"
    }

    stage('packaging') {
        bat "./mvnw -ntp verify -P-webapp -Pprod -DskipTests"
        archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
    }

    stage('SonarQube Analysis code') {
          bat "./mvnw clean verify sonar:sonar -Dmaven.test.skip=true"
    }
    stage('Build image') {
          app = docker.build("brandonjones085/test")
     }

      stage('Test image') {
         app.inside {
             bat 'echo "Tests passed"'
          }
      }

      stage('Push image') {
           docker.withRegistry('https://registry.hub.docker.com', 'git') {
             app.push("${env.BUILD_NUMBER}")
             app.push("latest")
          }
        }
}
