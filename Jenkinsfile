pipeline {
	agent {
		node {
			label 'windows' 
        }
    }
    tools { 
        maven 'maven-3.5.4' 
        jdk 'jdk-10' 
    }
    stages {
        stage ('Initialize') {
            steps {
                sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                '''
            }
        }
        stage('Checkout') {
            steps {
				checkout([
					$class: 'GitSCM', 
    				branches: [[name: '*/master']], 
    				doGenerateSubmoduleConfigurations: false, 
   					extensions: [[$class: 'CleanCheckout']], 
    				submoduleCfg: [], 
	    			userRemoteConfigs: [[url: 'https://github.com/ashwani-opstree/project-aspnet-dotnet-webapp.git']]
           		])
            }
        }
        stage('SonarQube analysis') {
			steps {
                script {
             		sqScannerMsBuildHome = tool 'sonarscanner-msbuild-4.3';
        		}
    			withSonarQubeEnv('sonarqube') {
					bat "${sqScannerMsBuildHome}\\SonarScanner.MSBuild.exe begin /k:org.sonarqube:sonarqube-scanner-msbuild /n:SampleWebApplication /v:1.2.3 /d:sonar.cs.opencover.reportsPaths=coverage.xml"
					bat "'C:\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\MSBuild\15.0\\Bin\\MSBuild.exe' SampleWebApplication.sln /t:Rebuild"
					bat "${sqScannerMsBuildHome}\\SonarScanner.MSBuild.exe end"
               	} 
           	}
      	}
    }
}