pipeline {
    agent any
    
    environment {
        NEXUS_REG    = '192.168.65.128:8082'
        GIT_CRED_ID  = 'github-credentials'
        
        // Jenkins'in localhost proxy ağ döngüsüne girmesini engelleyen muafiyetler
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

        stage('5. Kubernetes Kümesine Otomatik Dağıt (Deploy)') {
            steps {
                script {
                    dir('release') {
                        echo 'Uzaktaki Kubernetes kümesine bağlanılıyor ve dağıtım başlatılıyor...'
                        
                        sh """
                            # 1. Eğer ortamda kubectl yoksa, resmi kaynaktan hemen indir ve yetkilendir
                            if ! command -v kubectl &> /dev/null; then
                                echo "kubectl bulunamadı, çalışma alanına dinamik olarak indiriliyor..."
                                curl -LO "https://dl.k8s.io/release/\$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                                chmod +x ./kubectl
                                KUBECTL_CMD="./kubectl"
                            else
                                KUBECTL_CMD="kubectl"
                            fi

                            # 2. Tamamen yalıtılmış temiz kabukta komutu uzak kümeye gönder
                            env -i HOME=${HOME} PATH=${PATH}:. \
                            KUBECONFIG=/var/jenkins_home/.kube/config \
                            NO_PROXY=${env.NO_PROXY} \
                            no_proxy=${env.no_proxy} \
                            \$KUBECTL_CMD apply -f kubernetes-manifests.yaml
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
