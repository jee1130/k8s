# kubeadm, kubelet, kubectl
kubeadm : 클러스터를 부트스트랩하는 명령어  
kubelet : 클러스터의 모든 머신에서 실행되는 파드와 컨테이너 시작과 같은 작업을 수행하는 컴포넌트  
kubectl : 클러스터와 통신하기 위한 커맨드 라인 유틸리티  
##### !!주의 컨트롤 플레인 서버보다 kubelet 버전이 높으면 버전 차이 버그 발생!!
##### !!주의 컨트롤 플레인 버전과 kubeadm 버전 일치해야함!!
# CONTROLL-PLANE, NODE
#### CONTROLL-PLANE  
kube-apiserver : k8s API를 사용하도록 요청을 받고 요청이 유효한지 검사  
kuebe-controller-manager : 파드를 관찰하며 개수를 보장  
etcd : key-value 타입의 저장소  
kube-scheduler : 파드를 실행할 노드 선택    
#### NODE
kubelet : 모든 노드에서 실행되는 k8s 에이전트, 데몬 형태로 동작  
kube-proxy : k8s의 network 동작읠 관리, iptable rule을 구성
컨테이너 런타임 : 컨테이너를 실행하는 엔진(ex:docker,containerd,runc)
# namespace
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

# yaml  
yaml : key-value 타입, 들여쓰기는 Tab대신 Space Bar로 사용 

# api version  
$kubectl explain pod // pod 정보 확인  


