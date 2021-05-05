pipeline {
    parameters {
    choice(name: 'action', choices: 'create\ndestroy', description: 'Create/update or destroy the eks cluster.')
	string(name: 'cluster', defaultValue : 'demo', description: "EKS cluster name;eg demo creates cluster named eks-demo.")
	choice(name: 'ENV', choices: 'STAGE\nQA\nPROD', description: 'Create/update or destroy the eks cluster.')
  }
  
  agent any

  stages {
    stage('checkout') {
        steps {
            git 'https://github.com/sairaghavadev/eks_test.git'
        }
    }
 
    stage('Version') {
        steps {
             sh 'terraform -version'
        }
    }
    stage('Setup') {
      steps {
        script {
          plan = params.cluster + '.plan'
        }
      }
    }
    
    stage('TF Plan') {
		steps {
			withAWS(credentials: 'test', region: 'us-east-1') {
			   sh """
					sudo terraform init
					sudo terraform plan \
						-var cluster-name=${params.cluster} \
						-out ${plan}
					echo ${params.cluster}
				"""
				}
			}
        }
        
     stage('TF Apply') {
      when {
        expression { params.action == 'create' }
      }
      steps {
        script {
          input "Create/update Terraform stack eks-${params.cluster} in aws?" 

          withAWS(credentials: 'test', region: 'us-east-1') {
            sh """
             if [ -f '$HOME/.kube']
             then
                 echo ".kube is present"
             else
                 sudo mkdir -p $HOME/.kube
             fi
              sudo terraform apply -input=false -auto-approve ${plan}
            """
          }
        }
        }
     }
    stage('TF destroy') {
        when {
        expression { params.action == 'destroy' }
		
      }
		steps {
			withAWS(credentials: 'test', region: 'us-east-1') {
			  sh """
					sudo terraform destroy -auto-approve
			   	"""
				}
			}
        }
}
}
