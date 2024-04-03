---
title: Alpaquita Containers Reduce Container Footprint for a Spring Boot App
tags: [core java, java, junit]
thumbnail-img: "/assets/img/blog/docker-coffee.jpg"
gh-repo: dmakariev/examples
gh-badge: [star, fork, follow]
---

I've just tried the new **Alpaquita Containers**. The result in my case was that 
the image size went down from **370MB** to `162MB`.
That is about `56%` less, which is pretty good ! 

## How I tested it ?  
I've used the example code from one of my previous articles [How to Build a Spring Boot Monolith with JBang](https://www.makariev.com/blog/how-to-build-spring-boot-monolith-with-jbang/)
if you want to try it yourself, you can clone the `https://github.com/dmakariev/examples` repository.
```bash
git clone https://github.com/dmakariev/examples.git
cd examples/jbang/spring-boot-jpa-vue
``` 

this will build the image based on `amazoncorretto:21-alpine`
```bash
docker compose build 
```

then to build the image based on `bellsoft/liberica-runtime-container:jdk-21-slim-musl` 
you have to execute the following 
```bash
docker compose -f bellsoft-compose.yaml build  
```

As I've written above - the result is `56%` smaller image 
you could see for yourself by executing 
```bash
docker images  
```
here is the output I received 
```bash 
REPOSITORY                    TAG         IMAGE ID       CREATED             SIZE
spring-boot-jpa-vue_backend   bellsoft    810e09bf2907   12 seconds ago      162MB
spring-boot-jpa-vue_backend   aws-alpine  d9e33d5a9219   About a minute ago  370MB
```
Give them a try, you can browse the images here [https://hub.docker.com/r/bellsoft/alpaquita-linux-base](https://hub.docker.com/r/bellsoft/alpaquita-linux-base)

Thanks and..

---

[![Coffee Time!](/assets/img/blog/docker-coffee.jpg)](/assets/img/blog/docker-coffee.jpg)

Happy coding!



