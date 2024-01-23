pipeline {
    agent {
        kubernetes {
            defaultContainer 'aws'
            yaml '''
            apiVersion: v1
            kind: Pod
            metadata:
              labels:
                app: nodejs-app
            spec:
              containers:
                - name: nodejs
                  image: node:14.17.0
                  command:
                    - cat
                  tty: true
              '''
        }
    }

    stages {
        stage('AWS') {
            agent {
                kubernetes {
                    label 'jenkinsrun'
                    defaultContainer 'aws'
                    yaml '''
kind: Pod
metadata:
  name: kaniko
spec:
  containers:
  - name: aws
    image: amazon/aws-cli:latest
    imagePullPolicy: Always
    command:
    - /usr/bin/sh
    tty: true
'''
                }
            }
            steps {
                withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: 'aws-cloudbees-architecture',
                        accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                        secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                    ]]) {
                    echo 'hello AWS'
                    sh 'aws sts get-caller-identity'
                    
                } //container
              } //steps
        }
        stage('Publish') {
            agent {
                kubernetes {
                    label 'jenkinsrun'
                    defaultContainer 'builder'
                    yaml '''
kind: Pod
metadata:
  name: kaniko
spec:
  containers:
  - name: builder
    image: gcr.io/kaniko-project/executor:debug
    imagePullPolicy: Always
    command:
    - /busybox/cat
    tty: true
'''
                }
            }
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/swashbuck1r/insurance-frontend.git']])
                withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: 'aws-cloudbees-architecture',
                        accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                        secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                    ]]) {
                    echo "hello kaniko"
                    sh  """cat << EOF > '/kaniko/.docker/config.json'
{
    "credsStore": "ecr-login"
}
EOF
"""
                    sh "/kaniko/executor --dockerfile `pwd`/Dockerfile --context `pwd` --destination=189768267137.dkr.ecr.us-east-1.amazonaws.com/insurance-frontend:${env.BUILD_ID}"
                    
                } //container
              } //steps
        }
    }
}
