pipeline {
    agent any
    
    environment {
        DB_HOST = 'db-test.c1aoqqaugpm7.eu-north-1.rds.amazonaws.com'
        DB_NAME = 'db-test'
        AWS_BUCKET = 'test-jnk'
        GZ_FILE_NAME = "backup_${new Date().format('yyyyMMdd_HHmmss')}.sql.gz"
    }
    
    stages {
        stage('MySQL Backup and Gzip') {
            steps {
                script {
                    // Create a backup of the MySQL database and gzip it
                    withCredentials([usernamePassword(credentialsId: 'rds', usernameVariable: 'DB_USER', passwordVariable: 'DB_PASSWORD')]) {
                        sh "mysqldump -h ${env.DB_HOST} -u ${env.DB_USER} -p${env.DB_PASSWORD} ${env.DB_NAME} | gzip > ${env.GZ_FILE_NAME}"
                    }
                }
            }
        }
        
        stage('Upload to S3') {
            steps {
                withCredentials([
                    [
                        $class: 'AmazonWebServicesCredentialsBinding',
                        accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                        secretKeyVariable: 'AWS_SECRET_ACCESS_KEY',
                        credentialsId: 'test'
                    ]
                ]) {
                    // Upload the gzip file to AWS S3
                    sh "aws s3 cp ${env.GZ_FILE_NAME} s3://${env.AWS_BUCKET}/"
                }
            }
        }
        
        stage('Delete Local Backup') {
            steps {
                // Delete the local gzip backup file
                sh "rm ${env.GZ_FILE_NAME}"
            }
        }
    }
}
