pipeline {
agent any

```
stages {
    stage('Checkout') {
        steps {
            checkout scm
        }
    }

    stage('Check Tools') {
        steps {
            sh 'java -version || true'
            sh 'mvn -v || true'
        }
    }

    stage('Compile') {
        steps {
            sh "mvn compile"
        }
    }
    
    stage('Test') {
        steps {
            sh "mvn test"
        }
    }
    
    stage('Package') {
        steps {
            sh "mvn package"
        }
    }
}

post {
    always {
        echo "Pipeline finished"
    }
}
```

}
