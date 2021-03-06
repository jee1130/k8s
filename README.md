# 1.kubeadm, kubelet, kubectl
kubeadm : 클러스터를 부트스트랩하는 명령어  
kubelet : 클러스터의 모든 머신에서 실행되는 파드와 컨테이너 시작과 같은 작업을 수행하는 컴포넌트  
kubectl : 클러스터와 통신하기 위한 커맨드 라인 유틸리티  
##### !!주의 컨트롤 플레인 서버보다 kubelet 버전이 높으면 버전 차이 버그 발생!!
##### !!주의 컨트롤 플레인 버전과 kubeadm 버전 일치해야함!!
source <(kubectl completion bash)  
echo "source <(kubectl completion bash)" >> ~/.bashrc  
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
ReplicationController보다 풍부한 selector(lable 지원)  
spec:  
  replicas: <배포갯수>  
  selector:
    matchLabels:  
      key: value
      version: "2.1"
    matchExpressions:  
    - {key: version, operator: In, value:["2.1","2.2"]} // operator :  
    in,notin,exists(버전존재만하면됨),doesnotexist(버전존재x) 등  
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
kubectl set image deployment app-deploy web=nginx:1.15  
*kubectl delete rc rc-nginx // rc를 delete해도 다시 rc 생성  
*Rolling update & Rollback 기능 지원  
#### 4)Daemon set  
전체 노드에서 Pod가 한 개씩 실행되도록 보장  
로그 수입기, 모니터링 에이전트와 같은 프로그램 실행 시 적용  
kubeadmin token list // 현재 보유한 token list 출력  
kubeadmin token create --ttl 1h // 1시간동안 token 생성 (token은 24시간 후 지워짐)  
kubeadmin join ip:port --token <token>  
*Rolling update & Rollback 기능 지원  
#### 5)stateful sets  
Pod의 상태(이름,볼륨)를 유지해주는 컨트롤러  
spec:  
  replicas: 3
  serviceName: <name>  
  podManagementPolicy: <옵션> // OrderedReady(순차적), Parallel(동시에)  
kubectl scale statefulset <Pod> --replicas=2  
kubectl edit statefulsets.apps <name>  
#### 6)job  
-Pod를 running 중인 상태로 유지  
-Batch 처리하는 Pod는 작업이 완료되면 종료됨  
-Batch 처리에 적합한 컨트롤러로 Pod의 성공적인 완료를 보장  
spec:  
  completions: 3 // 3번 실행  
  parallelism: 2 // 동시에 2개 실행  
  activeDeadlinSeconds: 5 // 저장 시간 내에 Job을 완료  
  template:  
    spec:  
      restartPolicy:Never // OnFailure(비정상종료 시 컨테이너만 restart), Never(Pod restart)  
  backoffLimit: 3 // 컨테이너 restart 시 3번까지만 restart(기본 6번)  
kubectl get jobs // job 상태 확인  
kubectl delete jobs.batch centos-jab // Pod말고 Job만 종료  
#### 7)CronJob  
-Job 컨트롤러로 실행할 Application Pod를 주기적으로 반복해서 실행  
-Data Backup, Send email, Cleaning tasks  
cronjab schedule: "분 시 일 월 주"  
spec:  
  schedule: "* * * * *"  
  startingDeadlineSeconds: 300  
  concurrencyPolicy: Allow // Allow(한번에 여러개 Job running중 가능), Forbid(한번에 동작 X)  
  successfulJobHistoryLimit: 3 // 최근 3(default)개만 history  
  JobTemplate:  
    spec:  
# 6.서비스(API)  
1) Kubernets Service  
      - 동일한 서비스를 제공하는 Pod 그룹의 단일 진입점을 제공  
2) Service 종류  
ClusterIP(default)  
      - Pod 그룹의 단일 진입점(Virtual IP) 생성(10.96.0.0/12)  
NodePort  
      - ClusterIP 가 생성된 후 모든 Worker Node에 외부에서 접속가능한 포트가 예약(30000-32767)  
LoadBalancer  
      - 클라우드 인프라스트럭쳐(AWS, Azure, GCP 등)나 오픈스택 클라우드에 적용  
ExternalName  
      - 클러스터 안에서 외부에 접속 시 사용할 도메인 등록해서 사용  
kubectl get service  
3) headless 서비스  
      - ClusterIP가 없는 서비스로 단일 진입점이 필요 없을 때 사용  
      - Service와 연결된 Pod의 endpoint로 DNS 레코드가 생성됨  
      - Pod의 DNS 주소: pod-ip-addr.namespace.pod.cluster.local //curl 10-36-0-1.default.pod.cluster.local  
ClusterIP: None // headless 서비스가 됨  
4) kube-proxy  
      - kubernetes service의 backend 구현  
      - endpoint 연결을 위한 iptables 구성  
      - nodePort로의 접근과 Pod 연결을 구현(iptables 구성)  
