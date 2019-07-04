프로메테우스 소개

Helm Chart 를 이용한 프로메테우스 설치
미니큐브(minikube) 실행
Helm 설치 방법
Chart를 이용한 프로메테우스 설치
쿠버네티스 환경에서 프로메테우스 모니터링 연동
포트포워딩으로 포트 expose
각 모니터링 대상 및 지표 확인
 
 

1. 프로메테우스 소개
 

프로메테우스는 모니터링 솔루션으로 가장 많이 언급되는 모니터링 툴로 2016년 CNCF(Cloud Native Computing Foundation) 에 오픈소스로 기부되었고, 지표수집을 통한 모니터링을 주요 기능으로 한다.
애플리케이션, 서버, OS 등 다양한 지표를 수집하여 모니터링 할 수 있으며 다양한 그래픽 모드와 대시보드를 지원한다.
서비스 디스커버리를 통해 클라이언트를 검색하며 시계열(Time Series) 데이터를 수집한다.
프로메테우스는 Pull 방식으로 서버가 주기적으로 클라이언트에 접속해서 데이터를 가져오는 방식으로 동작한다.


2. Helm Chart를 이용한 프로메테우스 설치 
 

프로메테우스는 OS별로 다운로드 페이지를 통해 다운로드 받을 수도 있고 도커 이미지로 제공도 하고 있다.

다운로드 페이지: https://prometheus.io/download/

도커이미지로 설치: https://prometheus.io/docs/prometheus/latest/installation/


여기서는 미니큐브(minikube)의 단일 클러스터 환경에서 Helm Chart를 이용해 프로메테우스를 설치해 보도록 한다.

 

1. 미니큐브(minikube) 실행
 

미니큐브를 실행한다. 미니큐브를 설치하는 방법은 따로 설명하지 않는다. 

 

2. Helm 설치방법
 

미니큐브 클러스터가 실행되었으므로 Helm을 설치한다.
Helm은 서버(Tiller)와 클라이언트 두개의 모듈이 있으며  설치는 다음 링크(https://helm.sh/docs/using_helm/)를 통해 할 수 있다. 미니큐브 환경에서는 다음과 같이 tiller를 자동으로 실행할 수 있다.

$ helm init
 

3. Chart를 이용한 프로메테우스 설치
 

git clone 으로 chart를 다운로드 받는다.

$ git clone https://github.com/kubernetes/charts
 

chart 설치 후에 /charts/stable/prometheus폴더로 이동

$ cd /charts/stable/prometheus
 
prometheus라는 네임스페이스에 prometheus라는 이름으로 설치를 한다.

$ helm install -f values.yaml stable/prometheus --name prometheus --namespace prometheus
 

이제 prometheus라는 이름의 네임스페이스로 프로메테우스가 설치된다.



3. 쿠버네티스 환경에서 프로메테우스 모니터링 연동 
 

1. 포트포워딩으로 포트 expose
 

프로메테우스를 설치하였으므로 쿠버네티스 환경의 지표를 수집하는지 확인해야 한다.
prometheus라는 이름의 네임스페이스를 만들었으므로 해당 네임스페이스의 팟(pod) 목록을 확인한다.

$ kubectl get pod -n prometheus


여기서 주의깊게 보아야 할것은 prometheus-server-**** 팟 이름이다. 통상 프로메테우스는 9090포트로 웹 인터페이스를 제공하며 해당 웹 인터페이스를 보기 위해서는 외부서비스를 expose해야 확인이 가능하다.
그러므로 prometheus-server-5947775fbc-qdpbk의 팟을 9090 포트포워딩으로 노출해 주도록 한다.


kubectl port-forward -n <name> <namespace> <port number>로 노출한다.

$ kubectl port-forward -n prometheus prometheus-server-5947775fbc-qbobk 9090

이제 9090포트가 노출되었으므로 localhost:9090으로 접근해보면 대시보드를 볼 수 있다.



2. 각 모니터링 대상 및 지표 확인
 

다음으로 모니터링 할 대상들을 살펴본다.  상단의 탭에서 Status를 선택한 후 Service Discovery를 선택한다.
kubernetes-apiserviers, kubernetes-node 등 쿠버네티스 관련 대상들에 대한 목록이 표시된다.


그럼 이제 Target 메뉴를 선택하면 각각의 모니터링 대상들에 대한 지표를 어디서 가져오는지 알 수 있다. (정상일때는 up상태이다. 순간적으로 클러스터가 stop되서 스크린샷은 State가 DOWN으로 되어있다)
 

Graph에서 각각의 대상에 대한 모니터링 상태를 다음과 같이 확인할 수 있다. 

 

참고 
참고자료: https://prometheus.io/docs/introduction/overview/
