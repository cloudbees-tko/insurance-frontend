pipeline {
    agent {
        kubernetes {
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
                '''.stripIndent()
        }
    }

    stages {
        stage('Publish') {
            steps {
                checkout scm
                withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: 'aws-cloudbees-architecture',
                        accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                        secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                    ]]) {
                    sh  """echo '{"credsStore": "ecr-login"}' >> '/kaniko/.docker/config.json'"""
                    sh "/kaniko/executor --dockerfile `pwd`/Dockerfile --context `pwd` --destination=189768267137.dkr.ecr.us-east-1.amazonaws.com/insurance-frontend-image:${env.BUILD_ID}"
                } //container
              } //steps 
        }
    }
}
