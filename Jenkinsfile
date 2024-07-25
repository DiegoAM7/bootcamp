pipeline {
    agent any

	tools {
    	maven 'Maven'
	}
    
    environment {
        // Define environment variables
        REGISTRY = "registry.hub.docker.com" // e.g., Docker Hub or any other registry
	    
		SONAR_HOST_URL = 'https://sonarqube.flexsolution.xyz'
		SONARQUBE_TOKEN = credentials('SonarQube')
    }

    stages {
        stage('Checkout') {
            steps {
                // Clona el repositorio
                git branch: 'main', url: 'https://github.com/DiegoAM7/bootcamp'
            }
        }
	stage('Test Maven') {
		steps {
			sh 'mvn --version'
		}
	}
        stage('Build') {
            steps {
                // Construye el proyecto Maven
                sh 'mvn clean package'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                script {
                    // Realiza el análisis de SonarQube
                    def scannerHome = tool 'sonarqube' // Nombre de la herramienta SonarQube Scanner configurada en Jenkins
                    withSonarQubeEnv('sonarqube') {
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=spring-boot-complete -Dsonar.sources=. -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.token=$SONARQUBE_TOKEN -Dsonar.java.binaries=target/classes"
                    }
                }
            }
        }
        stage('Quality Gate') {
            steps {
                script {
                    // Espera y comprueba el resultado del análisis de SonarQube
                    timeout(time: 1, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
        }
        /*
				stage('Build Docker Image') {
            steps {
                script {
                    // Construye la imagen Docker
                    def image = docker.build("${DOCKER_IMAGE}:${env.BUILD_ID}")
                    // Inicia sesión en el registro Docker
                    docker.withRegistry("https://${env.REGISTRY}", env.DOCKER_CREDENTIALS_ID) {
                        // Empuja la imagen al registro
                        image.push()
                    }
                }
            }
        } */

        /*
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Crea un archivo de despliegue para Kubernetes
                    sh """
                    cat <<EOF > k8s-deployment.yaml
                    apiVersion: apps/v1
                    kind: Deployment
                    metadata:
                      name: my-spring-boot-app
                      labels:
                        app: my-spring-boot-app
                    spec:
                      replicas: 1
                      selector:
                        matchLabels:
                          app: my-spring-boot-app
                      template:
                        metadata:
                          labels:
                            app: my-spring-boot-app
                        spec:
                          containers:
                          - name: my-spring-boot-app-container
                            image: ${REGISTRY}/${DOCKER_IMAGE}:${env.BUILD_ID}
                            ports:
                            - containerPort: 8080
                    ---
                    apiVersion: v1
                    kind: Service
                    metadata:
                      name: my-spring-boot-app-service
                    spec:
                      type: LoadBalancer
                      ports:
                      - port: 80
                        targetPort: 8080
                      selector:
                        app: my-spring-boot-app
                    EOF
                    """

                    // Despliega en Kubernetes
                    //withKubeConfig([credentialsId: env.KUBE_CREDENTIALS_ID]) {
                        sh 'kubectl apply -f k8s-deployment.yaml'
                    //}
                }
            }
        }*/
        stage('Cleanup') {
            steps {
                // Limpieza despues de cada build
                cleanWs()
            }
        }
    }
}
