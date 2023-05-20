# Docker & Docker Commands

### Dockerfile Example
```Dockerfile
FROM python:3.6 # The base image to pull

RUN pip install flask # Run a linux command

COPY . /opt/ # Copy files/dirs to the container

EXPOSE 8080 # Expose ports

WORKDIR /opt # Set working directory for following instructions

ENTRYPOINT ["python", "app.py"] # Command with args to run in the container
```

### Dockerfile Keywords:
```Dockerfile
FROM - specifies the base(parent) image. Alpine version is the minimal docker image based on Alpine Linux which is only 5mb in size.
RUN - runs a Linux command. Used to install packages into container, create folders, etc
ENV - sets environment variable. We can have multiple variables in a single dockerfile.
COPY - copies files and directories to the container.
EXPOSE - expose ports
ENTRYPOINT - provides command and arguments for an executing container.
CMD - provides a command and arguments for an executing container. There can be only one CMD.
VOLUME - create a directory mount point to access and store persistent data.
WORKDIR - sets the working directory for the instructions that follow.
LABEL - provides metada like maintainer.
ADD - Copies files and directories to the container. Can unpack compressed files.
ARG - Define build-time variable.```
```

### Docker Commands
```sh
# List images on the system
docker images
docker image ls

# build an image from a Dockerfile with multiple tags
docker build -t <tag> -t <tag> <path-to-Dockerfile>

# Tag an existing image
docker tag <image:tag> <target_image:tag>
# Example: docker tag httpd httpd:v1.2

# Save images to a tar archive
docker save image -o <output-tar>

# Load images from a tar archive
docker load <tararchive>

# run an image as a container using a port
docker run -p <hostport:containerport> <image:tag>

# push to a repo
docker push <image:tag>
```
