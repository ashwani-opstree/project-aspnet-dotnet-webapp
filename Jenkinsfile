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
        stage('SonarQubeAnalysis Begin') {
			steps {
                script {
             		sqScannerMsBuildHome = tool 'sonarscanner-msbuild-4.3';
        		}
    			withSonarQubeEnv('sonarqube') {
					bat "${sqScannerMsBuildHome}\\SonarScanner.MSBuild.exe begin /k:org.sonarqube:sonarqube-scanner-msbuild /n:SampleWebApplication /v:1.2.3 /d:sonar.cs.opencover.reportsPaths=coverage.xml"
               	} 
           	}
      	}
        stage('BuildArtifact') {
            steps {
                bat "\"C:\\Program Files (x86)\\NuGet\\bin\\nuget\" restore"
                bat "\"C:\\Program Files (x86)\\Microsoft Visual Studio\\2017\\Community\\MSBuild\\15.0\\Bin\\MSBuild.exe\" SampleWebApplication.sln /t:Package /target:Clean /p:FilesToIncludeForPublish=AllFilesInProjectFolder /p:Configuration=Release" 
                bat "C:\\opencover\\OpenCover.Console.exe -target:\"c:\\Program Files\\dotnet\\dotnet.exe\" -targetargs:\"test -f netcoreapp1.0 -c Release UnitTestProject\\UnitTestProject.csproj\" -mergeoutput -hideskipped:File -output:coverage.xml -oldStyle -filter:\"+[UnitTestProject*]*\" -register:user"
            }
        }
        stage('SonarQubeAnalysis End') {
			steps {
                script {
             		sqScannerMsBuildHome = tool 'sonarscanner-msbuild-4.3';
        		}
    			withSonarQubeEnv('sonarqube') {
					bat "${sqScannerMsBuildHome}\\SonarScanner.MSBuild.exe end"
               	} 
           	}
      	}
        stage('Release') {
            steps {
				nexusPublisher nexusInstanceId: 'nexus3', 
				nexusRepositoryId: 'dotnet-release', 
				packages: [[
					$class: 'MavenPackage', 
					mavenAssetList: [[
						classifier: '', 
						extension: '', 
						filePath: 'SampleWebApplication\\obj\\Release\\Package\\SampleWebApplication.zip'
					]], 
					mavenCoordinate: [
						artifactId: 'dotnet-webapp', 
						groupId: 'com.dotnet.webapp', 
						packaging: 'zip', 
						version: '1.1.0'
					]
				]]				
            }
		} 
		stage('Deploy App'){
			steps {
				powershell '''
					$Body = @{
    					User     = 'admin'
    					password = 'admin123'
					}
					Remove-Item C:\\www\\* -Recurse -Force
					Invoke-WebRequest -Uri http://18.210.172.8:8081/repository/dotnet-release/com/dotnet/webapp/dotnet-webapp/1.0.0/dotnet-webapp-1.0.0.zip -Body $Body  -OutFile C:\\www\\sampleproject.zip
					Expand-Archive C:\\www\\sampleproject.zip -DestinationPath c:\\www 
					New-Item -Path C:\\www\\app -ItemType Directory
					Copy-Item -Path C:\\www\\Content\\C_C\\Users\\Jenkins\\workspace\\Dotnet-webapp\\SampleWebApplication\\obj\\Release\\Package\\PackageTmp\\* -Destination C:\\www\\app\\ -recurse -Force
					'''
			}
		}
    }
}