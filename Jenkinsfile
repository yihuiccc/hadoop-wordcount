pipeline {
    agent any
    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/yihuiccc/hadoop-wordcount.git'
            }
        }
        stage('Static Code Analysis') {
            steps {
                script {
                    sonarQubeScanner {
                        options: [ '-Dsonar.projectKey=HadoopWordCountKey' ]
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
                    --cluster=your-cluster-name \
                    --region=your-region \
                    --jar file:///usr/lib/hadoop-mapreduce/hadoop-streaming.jar \
                    -- -files mapper.py,reducer.py \
                    -mapper "python3 mapper.py" \
                    -reducer "python3 reducer.py" \
                    -input gs://your-bucket/input.txt \
                    -output gs://your-bucket/output/
                """
            }
        }
    }
}
