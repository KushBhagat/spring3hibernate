pipeline {
	agent any
	
	tools {
		maven 'M3'
		jdk 'jdk1.8'
		}
	
	options {
		buildDiscarder(logRotator(numToKeepStr:'30'))
		timeout(time:1, unit: 'HOURS')
	}
	
	parameters {
		string(name: 'PHASE', defaultValue: 'CLEAN,BUILD,TEST,SONAR,QUALITYGATE,', description: 'COMPLETE for new branches to build required test packages, CLEAN for maven clean')
		string (name: 'repo',defaultValue: '', description: '', trim: false)
}
	
	stages 
	{
               stage('Checkout') {
                     steps 
					 {   
                      checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'http://github.com/KushBhagat/${params.repo}.git']]])
                     }    
                }

		stage('Clean') {
			when {
				expression { params.PHASE.contains('CLEAN')  }
			}
			steps {
				withMaven(
				maven: 'M3') {
					sh 'mvn clean'
				}
			}
		}
        stage('CompleteBuild') 
						{
                          when {
                                expression { params.PHASE.contains('COMPLETE')  }
                               }
                          steps 
						       {
                                withMaven(
                                        maven: 'M3',
                                        mavenOpts: '-XX:+TieredCompilation -XX:TieredStopAtLevel=1',
                                        options:[artifactsPublisher(allowEmptyArchive:true),junitPublisher(disabled:true),openTasksPublisher(disabled:true)]) {
                                                sh 'mvn -U -Dmaven.test.skip=false -Dmaven.includeScope=compile -Dmaven.test.failure.ignore=true install'
                                        }
                                }
                        }
        
		stage('Build') 
		{
			when {
				expression { params.PHASE.contains('BUILD')  }
			}
			steps {
				withMaven(
					maven: 'M3',
					mavenOpts: '-XX:+TieredCompilation -XX:TieredStopAtLevel=1',
					options:[artifactsPublisher(allowEmptyArchive:true),junitPublisher(disabled:true),openTasksPublisher(disabled:true)]) 
					{
						sh 'mvn -f pom.xml -Dmaven.test.skip=true --fail-at-end install'
					}
				
			}
		}
		stage('Test') {
		when {
			expression { params.PHASE.contains('TEST')  }
			}
		steps {
			withMaven(
				maven: 'M3',
				mavenOpts: '-XX:+TieredCompilation -XX:TieredStopAtLevel=1',
				options:[artifactsPublisher(disabled:true),junitPublisher(allowEmptyResults:true),openTasksPublisher(disabled:true)]) 
				{
					sh 'mvn -f pom.xml -Dmaven.test.failure.ignore=true --fail-at-end test'
				}
			}
		}
			
		stage('Sonar') {
			
			when {
				expression { params.PHASE.contains('SONAR')  }
			}	
			steps {
				echo 'Sonar'
				
				timeout(time: 45, unit: 'MINUTES') 
				{
						//My SonarQube Server is configured in global tool configuration of Jenkins
                              withSonarQubeEnv('My SonarQube Server') 
							  {  withMaven(maven: 'M3',)
                    			       {sh 'mvn -f pom.xml sonar:sonar'}
							  }
						
				}
			}
		}
		stage("Quality Gate") {
		
		  when {
				expression { params.PHASE.contains('QUALITYGATE')  }
			} 
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
		stage('Deploy') {
			when {
				expression { params.PHASE.contains('DEPLOY')  }
			}
			steps {
				withMaven(maven: 'M3',
						options: [artifactsPublisher(disabled: false),openTasksPublisher(disabled: true)]) 
						{ sh "mvn -Dmaven.test.skip=true deploy"  }
			}
		}
	
	}
} 
