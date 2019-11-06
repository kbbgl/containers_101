### Docker Compose v1

Following is an example of a multiservice deployment of 2 containers:

```yaml
# docker-compose.yml
web:
    build: . # context of the build. Will use the Dockerfile in current directory
    ports:
        - 5000:5000
    links:
        - mongo
mongo:
    image: mongo
```

If we then run `docker-compose build`,  `docker` will find the default `docker-compose.yml` file and use it to build the containers according to the instructions.

After building, we can run `docker-compose up` to start the application.

`docker-compose stop` will stop all running images.

`docker-compose rm` will delete all running images.


### Docker Compose v2

Has three layers:
1) Services
2) Networks
3) Volumes

```yaml
# docker-compose.yml
version: '2'
services:
    web:
        build: 
            context: .
            dockerfile: Dockerfile
        image: demoapp:0.1
    mongo:
        image: mongo
```