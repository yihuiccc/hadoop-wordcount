pipeline {
    agent {
        kubernetes {
            label 'gcloud-agent'
            defaultContainer 'gcloud'
            yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    jenkins-agent: gcloud-agent
spec:
  containers:
  - name: gcloud
    image: google/cloud-sdk:latest
    command:
    - cat
    tty: true
"""
        }
    }
    stages {
        stage('Checkout Code') {
            steps {
                // Checkout the repository containing mapper.py and reducer.py
                checkout scm
            }
        }
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/yihuiccc/hadoop-wordcount.git'
            }
        }
        stage('Static Code Analysis') {
            steps {
                script {
                    // Use SonarScanner tool configured in Jenkins
                    def scannerHome = tool 'sonarqube-scanner'
                    // Set up the SonarQube environment for the analysis
                    withSonarQubeEnv() {
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=HadoopWordCountKey"
                    }
                }
            }
        }
        stage('Submit Job to Dataproc') {
            when {
                expression {
                    return currentBuild.resultIsBetterOrEqualTo('SUCCESS')
                }
            }
            steps {
                sh """
                gcloud dataproc jobs submit hadoop \
                    --cluster=hadoop-cluster \
                    --region=us-central1 \
                    --jar file:///usr/lib/hadoop-mapreduce/hadoop-streaming.jar \
                    -- -files  mapper.py, reducer.py \
                    -mapper "python3 mapper.py" \
                    -reducer "python3 reducer.py" \
                    -input gs://dataproc-staging-us-central1-154464686072-1fevtjdd/input.txt \
                    -output gs://dataproc-staging-us-central1-154464686072-1fevtjdd/wordcount-output/
                """
            }
        }
    }
}
