쿠버네티스 클러스터 상에서 cAdvisor 시작하는 법

1. 도커 버전 확인 
  docker version

2. 도커 버전이 낮다면 도커를 1.13.1 이상으로 업그레이드
  yum install upgrade docker.io
  (yum update)

3. cAdvisor 실행
1) Docker Registry Pull
  docker pull google/cadvisor
  docker ps
  
2) Docker Image 실행

   docker run --volume=/var/run:/var/run:rw --volume=/cgroup:/cgroup:ro --volume=/var/lib/docker:ro --publish=8080:8080 --detach=true --   name=cadvisor google/cadvisor:latest

3) cAdvisor 확인
  docker ps

4) http://localhost:8080 접속

[Trouble shooting]

1. docker ps 경우 문제가 생겨 목록에 없는 경우
docker ps -as로 어떤 오류메세지인지 확인

2. 혹은 docker logs cadvisor로 로그 확인

