pipeline { 
    agent any

    tools {
        maven 'maven3'   // Manage Jenkins -> Global Tool Configuration lo maven3 name undali
        jdk 'jdk17'      // Manage Jenkins -> Global Tool Configuration lo jdk17 undali
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'   // Jenkins lo Sonar Scanner tool id
        SONAR_HOST_URL = 'http://13.221.38.103:9000'
        DOCKER_REPO = 'sairammuchintala/bloggingapp'
    }

    stages {

        stage('Clean Workspace') {
            steps {
                echo 'Cleaning workspace...'
                deleteDir()
            }
        }

        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Sairam9849/FullStack-Blogging-App.git'
            }
        }

        stage('Compile') {
            steps {
                withMaven(maven: 'maven3', jdk: 'jdk17') {
                    sh "mvn clean compile"
                }
            }
        }

        stage('Test') {
            steps {
                withMaven(maven: 'maven3', jdk: 'jdk17') {
                    sh "mvn test"
                }
            }
        }

        // Trivy stage intentionally skipped for now to avoid snap + /var/lib/jenkins issue
        stage('Trivy FS Scan') {
            steps {
                echo 'Skipping Trivy FS Scan (Trivy snap cannot run under /var/lib/jenkins).'
                echo 'Later, install Trivy via apt and re-enable this stage.'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    sh """
                        ${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=fullstack-blog \
                        -Dsonar.sources=. \
                        -Dsonar.java.binaries=target \
                        -Dsonar.host.url=${SONAR_HOST_URL} \
                        -Dsonar.login=${SONAR_TOKEN}
                    """
                }
            }
        }

        stage('Package') {
            steps {
                withMaven(maven: 'maven3', jdk: 'jdk17') {
                    sh "mvn clean package"
                }
            }
        }

        stage('Publish Artifacts') {
            steps {
                withMaven(maven: 'maven3', jdk: 'jdk17') {
                    script {
                        def version = sh(
                            script: "mvn help:evaluate -Dexpression=project.version -q -DforceStdout",
                            returnStdout: true
                        ).trim()

                        def repoId = version.endsWith('-SNAPSHOT') ? 'maven-snapshots' : 'maven-releases'
                        def repoUrl = version.endsWith('-SNAPSHOT')
                            ? 'http://3.87.19.206:8081/repository/maven-snapshots/'
                            : 'http://3.87.19.206:8081/repository/maven-releases/'

                        echo "Deploying ${version} to ${repoId} at ${repoUrl}"

                        sh "mvn deploy -DaltDeploymentRepository=${repoId}::default::${repoUrl}"
                    }
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        def version = sh(
                            script: "mvn help:evaluate -Dexpression=project.version -q -DforceStdout",
                            returnStdout: true
                        ).trim()

                        sh "docker login -u ${DOCKER_USER} -p ${DOCKER_PASS}"
                        sh "docker build -t ${DOCKER_REPO}:${version} -t ${DOCKER_REPO}:latest ."
                        sh "docker push ${DOCKER_REPO}:${version}"
                        sh "docker push ${DOCKER_REPO}:latest"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline finished successfully."
        }
        failure {
            echo "Pipeline failed. Check logs for details."
        }
    }
}
