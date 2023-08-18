pipeline {
  agent any
  parameters {
    choice (name: 'chooseNode', choices: ['Green', 'Blue'], description: 'Choose which Environment to Deploy: ')
  }
  environment {
    listenerARN = 'arn:aws:elasticloadbalancing:us-east-1:027411030380:listener/app/myALB/f30442dbf18c6e52/12ed39e484465afd'
    blueARN = 'arn:aws:elasticloadbalancing:us-east-1:027411030380:targetgroup/blue/7d9b18d4bac64dbc'
    greenARN = 'arn:aws:elasticloadbalancing:us-east-1:027411030380:targetgroup/green/1edfd9ae854ec245'
  }
  stages {
    stage('Deployment Started') {
      parallel {
        stage('Green') {
          when {
            expression {
              params.chooseNode == 'Green'
            }
          }
          stages {
            stage('Offloading Green') {
              steps {
                sh """aws elbv2 modify-listener --listener-arn ${listenerARN} --default-actions '[{"Type": "forward","Order": 1,"ForwardConfig": {"TargetGroups": [{"TargetGroupArn": "${greenARN}", "Weight": 0 },{"TargetGroupArn": "${blueARN}", "Weight": 1 }],"TargetGroupStickinessConfig": {"Enabled": true,"DurationSeconds": 1}}}]'"""
              }
            }
            stage('Deploying to Green') {
              steps {
                sh '''scp -r index.html ec2-user@3.6.126.50:/usr/share/nginx/html/
                ssh -t ec2-user@3.6.126.50 -p 22 << EOF 
                sudo service nginx restart
                '''
              }
            }
            stage('Validate and Add Green for testing') {
              steps {
                sh """
                if [ "\$(curl -o /dev/null --silent --head --write-out '%{http_code}' http://3.6.126.50/)" -eq 200 ]
                then
                    echo "** BUILD IS SUCCESSFUL **"
                    curl -I http://3.6.126.50/
                    aws elbv2 modify-listener --listener-arn ${listenerARN} --default-actions '[{"Type": "forward","Order": 1,"ForwardConfig": {"TargetGroups": [{"TargetGroupArn": "${greenARN}", "Weight": 0 },{"TargetGroupArn": "${blueARN}", "Weight": 1 }],"TargetGroupStickinessConfig": {"Enabled": true,"DurationSeconds": 1}}}]'
                else
	                echo "** BUILD IS FAILED ** Health check returned non 200 status code"
                    curl -I http://3.6.126.50/
                exit 2
                fi
                """
              }
            }
          }
        }
        stage('Blue') {
          when {
            expression {
              params.chooseNode == 'Blue'
            }
          }
          stages {
            stage('Offloading Blue') {
              steps {
                sh """aws elbv2 modify-listener --listener-arn ${listenerARN} --default-actions '[{"Type": "forward","Order": 1,"ForwardConfig": {"TargetGroups": [{"TargetGroupArn": "${greenARN}", "Weight": 1 },{"TargetGroupArn": "${blueARN}", "Weight": 0 }],"TargetGroupStickinessConfig": {"Enabled": true,"DurationSeconds": 1}}}]'"""
              }
            }
            stage('Deploying to Blue') {
              steps {
                sh '''scp -r index.html ec2-user@3.110.209.118:/usr/share/nginx/html/
                ssh -t ec2-user@3.110.209.118 -p 22 << EOF 
                sudo service nginx restart
                '''
              }
            }
            stage('Validate Blue and added to TG') {
              steps {
                sh """
                if [ "\$(curl -o /dev/null --silent --head --write-out '%{http_code}' http://3.110.209.118/)" -eq 200 ]
                then
                    echo "** BUILD IS SUCCESSFUL **"
                    curl -I http://3.110.209.118/
                    aws elbv2 modify-listener --listener-arn ${listenerARN} --default-actions '[{"Type": "forward","Order": 1,"ForwardConfig": {"TargetGroups": [{"TargetGroupArn": "${greenARN}", "Weight": 1 },{"TargetGroupArn": "${blueARN}", "Weight": 1 }],"TargetGroupStickinessConfig": {"Enabled": true,"DurationSeconds": 1}}}]'
                else
	                echo "** BUILD IS FAILED ** Health check returned non 200 status code"
                    curl -I http://3.110.209.118/
                exit 2
                fi
                """
              }
            }
          }
        }
      }
    }
  }
}