iptables -t nat -S | grep 80  
# 7.Ingress(API)  
HTTP나 HTTPS를 통해 클러스터 내부의 서비스를 외부로 노출  
- Service에 외부 URL을 제공
- 트래픽을 로드밸런싱  
- SSL 인증서 처리  
- Virtual hosting을 지정  
1) Ingress Controller 설치(쿠버네티스 홈페이지)  
# 8. 레이블(label)과 애너테이션(Annotation)
1) 레이블(label) : Node를 포함하여 pod, deployment 등 모든 리소스에 할당  
　　　　　　　　 리소스의 특성을 분류하고, Selector를 이용해서 선택  
　　　　　　　　 Key-value 한쌍으로 적용  
kubectl get pods --show-labels // label 목록  
kubectl get pods --selector rel=beta // 특정 label 목록  
kubelctl get nodes -L disk,gpu // 특정 label 확인  
kubectl label pod <pod명> name=test rel=beta --overwrite // label수정 및 적용  
kubectl label pod <pod명> run- // label 삭제  
Selector : yaml 파일에 옵션 넣어서 특정 label들 명령하기  
2) 워커 노드에 레이블 설정  
ex) disk=ssd, GPU=true  
kubectl label nodes <노드 이름> <레이블 키>=<레이블 값>  
yaml파일에는 nodeSelector: key: value // 특정 label값이 있는 node에 실행  
3) 애너테이션(Annotation) : label과 동일하게 key-value를 통해 리소스의 특성을 기록  
　　　　　　　　kubernetes에게 특정 정보 전달할 용도로 사용 // ex) Deployment의 rolling update 정보 기록  
　　　　　　　　관리를 위해 필요한 정보를 기록할 용도로 사용 // 릴리즈, 로깅, 모니터링에 필요한 정보들을 기록  
annotations:  
　builder: "JIHOON LEE"  
　builddate: 20220309  
　imageregistry: "https://hub.docker.com/"  
4) Canary Deployment(카나리 배포)  
포드를 배포(업데이트)하는 방법  
      (1)블루 그린 업데이트 : 블루(old) 다운하고 그린(new) 업하는 방법. 서비스 중단 있음  
      (2)카나리 업데이트 : 기존 버전을 유지한 채로 새로운 버전 추가하여 같이 동작하면서 변경   
      (3)롤링 업데이트 : 서비스 운영중에 하나씩 버전업데이트  
# 9. ConfigMap  
ConfigMap : 컨테이너 구성 정보를 한곳에 모아서 관리  
1) ConfigMap 생성  
kubectl create configmap <CONFIG_NAME> --from-file=파일명 --from-literal-key=value
2) ConfigMap 적용  
env:  
-name:INTERVAL  
　valueFrom:  
　　configMapKeyRef:  
　　　name: config명  
　　　key:INTERVAL  
3) ConfigMap 전체를 적용하기  
envFrom:  
　-configMapRef:  
　 　name: config명  
4) ConfigMap 볼륨마운트로 적용 가능  

# 10. Secret  
secret : 컨테이너가 사용하는 password, auth token, ssh key와 같은 중요한 정보를 저장하고 민감한 구성정보를 base64로 인코딩해서 한 곳에 모아서 관리  
1) secret 만들기  
kubectl create secret <Available Commands> name [flags] [options]  
(1) Available Commands 옵션 3가지  
docker-registry : --docker-username=tiger    
generic : --from-literal=2 --from-file=./genid-web-config/   
tls : --cert=path 
2) secret 사용하기  
env:  
-name: INTERVAL  
　valueFrom:
　　secretKeyRef:  
　　　name: secret명  
　　　key: INTERVAL  
3) secret 데이터 용량 제한  
secret etcd에 암호화 하지 않은 텍스트로 저장되므로 secret value가 커지면 메모리 용량을 많이 사용하게 됨  
secret의 최대 크기는 1MB  

# 11. Multi-master(HA) 쿠버네티스 설치하기  
-kubeadm  
      쿠버네티스에서 공식 제공하는 클러스터 생성/관리 도구  
-control plane(master node) - load balancer 필요  
      워커 노드들의 상태를 관리하고 제어  
      Highly Available(HA) cluster 운영 
      API는 loadbalancer를 통해 worker node에 노출  
      최소 3개 중첩된 control plane을 구성(5,7개의 master nodes)  
-worker node  
      도커 플랫폼을 통해 컨테이너를 동작하며 실제 서비스 제공  
1) all system-runtime(Docker) Install  
2) controll plane, worker node - kubeadm 설치  
      - 설치 전 환경설정  
      - kubeadm, kubectl, kubelet 설치  
3) LB(Load Balancer) 구성  
4) kubeadm을 이용한 HA 클러스터 구성  
      - master1 : kubeadm init 명령으로 초기화 - LB 등록  
      - master2, master3을 master1에 join  
      - CNI(Container Network Interface) 설치  
      - worker node를 LB 통해 master와 join  
      - 설치된 시스템 확인  
master1에서 /etc/hosts 에 master1-3,worker1-2,LB 넣기  
3) LB 구성  
nginx 구성파일 만들어 LB 구성  
(1)mkdir /etc/nginx  
(2)/etc/nginx/nginx.conf
events { }  
stream{  
    upstream stream_backend {  
      least_conn;  
      master1 IP:6443;  
      master2 IP:6443;  
      master3 IP:6443;  
    }  
    server {  
      listen   6443;  
      proxy_pass  steram_backend;  
      proxy_timeout   300s;  
      proxy_connect_timeout   1s;  
    }
}
(3)docker run --name proxy -v /etc/nginx/nginx.conf:/etc/nginx/nginx.conf:ro --restart=always -p 6443:6443 -d nginx  
4) kubeadm을 이용한 HA 클러스터 구성  
(1)master1: kubeadm init 명령으로 초기화 - LB 등록  
master1: kubeadm init --control-plane-endpoint "lb.example.com:6443 --upload-certs  
(2)master2, master3을 master1에 join  
kubeadm join lb.example.com:6443 --token <token>  
kubectl get nodes  
(3)CNI(Container Network Interface) 설치 : master1  
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"  
kubectl get nodes  
(4)worker node를 LB통해 master와 join  
kubeadm join lb.example.com:6443 --token <token>  
