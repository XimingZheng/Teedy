# Practice 10 - CI/CD with Jenkins and Docker

## What Was Configured

- Jenkins builds the Teedy WAR with Maven.
- Jenkins builds a Docker image from the repository `Dockerfile`.
- Jenkins pushes both `${BUILD_NUMBER}` and `latest` tags to Docker Hub.
- Jenkins recreates three Teedy containers on host ports `8082`, `8083`, and `8084`.

## Jenkins Requirements

- Install the Jenkins **Docker Pipeline** plugin.
- Configure Docker Hub credentials in Jenkins with ID:

```text
dockerhub_credentials
```

- The current Docker Hub image configured in `Jenkinsfile` is:

```text
simonzheng050428/teedy
```

If your Docker Hub repository uses a different account or repository name, set the Jenkins build parameter `DOCKER_IMAGE` to your own `username/repository`.

## Jenkins Pipeline Result to Show

Run the Jenkins job and show that these stages pass:

```text
Checkout
Build WAR
Build Docker Image
Push Docker Image
Run Three Containers
```

## Docker Hub Repository to Show

Open Docker Hub and show the repository:

```text
https://hub.docker.com/r/simonzheng050428/teedy
```

The pushed tags should include:

```text
latest
<Jenkins BUILD_NUMBER>
```

## Three Containers to Show

After the Jenkins job finishes, run:

```bash
docker ps --filter "name=teedy-container"
```

Expected containers and ports:

```text
teedy-container-8082   0.0.0.0:8082->8080/tcp
teedy-container-8083   0.0.0.0:8083->8080/tcp
teedy-container-8084   0.0.0.0:8084->8080/tcp
```

The three application URLs are:

```text
http://localhost:8082
http://localhost:8083
http://localhost:8084
```
