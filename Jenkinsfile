pipeline {
    agent any

    stages {

        stage('Docker build'){
            stage ('JenkinsDemo'){
                    steps {
                        dir('JenkinsDemo'){
                            sh 'npm install'
                            sh 'docker build -t ravindrasargar/jenkinsdemo .'
                        }
                    }
                }
        }

        stage('Docker push'){
            steps{
                script{
                    docker.withRegistry('https://003656774475.dkr.ecr.us-east-1.amazonaws.com', 'ecr:us-east-1:ravindra-aws') {
                        docker.image('jenkins/demo').push("$currentBuild.number")
                    }
                }
            }
        }

        stage('Deploy in ECS') {
              steps {
                  withCredentials([string(credentialsId: 'AWS_EXECUTION_ROL_SECRET', variable: 'AWS_ECS_EXECUTION_ROL'),string(credentialsId: 'AWS_REPOSITORY_URL_SECRET', variable: 'AWS_ECR_URL')]) {
                      script {
                          updateContainerDefinitionJsonWithImageVersion()
                          sh("/usr/local/bin/aws ecs register-task-definition --region ${AWS_ECR_REGION} --family ${AWS_ECS_TASK_DEFINITION} --execution-role-arn ${AWS_ECS_EXECUTION_ROL} --requires-compatibilities ${AWS_ECS_COMPATIBILITY} --network-mode ${AWS_ECS_NETWORK_MODE} --cpu ${AWS_ECS_CPU} --memory ${AWS_ECS_MEMORY} --container-definitions file://${AWS_ECS_TASK_DEFINITION_PATH}")
                          def taskRevision = sh(script: "/usr/local/bin/aws ecs describe-task-definition --task-definition ${AWS_ECS_TASK_DEFINITION} | egrep \"revision\" | tr \"/\" \" \" | awk '{print \$2}' | sed 's/\"\$//'", returnStdout: true)
                          sh("/usr/local/bin/aws ecs update-service --cluster ${AWS_ECS_CLUSTER} --service ${AWS_ECS_SERVICE} --task-definition ${AWS_ECS_TASK_DEFINITION}:${taskRevision}")
                          }
                    }
            }
        }
    }
}
