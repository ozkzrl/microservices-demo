pipeline {
    agent any
    
    environment {
        // Jenkins'teki kimlik bilgisini pipeline genelinde değişkenlere atıyoruz
        NEXUS_REG = '192.168.65.128:8082'
        NEXUS_AUTH = credentials('nexus-credentials') 
        // Bu tanım otomatik olarak iki alt değişken üretir: NEXUS_AUTH_USR ve NEXUS_AUTH_PSW
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
                echo 'Üretilen Docker imajı yerel Nexus depomuza gönderiliyor...'
                script {
                    // Otomatik üretilen USR ve PSW değişkenlerini doğrudan sh içinde kullanıyoruz
                    sh "echo '${NEXUS_AUTH_PSW}' | docker login ${NEXUS_REG} -u ${NEXUS_AUTH_USR} --password-stdin"
                    sh "docker push ${NEXUS_REG}/online-boutique-frontend:${env.BUILD_NUMBER}"
                    sh "docker logout ${NEXUS_REG}"
                }
            }
        }
        
        stage('4. Kubernetes Test Ortamına Deploy Et') {
            steps {
                echo 'Kubernetes ortamına deploy ediliyor...'
                // Deployment adımları buraya gelecek
            }
        }
    }
    
    post {
        failure {
            echo 'Eyvah! Süreçte bir hata oluştu. Lütfen logları kontrol edin.'
        }
    }
}
