pipeline {
    agent any

    environment {
        // Kendi altyapımıza ait değişkenleri tanımlıyoruz
        NEXUS_REGISTRY = '192.168.65.128:8082'
        IMAGE_NAME     = 'online-boutique-frontend'
        IMAGE_TAG      = "${BUILD_NUMBER}" // Her build numarası bir etiket (örn: 2, 3, 4)
        SERVICE_DIR    = 'src/frontend'    // Projede frontend kodlarının durduğu klasör
    }

    stages {
        stage('1. Kodu Temizle ve Hazırla') {
            steps {
                echo 'Eski kalıntılar temizleniyor ve süreç başlıyor...'
                // İlk adımda GitHub'dan gelen kodun doğruluğunu teyit ediyoruz
            }
        }

        stage('2. Docker İmajı Derle (Build)') {
            steps {
                echo "Frontend servisi için Docker imajı üretiliyor: ${IMAGE_NAME}:${IMAGE_TAG}"
                script {
                    // src/frontend klasörüne gidip oradaki Dockerfile'ı tetikliyoruz
                    dir("${SERVICE_DIR}") {
                        sh "docker build -t ${NEXUS_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} ."
                    }
                }
            }
        }

        stage('3. Nexus Registry\'e Gönder (Push)') {
            steps {
                echo 'Üretilen Docker imajı yerel Nexus depomuza gönderiliyor...'
                script {
                    // Nexus Docker deponuz kullanıcı adı/şifre istiyorsa burayı güncelleyeceğiz. 
                    // Şimdilik anonim/açık izin verdiğimizi varsayarak doğrudan pushluyoruz:
                    sh "docker push ${NEXUS_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('4. Kubernetes Test Ortamına Deploy Et') {
            steps {
                echo 'K3s Test Cluster\'ı üzerinde canlıya alınıyor...'
                script {
                    // Burası bir sonraki adımda Kubernetes manifestolarını bağlayacağımız yer.
                    echo 'İmaj hazır, Kubernetes dağıtımı tetiklenecek.'
                }
            }
        }
    }

    post {
        success {
            echo 'Tebrikler Özkan Bey! Süreç başarıyla tamamlandı.'
        }
        failure {
            echo 'Eyvah! Süreçte bir hata oluştu. Lütfen logları kontrol edin.'
        }
    }
}
