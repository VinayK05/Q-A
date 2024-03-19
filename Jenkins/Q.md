

** About Jeknins pipeline execution on multiple agents **


In all the previous examples, only a single agent has been used. This means Jenkins will allocate an executor wherever one is available, regardless of how it is labeled or configured. Not only can this behavior be overridden, but Pipeline allows utilizing multiple agents in the Jenkins environment from within the same Jenkinsfile, which can helpful for more advanced use-cases such as executing builds/tests across multiple platforms.

In the example below, the "Build" stage will be performed on one agent, and the built results will be reused on two subsequent agents, labelled "linux" and "windows" respectively, during the "Test" stage.

Declarative syntaxScripted syntax

``
pipeline {

    agent none
    
    stages {
    
        stage('Build') {
        
            agent any
            
            steps {
            
                checkout scm
                
                sh 'make'
                
                stash includes: '**/target/*.jar', name: 'app' 
                
            }
            
        }
        
        stage('Test on Linux') {

            agent { 
            
                label 'linux'
                
            }
            
            steps {
            
                unstash 'app'
                
                sh 'make check'
                
            }
            
            post {
            
                always {
                
                    junit '**/target/*.xml'

                }
                
            }
            
        }
        
        stage('Test on Windows') {
        
            agent {
            
                label 'windows'
                
            }
            
            steps {
            
                unstash 'app'
                
                bat 'make check' 
                
            }
            
            post {
            
                always {
                
                    junit '**/target/*.xml'
                    
                }
                
            }
            
        }
        
    }
    
}


``

The stash step allows capturing files matching an inclusion pattern (**/target/*.jar) for reuse within the same Pipeline. Once the Pipeline has completed its execution, stashed files are deleted from the Jenkins controller.
The parameter in agent/node allows for any valid Jenkins label expression. Consult the Pipeline Syntax Reference Guide for more details.
unstash will retrieve the named "stash" from the Jenkins controller into the Pipeline’s current workspace.
The bat script allows for executing batch scripts on Windows-based platforms.
