## Container VS Virtual Machine

<img width="1406" height="642" alt="image" src="https://github.com/user-attachments/assets/4bb81529-df7c-4a0d-bd48-49d6e4fc1695" />

## Microservice

<img width="1284" height="541" alt="image" src="https://github.com/user-attachments/assets/ec3878cc-8a21-4154-a0c4-60b2604ae550" />

- 한개의 Container를 여러개의 Container로 쪼갬
- ex) 쇼핑몰 1개를 여러개의 서비스로 나누기 (로그인, 웹페이지..등)

## docker 

<img width="1124" height="592" alt="image" src="https://github.com/user-attachments/assets/58a5e8bc-b516-4e74-a528-ae64e83ff3b2" />

- Dockerfile : 포장 지시서
- Images : 포장이 완성된 완성품
- Container : 제품이 실제로 동작하는 실체
- Registry : 제품 보관 창고(마트,물류창고)
## K8s 
- 자동 배포 및 롤백
- 서비스 디스커버리 & 로드밸런싱
- 자동 스케일링
- 자동 복구
- 환경변수 관리

<img width="1017" height="431" alt="image" src="https://github.com/user-attachments/assets/69c9cbc1-9135-49f6-ae38-dd1764e165b8" />

- API Server : 모든 명령과 요청을 받는 진입점
- Scheduler : 새로 생성될 Pod를 어떤 Node에 배치할지 결정
- Controller Manager : 쿠버네티스 리소스의 상태를 관리자가 원하는 상태로 맞추는 제어 루프 실행
- Cloud Controller Manager : 클라우드 API와 연동하여 로드밸런서, 스토리지, 노드 정보 관리
- etcd : 쿠버네티스의 모든 상태와 설정을 저장한ㄴ 분산 Key-Value 저장소 

## K8s 아키텍처 

<img width="1021" height="410" alt="image" src="https://github.com/user-attachments/assets/98bd9865-dc0b-4463-a889-1d4e300279a1" />

- Kubelet : 노드 에이전트, API 서버로부터 지시를 받아 컨테이너 생성/삭제 관리
- kube-proxy : 서비스 네트워킹 담당, 로드밸런싱 구현
- Container Runtime : 컨테이너를 실행하는 런타임 
## Openshift 
- Openshift는 RedHat에서 운영중인 오케스트레이션 도구라고 할 수 있다.
- CLI환경인 K8S와는 달리 OpenShift는 GUI환경으로 Windwos를 사용하는 사용자들에게는 더 편하게 다가올 수 있다.
<img width="1895" height="869" alt="image" src="https://github.com/user-attachments/assets/e3b134ec-1b7b-4944-ac1b-ce251eb9b676" />


<img width="1870" height="863" alt="image" src="https://github.com/user-attachments/assets/c6491059-e098-4866-a4c5-68629e0fdb6e" />

<img width="1920" height="949" alt="image" src="https://github.com/user-attachments/assets/50b80491-ea61-46af-a2e5-fc9b7864f253" />
