pipeline {
    agent any
    
    environment {
        NEXUS_REG   = '192.168.65.128:8082'
        GIT_CRED_ID = 'github-credentials' 
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
                    withCredentials([usernamePassword(credentialsId: 'nexus-credentials', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                        sh "echo '${NEXUS_PASS}' | docker login ${NEXUS_REG} -u ${NEXUS_USER} --password-stdin"
                        sh "docker push ${NEXUS_REG}/online-boutique-frontend:${env.BUILD_NUMBER}"
                        sh "docker logout ${NEXUS_REG}"
                    }
                }
            }
        }
        
        stage('4. Kubernetes Manifestosunu Güncelle (GitOps)') {
            steps {
                script {
                    echo "--- Jenkins Çalışma Alanındaki Klasör Yapısı ---"
                    sh "ls -la" 
                    
                    echo "--- Dosyanın Tam Konumunu Arıyoruz ---"
                    sh "find . -name '*manifests*.yaml' -o -name '*manifests*.yml'"
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
