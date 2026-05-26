pipeline {
    agent any
    
    environment {
        NEXUS_REG      = '192.168.65.128:8082'
        NEXUS_AUTH_USR = 'admin'
        NEXUS_AUTH_PSW = 'Qwer4321/' // Nexus şifreniz
        
        // GitHub'a push yapabilmek için Jenkins'e eklediğiniz Credential ID'si
        GIT_CRED_ID    = 'github-credentials' 
    }
    
    stages {
        stage('1. Kodu Temizle ve Hazırla') {
            steps {
                echo 'Eski kalıntılar temizleniyor...'
            }
        }
        
        stage('2. Docker İmajı Derle (Build)') {
            steps {
                dir('src/frontend') {
                    sh "docker build -t ${NEXUS_REG}/online-boutique-frontend:${env.BUILD_NUMBER} ."
                }
            }
        }
        
        stage("3. Nexus Registry'e Gönder (Push)") {
            steps {
                script {
                    sh "echo '${NEXUS_AUTH_PSW}' | docker login ${NEXUS_REG} -u ${NEXUS_AUTH_USR} --password-stdin"
                    sh "docker push ${NEXUS_REG}/online-boutique-frontend:${env.BUILD_NUMBER}"
                    sh "docker logout ${NEXUS_REG}"
                }
            }
        }
        
       stage('4. Kubernetes Manifestosunu Güncelle (GitOps)') {
    steps {
        script {
            // Teşhis adımından sonra dosya hangi klasördeyse buraya yazıyoruz.
            // Örnek olarak 'release' klasöründe olduğunu varsayalım:
            dir('release') { 
                
                sh """
                    sed -i 's|image: gcr.io/google-samples/microservices-demo/frontend:.*|image: ${NEXUS_REG}/online-boutique-frontend:${env.BUILD_NUMBER}|g' kubernetes-manifests.yaml
                """
                
                withCredentials([usernamePassword(credentialsId: "${env.GIT_CRED_ID}", usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                    sh """
                        git config user.email "jenkins@local.com"
                        git config user.name "Jenkins CI"
                        git add kubernetes-manifests.yaml
                        git commit -m "Automated CD: Frontend image updated to version ${env.BUILD_NUMBER} [skip ci]" || echo "Değişiklik yok"
                        git push https://${GIT_USER}:${GIT_TOKEN}@github.com/ozkzrl/microservices-demo.git HEAD:main
                    """
                }
            }
        }
    }
}
    
    post {
        failure {
            echo 'Eyvah! Süreçte bir hata oluştu. Lütfen logları kontrol edin.'
        }
    }
}
