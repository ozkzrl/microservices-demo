pipeline {
    agent any
    
    environment {
        NEXUS_REG    = '192.168.65.128:8082'
        GIT_CRED_ID  = 'github-credentials'
        
        // YENİ: Jenkins'in localhost proxy ağ döngüsüne girmesini engelleyen muafiyetler
        NO_PROXY     = '127.0.0.1,localhost,192.168.65.129'
        no_proxy     = '127.0.0.1,localhost,192.168.65.129'
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
                    dir('release') {
                        // Dosyanın içindeki varsayılan imajı lokal Nexus imaj adresiyle değiştiriyoruz
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

        // YENİ AŞAMA: Güncellenen manifestoyu uzaktaki K8s kümesine otomatik uygula (CD)
        stage('5. Kubernetes Kümesine Otomatik Dağıt (Deploy)') {
            steps {
                script {
                    dir('release') {
                        echo 'Uzaktaki Kubernetes kümesine bağlanılıyor ve dağıtım başlatılıyor...'
                        
                        // env -i ile kabuğu tamamen yalıtarak proxy karmaşasını kesin olarak engelliyoruz
                        // control-plane üzerinde elle oluşturup test ettiğimiz .kube/config dosyasını temel alıyoruz
                        sh """
                            env -i HOME=${HOME} PATH=${PATH} \
                            KUBECONFIG=${HOME}/.kube/config \
                            NO_PROXY=${env.NO_PROXY} \
                            no_proxy=${env.no_proxy} \
                            kubectl apply -f kubernetes-manifests.yaml
                        """
                        
                        echo 'Dağıtım komutu uzaktaki kümeye başarıyla iletildi!'
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo "Tebrikler! Build #${env.BUILD_NUMBER} başarıyla Nexus'a pushlandı, Git reponuz güncellendi ve uzak K8s kümesinde canlıya alındı."
        }
        failure {
            echo 'Eyvah! Süreçte bir hata oluştu. Lütfen logları kontrol edin.'
        }
    }
}
