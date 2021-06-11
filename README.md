# harbor-private-docker-registry

Network ports Open
```
443, 4443, 80, 2357,…
```



Harbor 다운로드 및 압축풀기
```
참고 url : https://github.com/goharbor/harbor/releases
```



Latest release, Verified 된 버전으로 링크를 확인
```
$ wget https://github.com/goharbor/harbor/releases/download/v2.2.1/harbor-offline-installer-v2.2.1.tgz

$ tar xzvf harbor-offline-installer-v2.2.1.tgz
```



인증서 파일, 키 파일 복사 -  다루기 편하도록 두 파일을 cert 디렉토리로 복사. 공식 홈페이지에서는 OpenSSL을 이용한 방식의 .crt파일과 .key파일을 넣어주고 있지만 pem파일을 그대로 넣어도 잘 동작함.
```
$ cp reg.example.com.crt.pem /data/cert/

$ cp reg.example.com.key.pem /data/cert/
```



harbor.yml 설정파일 입력
```
$ cd harbor

$ cp harbor.yml.tmpl harbor.yml

$ vi harbor.yml

hostname: reg.example.com




  certificate: /data/cert/reg.example.com.crt.pem

  private_key: /data/cert/reg.example.com.key.pem




harbor_admin_password: mypassword




database:

  # The password for the root user of Harbor DB. Change this before any production use.

  password: mypassword
```



도커 데몬 인식되는 cert 변환
```
$ openssl x509 -inform PEM -in reg.example.com.crt -out reg.example.com.cert
```



인증서 도커 내에 복사
```
$ cp reg.example.com.cert /etc/docker/certs.d/reg.example.com:443/

$ cp reg.example.com.key /etc/docker/certs.d/reg.example.com:443/

$ cp ca.crt /etc/docker/certs.d/reg.example.com:443/ <= CA file 복사
```



도커 재시작
```
$ systemctl restart docker
```



docker-compose 파일을 다시 생성
```
$ sudo ./prepare
```



다시 생성된 docker-compose 파일을 적용하기 위해 harbor를 실행합니다.
```
$ sudo docker-compose up -d
```



도메인 주소 이동 및 인증서 적용 확인
```
https://reg.example.com
```



Stop Harbor 
```
$ docker-compose down -v
```



Restart Harbor
```
$ docker-compose up -d
```



scp 로 파일 이동(로컬 → 원격지 192.168.1.6)
```
$ scp -P 1022 _wildcard_.example.com.crt.pem root@192.168.1.6:/tmp
```



클라이언트 호스트에서 접근가능하도록 설정
```
$ cp reg.example.com.crt.pem /etc/ssl/cert/reg.example.com.crt.pem

$ update-ca-certificates
```



도커 재시작
```
$ systemctl restart docker
```



도커 로그인
```
$ docker login https://reg.example.com

Login Succeeded
```



harbor 에 컨테이너 이미지를 pull, push
```
$ docker pull busybox:latest

$ docker tag busybox:latest reg.example.com/public/busybox:v1

$ docker push reg.example.com/public/busybox:v1
```


```
$ docker rmi reg.example.com/public/busybox:v1

$ docker rmi busybox
```


```
$ docker pull reg.example.com/public/busybox:v1

$ docker tag reg.example.com/public/appjs:latest reg.example.com/public/busybox:v1

$ docker push reg.example.com/public/appjs:latest
```
