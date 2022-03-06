# 1.kubeadm, kubelet, kubectl
kubeadm : 클러스터를 부트스트랩하는 명령어  
kubelet : 클러스터의 모든 머신에서 실행되는 파드와 컨테이너 시작과 같은 작업을 수행하는 컴포넌트  
kubectl : 클러스터와 통신하기 위한 커맨드 라인 유틸리티  
##### !!주의 컨트롤 플레인 서버보다 kubelet 버전이 높으면 버전 차이 버그 발생!!
##### !!주의 컨트롤 플레인 버전과 kubeadm 버전 일치해야함!!
# 2.CONTROLL-PLANE, NODE
#### CONTROLL-PLANE  
kube-apiserver : k8s API를 사용하도록 요청을 받고 요청이 유효한지 검사  
kuebe-controller-manager : 파드를 관찰하며 개수를 보장  
etcd : key-value 타입의 저장소  
kube-scheduler : 파드를 실행할 노드 선택    
#### NODE
kubelet : 모든 노드에서 실행되는 k8s 에이전트, 데몬 형태로 동작  
kube-proxy : k8s의 network 동작읠 관리, iptable rule을 구성
컨테이너 런타임 : 컨테이너를 실행하는 엔진(ex:docker,containerd,runc)
# 3.namespace
namespace : API를 논리적으로 나눠서 관리하기 쉽게 만듬    
#### CLI  
$kubectl get pod -n default // namespace별 파드 조회    
$kubectl get namespace // namespace 조회  
$kubectl create namespace namespace이름 // namespace 생성  
#### yaml  
$kubectl create namespace namespace이름 --dry-run -o yaml // yaml포맷으로 보기  
test.yaml로 저장  
$kubectl create -f test.yaml // namesapce 생성  
#### base namespace 변경  
$kubectl config view // context 조회  
$kubectl config current-context // 현재 사용하는 context 조회  
$kubectl config set-context context명 --cluster=kubernetes --user=kubenetes-admin --namesapce=blue // context 생성  
$kubectl config use-context context명 // base namespace 변경  
##### !!주의 namespace delete하면 안에 있는 정보 전체 delete됨!!

#### yaml  
yaml : key-value 타입, 들여쓰기는 Tab대신 Space Bar로 사용 

#### api version  
$kubectl explain pod // pod 정보 확인  

# 4.Pod  
$docker build -t 컨테이너명/app.js // 컨테이너 만들기  
$docker push 컨테이너명/app.js // 컨테이너를 docker hub에 push  
컨테이너 하나 = 애플리케이션 하나  
Pod : 컨테이너를 표현하는 k8s API의 최소 단위  
      Pod에는 하나 또는 여러 개의 컨테이너가 포함 될 수 있음  

#### CLI  
##### 동작중인 Pod 정보 보기  
$kubectl get pods  
$kubectl get pods -o wide  
$kubectl describe pod 파드명  
$kubectl get pods --all-namespaces // 전체 namespace에서 동작 pod 확인
##### 동작중인 Pod 수정  
$kubectl edit pod 파드명  
##### 동작중인 Pod 삭제  
$kubectl delete pod 파드명  
$kubectl delete pod -all   
##### 기타  
$kubectl run webserver --image=nginx:1.14 --port=80 // 80 port를 통해서 webserver라는 Pod 생성    
$watch 명령어 // 2초마다 명령어 실행 (뒤에 --watch 옵션으로 넣어도 됨)  
$kubectl exec multipod -c nginx-container -it -- /bin/bash // multipod 안에 nginx-container로 들어가기  
$kubectl logs multipod -c nginx-container // multipod 안에 nginx-container 로그 보기  

##### yaml  
$kubectl create -f webserver.yaml // yaml로 Pod 생성  

### Liveness Probe  
매커니즘  
1. httpdGet probe : 지정한 IP주소, port, path에 HTTP GET 요청하여 응답 확인. 반환코드가 200이 아닌 값이 연속 3번 나오면 오류. 컨테이너를 다시 시작.  
2. tcpSocket probe : 지정된 port에 TCP연결을 시도. 연결되지 않으면 컨테이너를 다시 시작.  
3. exec probe : exec 명령을 전달하고 명령의 종료코드가 0이 아니면 컨테이너 다시 시작.  
예제)exec:  
       command:
         -ls
         -/data/file
