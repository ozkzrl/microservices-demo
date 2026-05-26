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
                    // Dosyanın bulunduğu 'release' klasörünün içine giriyoruz
                    dir('release') {
                        // Dosyanın içindeki varsayılan google imaj adresini kendi lokal Nexus imaj adresimiz ve build numaramızla değiştiriyoruz
                        sh """
                            sed -i 's|image: gcr.io/google-samples/microservices-demo/frontend:.*|image: ${NEXUS_REG}/online-boutique-frontend:${env.BUILD_NUMBER}|g' kubernetes-manifests.yaml
                        """
                        
                        // Güncellenen dosyayı GitHub repomuza geri pushluyoruz
                        withCredentials([usernamePassword(credentialsId: "${env.GIT_CRED_ID}", usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                            sh """
                                git config user.email "jenkins@local.com"
                                git config user.name "Jenkins CI"
                                git add kubernetes-manifests.yaml
                                git commit -m "Automated CD: Frontend image updated to version ${env.BUILD_NUMBER} [skip ci]" || echo "Degisiklik yok"
                                git push https://${GIT_USER}:${GIT_TOKEN}@github.com/ozkzrl/microservices-demo.git HEAD:main
                            """
                        }
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
