# mig-volume
PV, PVC를 이관 테스트 하기 위한 sample입니다.   

아래와 같이 테스트 하십시오.    
## 사전준비
- VM 4대를 준비합니다.   
  - As-Is cluster, As-Is NFS 
  - To-Be cluster, To-Be NFS
- [minikube설치](https://happycloud-lee.tistory.com/20?category=832243)를 참조하여 minikube설치  
- [ssh설정](https://happycloud-lee.tistory.com/145?category=893924)를 참고하여 ssh설정  
  - As-Is cluster: sydney, As-Is NFS: nfs-sydney
  - To-Be cluster: tokyo, To-Be NFS: nfs-tokyo
- [NFS설치](https://happycloud-lee.tistory.com/46?category=832247)를 참고하여NFS를 설치   
  - volume 디렉토리는 '/data'로 하십시오.  
- [Dynamic provisioning](https://happycloud-lee.tistory.com/178?category=832243)을 참고하여 구성   
- sample 다운로드   
  - ssh sydney  
  - mkdir work
  - cd work
  - git clone https://github.com/happykube/mig-volume.git
  - chmod +x ./kubens && cp ./kubens /usr/local/bin 
- As-Is에 PVC, PV생성    
  - kubectl apply -f product-pvc-delete.yaml 
  - kubectl get pvc 
- nginx Pod생성  
  - scp ./product/* nfs-sydney:/data/default-product-{pvc volume}/
  - kubectl apply -f product.yaml
  - web browser에서 http://{sydney node ip}:30000 로 테스트   

## 이관  
- mkdir -p ~/work/mig && cd ~/work/mig
- kubectl get pvc product -o yaml > pvc.yaml
- kubectl get pv {pv name} -o yaml > pv.yaml 
- cp pvc.yaml pvc-mig.yaml && cp pv.yaml pv-mig.yaml
- pvc-mig-sample.yaml과 pv-mig-sample.yaml을 참조하여pvc-mig.yaml과pv-mig.yaml을 수정
- cd ~/work
- ssh tokyo
- mkdir work && exit 
- scp -r ~/work tokyo:~/work
- scp ./product/* nfs-tokyo:/data/default-product-{pvc volume}/
- ssh tokyo && cd work
- vi work/pv-mig.yaml 에서 nfs서버의 ip를 nfs-tokyo의 ip로 변경 
- kubectl apply -f work/pvc-mig.yaml
- kubectl apply -f work/pv-mig.yaml
- kubectl get pvc : binding 확인 
- kubectl apply -f product.yaml 
- web browser에서 http://{tokyo node ip}:30000 로 테스트
