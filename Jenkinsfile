pipeline {
    agent any
    stages {
        stage('SCM') {
            steps {
                checkout scm
            }
        }

        stage('Download Build Wrapper') {
            steps {
                powershell '''
                  $path = "$HOME/.sonar/build-wrapper-win-x86.zip"
                  rm $HOME/.sonar/build-wrapper-win-x86 -Recurse -Force -ErrorAction SilentlyContinue
                  rm $path -Force -ErrorAction SilentlyContinue
                  New-Item -ItemType directory -Path .sonar -Force
                  [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
                  (New-Object System.Net.WebClient).DownloadFile("<SonarQube URL>/static/cpp/build-wrapper-win-x86.zip", $path) <# Replace with your SonarQube server URL #>
                  Add-Type -AssemblyName System.IO.Compression.FileSystem
                  [System.IO.Compression.ZipFile]::ExtractToDirectory($path, "$HOME/.sonar")
                '''
            }
        }

        stage('Build') {
            steps {
                powershell '''
                  $env:Path += ";$HOME/.sonar/build-wrapper-win-x86"
                  rm build -Recurse -Force -ErrorAction SilentlyContinue <# To ensure a clean build for the analysis #>
                  New-Item -ItemType directory -Path build
                  cmake -S . -B build
                  build-wrapper-win-x86-64.exe --out-dir bw-output cmake --build build/ --config Release
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
