# swarm01
## nginx-nodejs-redis

### Ref
* https://github.com/docker/awesome-compose/tree/master/nginx-nodejs-redis
### url
* https://spcn29swarm01.xops.ipv9.me/

### wakatime project
* https://wakatime.com/@spcn29/projects/zcihtacbcp
---



### ขั้นตอนสำหรับทำงาน    
* nginx-nodejs-redis

    clone แอพ จาก github

1. compose Up ไฟล์ compose.yaml
2. docker login

    ถ้าเคยเข้าสู่ระบบแล้วสามารถข้ามได้เลย

    ```
    docker login 
    ```
    ใส่ Username , password ของ dockerhub

3. เพิ่ม tag 
   
   โดยจะใช้ tag 0227 คือวันที่ในการ push เดือน 02 วันที่ 27
    ``` 
    docker images
    ```
    ``` 
    docker tag nginx-nodejs-redis-web1:latest phisit11/nginx-nodejs-redis-web1:0227

    docker tag nginx-nodejs-redis-web2:latest phisit11/nginx-nodejs-redis-web2:0227

    docker tag nginx-nodejs-redis-nginx:latest phisit11/nginx:0227
    ```
4. push Image to DockerHub
    ``` 
    docker push phisit11/nginx-nodejs-redis-web1:0227
    docker push phisit11/nginx-nodejs-redis-web2:0227
    docker push phisit11/nginx:0227
    ```

![](https://user-images.githubusercontent.com/109591322/224490370-4b90adc9-ad68-41ba-85a4-799b0e544007.png)   

เมื่อ push สำเร็จจะขึ้นหน้าตาแบบรูปด้านบน ที่ dockerhub
### เพิ่ม stacks บน portainer
1. เปิด https://portainer.ipv9.me
2. กด Stacks > add stacks 
3. ใส่ code ลงในส่วนของ Web editor บน portainer
* อยู่ในไฟล์ docker-compose.yaml
    ```
    version: '3'

    services:

    redis:
        image: 'redislabs/redismod'
        networks:
        - backend
    web1:
        image: phisit11/nginx-nodejs-redis-web1:0227
        hostname: web1
        networks:
        - backend
    web2:
        image: phisit11/nginx-nodejs-redis-web2:0227
        hostname: web2
        networks:
        - backend
    nginx:
        image: phisit11/nginx:0227
        depends_on:
        - web1
        - web2
        networks:
        - backend
        - webproxy
        deploy:
            replicas: 1
            labels:
            - traefik.docker.network=webproxy
            - traefik.enable=true
            - traefik.constraint-label=webproxy
            - traefik.http.routers.${APPNAME}-https.entrypoints=websecure
            - traefik.http.routers.${APPNAME}-https.rule=Host("${APPNAME}.xops.ipv9.me")
            - traefik.http.routers.${APPNAME}-https.tls.certresolver=default
            - traefik.http.routers.${APPNAME}-https.tls=true
            - traefik.http.services.${APPNAME}.loadbalancer.server.port=80
            
            resources:
                reservations:
                cpus: '0.1'
                memory: 10M
                limits:
                cpus: '0.4'
                memory: 250M   

    networks:
        webproxy:
            external: true
        backend:
            driver: overlay
    ```
    * Version ระบุเป็น version ที่สนับสนุนกับ application ที่ใช้งาน ควรมากกว่า 3 
    * Service ใช้ระบุส่วนที่ใช้งาน ประกอบไปด้วย image, command, volumes, restart, networks, environment, expose และ deploy เป็นต้น
    * Networks ใช้งานการตั้งค่าเครือข่ายภายนอกคอนเทนเนอร์

4. Add an environment variables
    * Environment variables > Add an environment variables
    * name : APPNAME 
    * value : spcns29swarm01
5. ทดสอบการใช้งาน 
    * ไปที่ https://spcn29swarm01.xops.ipv9.me/  
    ถ้าสามารถใช้งานได้จะขึ้นหน้าตาดังรูปด้านล่าง

![](https://user-images.githubusercontent.com/109591322/222915178-92090cab-c17c-48d9-8899-cbaa3ec5b2e6.png)