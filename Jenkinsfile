pipeline {
    agent any
    
    environment {
        CREDENTIALS_ID = '31'
        GITHUB_REPO_TEST = 'git@github.com:fsoymaz/CalculatorTest.git'
        NUGET_FEED = "${WORKSPACE}/nuget_packages"
    }
    
    stages {
        stage('📥 Checkout Calculator Backend') {
            steps {
                echo 'Calculator Backend repository klone ediliyor...'
                echo "Branch: ${BRANCH_NAME ?: 'local'}"
                checkout scm
            }
        }
        
        stage('🔨 Build NuGet Package') {
            steps {
                echo 'CalculatorLib NuGet package oluşturuluyor...'
                sh '''
                    mkdir -p ${NUGET_FEED}
                    cd ${WORKSPACE}/CalculatorLib
                    dotnet pack -c Release -o ${NUGET_FEED}
                    echo "Package oluşturuldu:"
                    ls -la ${NUGET_FEED}
                '''
            }
        }
        
        stage('📦 Clone CalculatorTest') {
            steps {
                echo 'CalculatorTest repository klone ediliyor...'
                sh '''
                    cd ${WORKSPACE}
                    rm -rf CalculatorTest || true
                    git clone ${GITHUB_REPO_TEST} CalculatorTest
                '''
            }
        }
        
        stage('📋 Setup Dependencies') {
            steps {
                echo 'NuGet feed ekleniyor ve dependencies yükleniyor...'
                sh '''
                    cd ${WORKSPACE}/CalculatorTest/CalculatorTests
                    dotnet nuget add source ${NUGET_FEED} --name jenkins-nuget || true
                    dotnet restore
                '''
            }
        }
        
        stage('🧪 Run Unit Tests') {
            steps {
                echo 'Unit testler çalıştırılıyor...'
                sh '''
                    cd ${WORKSPACE}/CalculatorTest/CalculatorTests
                    dotnet test --no-restore --verbosity normal
                '''
            }
        }
        
        stage('✅ Test Decision') {
            steps {
                echo 'Test sonuçları kontrol ediliyor...'
                script {
                    try {
                        sh '''
                            cd ${WORKSPACE}/CalculatorTest/CalculatorTests
                            dotnet test --no-restore
                        '''
                        echo '✅ TÜM TESTLER BAŞARILI!'
                        env.TEST_PASSED = 'true'
                    } catch (Exception e) {
                        echo '❌ TESTLER BAŞARILI DEĞİL!'
                        echo '❌ PUSH ENGELLEND! Test başarısız oldu.'
                        env.TEST_PASSED = 'false'
                        error 'Tests failed - Push will be rejected'
                    }
                }
            }
        }
        
        stage('✅ Build Success - PR Approvable') {
            when {
                allOf {
                    expression { env.TEST_PASSED == 'true' }
                    branch 'PR-*'  // Pull Request branch'lerinde
                }
            }
            steps {
                echo '✅ TÜM TESTLER BAŞARILI!'
                echo '✅ Bu Pull Request merge edilebilir (main\'e)'
                echo '✅ GitHub PR status: APPROVED'
            }
        }
        
        stage('✅ Direct Main Push - Success') {
            when {
                allOf {
                    expression { env.TEST_PASSED == 'true' }
                    branch 'main'  // Doğrudan main'e push
                }
            }
            steps {
                echo '✅ Main branch üzerinde testler başarılı'
                echo '✅ Production deploy hazır'
            }
        }
    }
    
    post {
        success {
            echo '✅ PIPELINE BAŞARILI'
            echo '✅ Calculator repo\'suna push yapabilirsin - Testler geçti'
        }
        failure {
            echo '❌ PIPELINE BAŞARILI DEĞİL'
            echo '❌ PUSH ENGELLENDI - Testler başarısız oldu'
            echo '⚠️ Calculator repo\'suna push yapman engellendi'
        }
    }
}
