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
                '''.removeIndent()
        }
    }

    stages {
        stage('Publish') {
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