##### 옵션  
initialDelaySeconds : 0 // 컨테이너가 만들어진 뒤 0초 뒤에 Liveness 시작  
timeoutSeconds : 1 // timeout 시간 1초  
periodSeconds : 10 // 10초마다 한번씩 검사  
successThreshold : 1 // 1번 성공하면 성공  
failureThreshold : 3 // 3번 실패하면 실패 

#### 초기화 컨테이너 (init container)    
컨테이너 실행 전에 미리 동작시킬 컨테이너  
container가 실행되기 전에 사전 작업이 필요한 경우 사용   
#### infra container(pause)  
pause : 인프라 정보 컨테이너. 컨테이너 생성 시 자동생성  
#### static Pod
static pod : API에게 요청 X. 각 node에서 kubelet demon에서 실행  
기본 /etc/kubernetes/manifests/ 디렉토리에 yaml 파일을 저장 시 적용  
/var/lib/kubelet/config.yaml 에서 변경 후 kubelet demon restart  

#### Pod에 resource 할당  
requests : resource가 여유가 있는 곳으로 할당  
limits : 최대로 쓸 수 있는 resource. 초과할 시 강제 컨테이너 재시작  
         limits 설정만 할 경우 requests 자동으로 똑같이 설정됨  
cpu : 1core 는 1000m  
memory : 100Mi // 1MiB = 1024KiB  

#### Pod 환경변수 설정  
env:
      -name: test
       value: "testvaule"  
#### Pod 실행 패턴  
multi container 패턴 3가지  
Sidecar : 컨테이너 두가지가 같이 실행되야 하는 컨테이너
Adapter : 외부의 데이터를 가공  
Ambassador : 내부에서 외부로 데이터 가공  

# 5.컨트롤러
Controller : Pod의 개수 보장
#### 1)ReplicationController  
요구하는 Pod의 개수를 보장하며 파드 집합의 실행을 항상 안정적으로 유지  
기본 구성 : selector, replicas, template  
spec:  
  replicas: <배포갯수>  
  selector:  
    key:value  
  template:  
    <컨테이너 템플릿>  
kubectl get replicationcontrollers(rc)  
kubectl get pods --show-labels  
kubectl edit rc rc-nginx // rc 설정 편집하기  
kubectl scale rc rc-nginx --replicas=2 // scale 변경  
kubectl delete rc rc-nginx --cascade=false // Pod 삭제하지않고 rc만 삭제
#### 2)ReplicaSet  
ReplicationController보다 풍부한 selector  
spec:  
  replicas: <배포갯수>  
  selector:
    matchLabels:  
      key: value
      version: "2.1"
    matchExpressions:  
    - {key: version, operator: In, value:["2.1","2.2"]} // operator : in,notin,exists(버전존재만하면됨),doesnotexist(버전존재x) 등  
#### 3)Deployment  
ReplicasSet을 컨트롤해서 Pod 개수 조절  
metadata:
  name: deploy_name
  annotations:  
    kubernetes.io/change-cause: version 2.2
kubectl apply -f test.yaml
kubectl set image deployment <deploy_name> <container_name>=<new_version_image> --record // Rolling Update(지속적으로 서비스하며 업데이트)  
kubectl rollout history deployment <deploy_name> //히스토리  
kubectl rollout undo deploy <deploy_name> --to-revision=3 // Rolling Back(롤백)  
kubectl rollout status deployment <deploy_name> // 현재 상태  
kubectl rollout pause deployment <deploy_name> // 일시중지  
kubectl rollout resume deployment <deploy_name> // 재시작  
*kubectl delete rc rc-nginx // rc를 delete해도 다시 rc 생성  
#### 4)Daemon set
#### stateful sets
kubectl set image deployment app-deploy web=nginx:1.15
