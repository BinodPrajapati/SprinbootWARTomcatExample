pipeline {
    agent any

    parameters {
        string(name: 'SPRING_PROFILE', defaultValue: 'dev', description: 'Spring profile to use for the build (e.g., dev, qa)')
    }

    environment {
        DEV_SERVER = "http://dev-server:8080/manager/text"
        QA_SERVER = "http://qa-server:8080/manager/text"
        TOMCAT_USER = credentials('TOMCAT_USER') // Use Jenkins credentials store for Tomcat credentials
        JAVA_HOME = '/path/to/java' // Update with the correct Java installation path
        MAVEN_HOME = '/path/to/maven' // Update with the correct Maven installation path
        PATH = "${JAVA_HOME}/bin:${MAVEN_HOME}/bin:${env.PATH}" // Add Java and Maven to the PATH
        SONAR_HOST_URL = 'http://sonarqube-server:9000' // SonarQube server URL
        SONAR_LOGIN = credentials('SONAR_TOKEN') // Use Jenkins credentials for SonarQube token
    }

    stages {
        stage('Build') {
            steps {
                script {
                    echo "Building Spring Boot WAR file with profile: ${params.SPRING_PROFILE}"
                    sh "${MAVEN_HOME}/bin/mvn clean package -Dspring.profiles.active=${params.SPRING_PROFILE} -DskipTests"
                }
            }
        }

        stage('Run Test Cases') {
            steps {
                script {
                    echo "Running Test Cases..."
                    sh "${MAVEN_HOME}/bin/mvn test"
                }
            }
        }

        stage('SonarQube Code Quality Check') {
            steps {
                script {
                    echo "Performing SonarQube Code Quality Check..."
                    sh """
                        ${MAVEN_HOME}/bin/mvn sonar:sonar \
                        -Dsonar.host.url=${SONAR_HOST_URL} \
                        -Dsonar.login=${SONAR_LOGIN}
                    """
                }
            }
        }

        stage('Deploy to Dev Server') {
            steps {
                echo "Deploying to DEV Server..."
                deployToTomcat(DEV_SERVER, 'target/demo.war')
            }
        }

        stage('QA Approval') {
            when {
                expression {
                    currentBuild.result == null // Proceed only if the previous stages were successful
                }
            }
            steps {
                script {
                    input(message: 'Deploy to QA?')
                }
            }
        }

        stage('Deploy to QA Server') {
            when {
                expression {
                    currentBuild.result == null // Ensure QA deployment runs only if previous stages succeeded
                }
            }
            steps {
                echo "Deploying to QA Server..."
                deployToTomcat(QA_SERVER, 'target/demo.war')
            }
        }
    }

    post {
        success {
            echo "Deployment completed successfully!"
        }
        failure {
            echo "Deployment failed!"
        }
    }
}

// Helper function to deploy to Tomcat
def deployToTomcat(tomcatUrl, warFile) {
    script {
        try {
            def curlCommand = """
                curl -u ${env.TOMCAT_USER} \
                --upload-file ${warFile} \
                ${tomcatUrl}/deploy?path=/demo&update=true
            """
            sh curlCommand
            echo "Deployment to ${tomcatUrl} succeeded!"
        } catch (Exception e) {
            error "Deployment to ${tomcatUrl} failed: ${e.getMessage()}"
        }
    }
}
