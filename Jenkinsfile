pipeline {
  agent any

  parameters {
    choice(name: 'ACTION', choices: ['plan','apply','destroy'], description: 'Terraform action')
  }

  environment {
    TF_IN_AUTOMATION = "true"
    AWS_DEFAULT_REGION = "ap-south-1"
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Terraform Init') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'aws-keys',
          usernameVariable: 'AWS_ACCESS_KEY_ID',
          passwordVariable: 'AWS_SECRET_ACCESS_KEY'
        )]) {
          sh '''
            echo "AWS_ACCESS_KEY_ID set? ${AWS_ACCESS_KEY_ID:+YES}"
            echo "AWS_SECRET_ACCESS_KEY set? ${AWS_SECRET_ACCESS_KEY:+YES}"
            terraform init
          '''
        }
      }
    }

    stage('Terraform Validate') {
      steps { sh 'terraform validate' }
    }

    stage('Terraform Plan') {
      when { expression { params.ACTION == 'plan' || params.ACTION == 'apply' } }
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'aws-keys',
          usernameVariable: 'AWS_ACCESS_KEY_ID',
          passwordVariable: 'AWS_SECRET_ACCESS_KEY'
        )]) {
          sh 'terraform plan -out=tfplan'
        }
      }
    }

    stage('Terraform Apply') {
      when { expression { params.ACTION == 'apply' } }
      steps {
        input message: "Approve APPLY to AWS?"
        withCredentials([usernamePassword(
          credentialsId: 'aws-keys',
          usernameVariable: 'AWS_ACCESS_KEY_ID',
          passwordVariable: 'AWS_SECRET_ACCESS_KEY'
        )]) {
          sh 'terraform apply -auto-approve tfplan'
        }
      }
    }

    stage('Terraform Destroy') {
      when { expression { params.ACTION == 'destroy' } }
      steps {
        input message: "Approve DESTROY on AWS?"
        withCredentials([usernamePassword(
          credentialsId: 'aws-keys',
          usernameVariable: 'AWS_ACCESS_KEY_ID',
          passwordVariable: 'AWS_SECRET_ACCESS_KEY'
        )]) {
          sh 'terraform destroy -auto-approve'
        }
      }
    }
  }
}