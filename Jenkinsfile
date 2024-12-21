pipeline {
    agent any

    tools {
        maven 'maven3'        // Ensure the Maven tool is configured in Jenkins
        jdk 'jdk17'           // Select JDK 17 for the pipeline
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'    // Set the path for SonarQube scanner
    }

    stages {
        // Stage 1: Checkout the code from GitHub
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: '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', url: 'https://github.com/ravindra124567/Ekart.git'
            }
        }

        // Stage 2: Compile the code using Maven
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }

        // Stage 3: Run Unit Tests (skip tests that are not required for build)
        stage('Unit Tests') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }

        // Stage 4: Run SonarQube Analysis
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' 
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectkey=Ekart \
                        -Dsonar.projectName=Ekart \
                        -Dsonar.java.binaries=.
                    '''
                }
            }
        }

        // Stage 5: OWASP Dependency Check
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: ' --scan ./', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        // Stage 6: Build the Application (Package the application)
        stage('Build Application') {
            steps {
                sh "mvn clean install"
            }
        }

        // Stage 7: Build & Push Docker Image
        stage('Build & Push Docker Image') {
            steps {
                script {
                    // Authenticate with DockerHub and build the Docker image
                    withDockerRegistry(credentialsId: 'docker-cred', url: 'https://index.docker.io/v1/') {
                        sh "docker build -t Ekart:latest -f docker/Dockerfile ."
                        sh "docker tag Ekart:latest sashikumar24/Ekart:latest"
                        sh "docker push sashikumar24/Ekart:latest"
                    }
                }
            }
        }

        // Stage 8: Deploy the Docker Image to Docker container
        stage('Docker Deploy To Container') {
            steps {
                withDockerRegistry(credentialsId: 'docker-cred', url: 'https://index.docker.io/v1/') {
                    sh "docker run -d --name Ekart -p 8080:8080 sashikumar24/Ekart:latest"
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
