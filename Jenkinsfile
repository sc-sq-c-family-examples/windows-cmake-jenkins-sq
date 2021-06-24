pipeline {
    agent any
    options {
        // This is required if you want to clean before build with the "Workspace Cleanup Plugin"
        skipDefaultCheckout(true)
    }
    stages {
        stage('SCM') {
            steps {
                // Clean before build using the "Workspace Cleanup Plugin"
                cleanWs()
                checkout scm
            }
        }

        stage('Download Build Wrapper') {
            steps {
                powershell '''
                  $path = "$HOME/.sonar/build-wrapper-win-x86.zip"
                  New-Item -ItemType directory -Path .sonar -Force
                  [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
                  (New-Object System.Net.WebClient).DownloadFile("<SonarQube URL>/static/cpp/build-wrapper-win-x86.zip", $path) <# Replace with your SonarQube server URL #>
                  Add-Type -AssemblyName System.IO.Compression.FileSystem
                  [System.IO.Compression.ZipFile]::ExtractToDirectory($path, "$HOME/.sonar")
                  $env:Path += ";$HOME/.sonar/build-wrapper-win-x86"
                '''
            }
        }

        stage('Build') {
            steps {
                powershell '''                  
                  New-Item -ItemType directory -Path build
                  cmake -S . -B build
                  build-wrapper-win-x86-64.exe --out-dir bw-output cmake --build build/ --config Release <# The build is clean thanks to the "cleanWs()" step #>
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'; // Name of the SonarQube Scanner you created in "Global Tool Configuration" section
                    withSonarQubeEnv() {
                        powershell "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }
    }
}
