pipeline {
    agent any
    
    environment {
        NEXUS_REG      = '192.168.65.128:8082'
        NEXUS_AUTH_USR = 'admin'
        NEXUS_AUTH_PSW = 'Qwer4321/' // Önceki adımda çalışan şifreniz
        
        // Jenkins Credentials içerisine yükleyeceğiniz Kubeconfig ID'si
        KUBE_CONFIG    = credentials('kubernetes-config')
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
        
        stage('4. Kubernetes Test Ortamına Deploy Et') {
            steps {
                script {
                    // Üretilen dinamik imaj adını tam yol olarak tanımlıyoruz
                    def total_image_path = "${NEXUS_REG}/online-boutique-frontend:${env.BUILD_NUMBER}"
                    
                    // Kubernetes deployment güncelleme komutu ($KUBE_CONFIG Jenkins tarafından otomatik çözülür)
                    sh """
                        kubectl set image deployment/frontend frontend=${total_image_path} \
                        --namespace=test \
                        --kubeconfig=${KUBE_CONFIG}
                    """
                    
                    // Deployment'ın başarıyla tamamlanmasını (Rollout) takip ediyoruz
                    sh """
                        kubectl rollout status deployment/frontend \
                        --namespace=test \
                        --kubeconfig=${KUBE_CONFIG}
                    """
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
