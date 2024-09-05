## Jenkins Configuration
In case you are using a Jenkins container instead of installing it on your machine, you will have to:
1. Map your `jenkins_home` directory with your host.
2. Map your host's docker directory with the container.
3. Map your host's docker deamon with the container.
```sh
docker run -d \
  --name jenkins \
  -p 8080:8080 \
  -v ~/Desktop/jenkins-data:/var/jenkins_home \
  -v /usr/bin/docker:/usr/bin/docker \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -u root \
  jenkins/jenkins:lts
```

---

## Task Requirements
1. Clone a GitHub repository.
	1. Delete directory if exists.
	2. Clone the repository.
2. Build an image using the Dockerfile.
	1. Enter project's directory.
	2. Build the image.
3. Push the image to your dockerhub account.
	1. Create a repository on dockerhub.
	2. From Manage Jenkins -> Credentials -> System -> Global credentials, add new credentials of kind `Username with password`, enter your username and password, check `Treat username as secret` and give your credentials an ID.
	3. Use `withCredentials` block:
	```Groovy
	withCredentials([usernamePassword(credentialsId: 'my-dockerhub-cred', usernameVariable: 'myusername', passwordVariable: 'mypassword')]){
	
	}
	```
	4. Login to docker with the credentials.
	5. Tag the image with the dockerhub username.
	6. Push the tagged image.
	7. Logout.
	8. Delete the image and its tag.

---

## Pipeline Script
```Groovy
pipeline {
    agent any

    stages {
        stage('Cloning Repository') {
            steps {
                sh """
                [ -d "simple-dockerfile" ] && rm -rf "simple-dockerfile"
                git clone https://github.com/ianmiell/simple-dockerfile.git
                """
            }
        }
        stage('Building Docker Image') {
            steps {
                sh """
                cd simple-dockerfile
                docker build -t jenkins-task1:latest .
                """
            }
        }
        stage('Pushing to dockerhub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'my-dockerhub-cred', usernameVariable: 'myusername', passwordVariable: 'mypassword')]){
                    sh """
                    docker login -u $myusername -p $mypassword
                    docker tag jenkins-task1:latest $myusername/jenkins-task1:latest
                    docker push $myusername/jenkins-task1:latest
                    docker logout
                    docker rmi jenkins-task1:latest $myusername/jenkins-task1:latest
                    """
                }
            }
        }
    }
}
```
