1. Docker Stat
Usage : docker stats

 - 각 컨테이너의 cpu 이용량, 사용 중인 메모리와 전체 메모리, 네트워크 전송량 정보를 제공
 - docker remote api를 직접 개발해 더 자세한 stat 정보를 얻을 수 있음

2. CAdvisor
 - docker stats, remote API 커맨드 라인에서 정보를 얻기에는 유용하지만 그래프 형태로 보기에는 적합하지 않음
 - CAdvisor는 docker stat에 의한 정보를 비주얼 하게 제공
 - cpu, memory, network, disk-space 리소스를 그래프로 확인 가능
 - 설치와 사용이 간편함
 - aggregation 지원
 - 오픈소스이며 클러스터에서 provisioned 되어 동작함
<단점>
 - 하나의 docker host만 모니터 가능
 - 멀티 노드를 모니터 하려면 heapster를 사용해야 함.
 - alerting 메커니즘이 없음
 - rancher에서 cAdvisor를 사용

3.Scout
 - Scout는 많은 호스트와 컨테이너 그리고 오랜 시간에 걸친 데이터의 metrics를 aggregate 할 수 있는 hosted monitoring 서비스 입니다. metrics 기반으로 alert도 만들 수 있습니다.
 - scout계정을 생성 후 accout_key를 생성함
 - 호스트에 scoutd.yml 파일을 생성 후 accout_key와 모니터링 대상의 정보를 작성
 - scout agent 컨테이너를 실행
 - scout web view에서 모니터링이 되는 걸 확인
 - trigger 설정
 - plugins set? 때문에 cAdvisor에 비해 도커 정보 이외에도 다른 정보를 가져올 수 있습니다.
 - 원스탑 모니터링 시스템
<단점>
- cAdvisor처럼 개별 컨테이너의 detail한 정보를 제공하지 않음. 
 - 하나의 서버에서 여러 종류의 컨테이너를 실행하면 문제의 소지가 될 수 있음.(alert의 트리거)

4. Promethues
 - 모니터링 대세
 - visor + Promethues + Grafana 조합하여 사용하는 것이 많음.
