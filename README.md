# The supercharged Jenking v2 Docker image

![Supercharged Jenkins](https://github.com/flavioaiello/jenkins-dsl/blob/develop/superhero.png)

This is the ready to go fully declarative supercharged Jenkins Continuous Integration and Delivery server based up on the official release. [http://jenkins-ci.org/](http://jenkins-ci.org/). This one is running in multiple production environments and it's optimized to run on secure zones. The plugins are also part of the image so you don't need access to docker hub to spin up a container.

## Usage

### Docker Compose
To start jenkins using docker compose simply emit the following command:
```
docker-compose up -d
```

#### Docker compose sample recipe (http)

```
version: '2'

services:

  jenkins:
    build: .
    privileged: true
    network_mode: "bridge"
    environment:
      - JAVA_OPTS=-Duser.timezone=Europe/Zurich
      - JENKINS_OPTS=''
      - JENKINS_EXECUTORS=7
      - JENKINS_BUILDSDIR=$${JENKINS_HOME}/builds/$${ITEM_FULL_NAME}
      - JENKINS_BOOTSTRAP_REPOSITORY=yourrepository
      - JENKINS_BOOTSTRAP_REPOSITORY_BRANCH=develop
      - REGISTRY_USERNAME=admin
      - REGISTRY_PASSWORD=changeme
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /tmp/jenkins/workspace:/var/jenkins_home/workspace
      - /tmp/jenkins/builds:/var/jenkins_home/builds
    ports:
        - "8080:8080"
    restart: always
```

### Volumes
This Jenkins Docker image  will store job workspaces, builds and plugins internally to cope with Docker immutable infrastructure concept. To survive updates, it's highly tecommeded to use persistent volumes on your host. 

### Persistance and Backup
Persistance of workspaces and builds is accomplished by binding volumes to the host - you can simply back up this directory. The default configuration comes with a slightly modified directory structure to better comply with persistance:
* /var/jenkins_home/builds
* /var/jenkins_home/workspaces

The sample docker compose excerpt above shows how to bind those volumes. A default groovy init script will seamless rebuild the nextbuild number symlinks.
Refer to the Docker docs section on [Managing data in containers](https://docs.docker.com/userguide/dockervolumes/)

### Providing DSL jobs with GIT
DSL jobs can be maintained on a remote git repository. The jobs itself must be stored into the DSL folder. 

```
docker run -d -p 8080:8080 -e JENKINS_BOOTSTRAP_REPOSITORY=https://your.repo.git
```

### Upgrading
As all the data needed is in the /var/jenkins_home directory - so depending on how you manage that - depends on how you upgrade. If you bind external host volumes for builds and workspaces, your jobs will be updated with jenkins. 

### Executors
The number of build executors can simply be defined by setting an environment variable on your docker container. By default its set to 2 executors, but you can extend the image and change it to your desired number of executors. This can be done by eighter follow the sample docker compose excerpt above or passing parameters to the run command like below:

```
docker build -t jenkins .
docker run --name myjenkins -p 8080:8080 -e JENKINS_EXECUTORS=7 jenkins
```

### Attaching build executors
You can run builds out of the box. If you want to attach non-docker build slaves **through JNLP (Java Web Start)**: make sure you map the port: ```-p 50000:50000``` - which will be used when you connect a slave agent. If you are only using [SSH slaves](https://wiki.jenkins-ci.org/display/JENKINS/SSH+Slaves+plugin), then you do **NOT** need to put that port mapping. If you use the Docker Cloud plugin its strongly recommended to use SSH slaves.
You can also change the default legacy JNLP slave agent port for jenkins by defining `JENKINS_SLAVE_AGENT_PORT` in a derived Dockerfile.

### Passing JVM parameters
You might need to customize the JVM running Jenkins, typically to pass system properties like time zone, or tweak heap memory settings. Use JAVA_OPTS environment
variable for this purpose :

```
docker build -t jenkins .
docker run --name myjenkins -p 8080:8080 -e JAVA_OPTS='-Duser.timezone=Europe/Zurich -Dhudson.footerURL=http://mycompany.com' jenkins
```

### Certificates
You also can define jenkins arguments as `JENKINS_OPTS`. This is usefull to define a set of arguments to pass to jenkins launcher. The following sample Dockerfile uses this option to force use of HTTPS with a certificate. For security reasons the certificate will be stored on the host and mounted in to the container. This can be done by eighter follow the sample docker compose excerpt above or passing parameters to the run command like below:

```
docker build -t jenkins .
docker run --name myjenkins -p 8443:8443 -e JENKINS_OPTS="--httpPort=-1 --httpsPort=8443 --httpsCertificate=/var/jenkins_home/certs/fullchain.pem --httpsPrivateKey=/var/jenkins_home/certs/fullchain.key" jenkins
```

### Installing tools
You can derive this Dockerfile if you need more tools installed. In such a derived image, you can customize your jenkins instance with hook scripts or additional plugins. When jenkins container starts, it will copy the 'src/var/jenkins_home' to the container. 

### Installing plugins
For your convenience, you also can use a plain text file to define plugins to be installed (using core-support plugin format). All plugins need to be listed in the form `pluginID:version`. There ist now transitive dependency resolution, so keep the list small as the dockerfile resolves and installs dependent plugins for you.
```
credentials:1.18
maven-plugin:2.7.1
groovy:latest
...
```

## Configuration
Please define your pipeline or multibranch pipeline jobs directly into `config/jobs.groovy`. If you wish to organize your jobs you can configure views directly in `config/views.groovy`. Use a `Jenkinsfile` on each software repository you wish to build. This setup is built in on jenkins using the plugins as described below.

### JenkinsCI [Job DSL Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Job+DSL+Plugin)
Please refer to the [official tutorial](https://github.com/jenkinsci/job-dsl-plugin/wiki) and the [Job DSL Playground](http://job-dsl.herokuapp.com/). Not all of the 1000+ Jenkins plugins are supported by the built-in DSL. If the API Viewer does not list support for a certain plugin, the Automatically Generated DSL can be used to fill the gap.

### JenkinsCI [Pipeline Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Pipeline+Plugin)
Please refer to the [official tutorial](https://github.com/jenkinsci/pipeline-plugin/blob/master/TUTORIAL.md).

Browse the Jenkins issue tracker to see any open issues on Jenkins and according plugins!

## Contribute
If you want to further customize this image, please feel free to contribute.
