 
그라파나 연동

1. 그라파나(Grafana) 소개
2. 로그인 비밀번호 셋팅
3. 그라파나 설치
4. 그라파나 콘솔 접근
5. 데이터 소스(Data Sources)설정
6. 모니터링 정보 확인

 

그라파나(Grafana) 소개

특정 시스템에 대한 상태와 모니터링 그리고 오류 분석은 데이터를 관리하는 입장에서는 항상 신경써야 하는 부분이고, 시스템의 정상적인 구동을 관리 하기 위해 서비스 해야하는 부분이다.
이러한 모니터링 도구들은 대체로 도입 비용이 비싸거나, 오픈소스로 제공되는 것은 품질이나 시스템을 확장하려 할 때 문제점이 발생하곤 한다. 또한 특정언어나 시스템에 종속적이어서, 하나의 모니터링 도구에서 다양한 시스템을 모니터링 하는 것은 쉽지 않다.
하지만 모니터링 도구로서  다양한 기능과 시각화를 제공하는 오픈소스인 그라파나가 등장하게 되었다.
웹 애플리케이션으로 동작하는 오픈소스 다목적 대시보드로서 데이터를 시각화 하여 효과적인 데이터 기반 의사결정 및 모니터링을 지원한다.

그라파나의 기능은 다음과 같다.

1. 다양한 시각화를 위한 그래프를 지원하며 사용자가 커스터마이징 할 수 있음.
2. graphite, influxdb, opentsdb, ElasticSearch, Azure 등을 지원

 

로그인 비밀번호 셋팅
1) 프로메테우스로 쿠버네티스 클러스터 모니터링 하기 에서 다운받은 Chart를 이용
 
 $ cd [chart 디렉토리]/charts/stable/grafana

2) values.yaml을 열어 admin 사용자의 비밀번호를 변경

 $ vi values.yaml
 find /adminPassword

 adminUser: admin
 adminPassoword: **********



그라파나 설치
Helm Chart(*) 를 이용하여 그라파나를 설치한다. 네임스페이스는 grafana로 한다.
cd [chart 디렉토리]/charts/stable/grafana

 $ helm install -f values.yaml stable/grafana --name grafana --namespace grafana
 
grafana가 설치된다.
 
이제 그라파나의 콘솔로 접근하기 위해 배포된 팟(pod)의 이름을 알아야 한다.
팟의 목록을 나열하여 이름을 찾는다.

 $ kubectl get pod -n grafana
 
현재 배포된 팟의 목록이 나타난다.

 
해당 grafana 포트는 외부로 노출되지 않기때문에 프로메테우스 연동과 마찬가지로 포트 포워딩을 통해 해당 서버에 접속할 수 있게 해 준다. 보통 그라파나는 3000포트로 웹 접속을 할 수 있다. 아래와 같이 3000포트로 포트 포워딩을 해준다.

 $ kubectl port-forward -n grafana grafana-5bd4cf9965-hsnsq 3000
 


그라파나 콘솔 접근
이제 localhost:3000으로 접근해보면 아래와 같이 그라파나의 로그인 창을 볼 수 있다.
아까 설정한 admin과 설정한 Password로 로그인 한다. 

  
데이터 소스(Data Sources) 설정
모니터링을 위한 메트릭을 그라파나에 연결해야 데이터 시각화가 가능하다. 이는 데이터 소스 설정을 통해 할 수 있다.  로그인 후 왼쪽의 아이콘 중에 톱니바퀴에 마우스를 가져가면 Configuration → Data Sources를 설정할 수 있다. Data Sources를 선택하면 다음과 같이 화면이 나타난다.
Add data source를 클릭하면 다양한 데이터 소스 종류가 나타나며 이전에 연동했던 프로메테우스 메트릭을 선택한다.

Prometheus를 선택하고 나면 Data Sources에 대한 정보를 입력해야하는데 프로메테우스의 서버의 URL을 알아야 해당 메트릭을 등록할 수 있다. 해당 프로메테우스의 서버의 서비스 명을 알기위해 다음과 같이 입력한다.

 $ kubectl get svc -n prometheus

prometheus 의 서비스 목록을 볼 수 있다.


prometheus-server의 이름의 서비스의 내부 Cluster ip가 10.111.195.14로 포트는 80으로 되어 있다.

이 정보를 Data Sources에 설정해 준다. 이름은 편의상 Kubernetes로 설정하고 URL에 prometheus-server ip를 설정하고 Access는 Server(Default)를 선택한다. 그리고 Save & Test를 클릭하면 연결테스트 후에 소스가 업데이트 되었다고 메세지가 뜬다.



모니터링 정보 확인
모니터링을 위한 다양한 그래프나 특정 정보를  패널에 추가할 수 있다. 왼쪽 + 의 Import를 통해 기존의 만들어진 대시보드를 불러와서 설정할수도 있으나 여기서는 한번 원하는 쿼리정보를 추가해보도록 한다.

Dashboard 탭을 선택하고 Kubernetes라는 데이터 소스에 Add Query를 통해 node_disk_io_now라는 정보의 그래프를 추가해 본다. Time Series하게 해당 정보의 그래프가 표시됨을 확인할 수 있다.


Prometheus 2.0 Stats의 지표를 보면 프로메테우스를 통해 얻을수 있는 지표의 수치들을 확인할 수 있다.


 
참고내용
Helm Chart : helm은 쿠버네티스의 하위 프로젝트로 시작되었다가 2018년 6월에 CNCF재단의 정식 프로젝트로 승격되었다. Helm은 차트를 만들고, 차트 압축 파일(tgz)를 만들수 있는 쿠버네티스 패키지 배포를 가능하게 하는 툴이라고 볼 수 있다.

Helm 공식 사이트 : https://helm.sh/docs/

그라파나 공식사이트 : https://grafana.com/

그라파나 라이브 데모영상 : http://play.grafana.org/ 
