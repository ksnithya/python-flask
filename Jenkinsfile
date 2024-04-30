pipeline{
    agent any
    environment{
        Region = "ap-south-1"
        Name = "python-flask-demo"
        TAG = "v1"
        VERSION = "${env.BUILD_ID}"
        registry="822626997628.dkr.ecr.ap-south-1.amazonaws.com"
        github_token = credentials('github-token')
        
    }
    stages{
        stage("Check out"){
            steps{
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/ksnithya/python-flask.git']])
            }
        }
        stage("Build Docker Image"){
            steps{
                script{
                    echo "Building ${Name} image"
                    sh 'docker build -t ${Name}:${TAG} .'
                    sh 'docker tag ${Name}:${TAG} ${registry}/${Name}:${VERSION}'
                    
                }
            }
        }
        stage("Push Im age to ECR"){
            steps{
                script{
                    sh 'aws ecr get-login-password --region ${Region} | docker login --username AWS --password-stdin ${registry}'
                    sh 'docker push ${registry}/${Name}:${VERSION}'
                }
            }
        }
        stage('Clone/Pull Repo') {
            steps {
                script {
                    if (fileExists('eks-python-demo')) {

                        echo 'Cloned repo already exists - Pulling latest changes'

                        dir("eks-python-demo") {
                        sh 'git pull'
                        }

                    } else {
                        echo 'Repo does not exists - Cloning the repo'
                        sh 'git clone -b feature https://github.com/ksnithya/eks-python-demo.git'
                        dir("eks-python-demo"){
                            sh "ls -l"
                        }
                    }
                }
            }
        }
        stage('Update Manifest') {
            steps {
                dir("eks-python-demo") {
                    sh 'sed -i "s|image: .*$|image: ${registry}/${Name}:${VERSION}|" deploy.yml'
                    sh 'cat deploy.yml'
                }
            }
        }
        stage('Commit & Push') {
            steps {
                dir("eks-python-demo") {
                    // Configure Git
                    sh "git config --global user.email 'ksnithyamsc@gmail.com'"
                    sh 'git remote set-url origin https://$github_token@github.com/ksnithya/eks-python-demo.git'
                    
                    // Commit changes to feature branch
                    sh 'git checkout feature'
                    sh 'git add -A'
                    sh 'git commit -am "Updated image version for Build - $VERSION"'
                    sh 'git push origin feature'
                }
            }
        }
        stage('Merge Request') {
            steps {
                dir("eks-python-demo") {
                    sh "git config --global user.email 'ksnithyamsc@gmail.com'"
                    sh 'git remote set-url origin https://$github_token@github.com/ksnithya/eks-python-demo.git'
                    sh 'git checkout feature'
                    
                    // Prepare main branch
                    sh 'git fetch --all'
                    sh 'git checkout main'
                    sh 'git pull origin main'
                    
                    // Merge feature into main
                    sh 'git merge -m "merging to main branch" origin/feature'
                    
                    // Push changes
                    sh 'git push origin main'
                }
            }
        }
    }
}
