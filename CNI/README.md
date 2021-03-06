
# CNI 설치 가이드
* CNI 플러그인 중 하나인 Calico CNI를 사용하며, 3.13 이후 버전 사용 필요
* Calicoctl도 함께 설치 진행 필요
    * https://www.projectcalico.org/


## 구성 요소 및 버전
* calico/node ([calico/node:v3.13.4](https://hub.docker.com/layers/calico/node/v3.13.4/images/sha256-2656efc741e90750282ad89b9ec078588b98909e5cd0b8d1256f2059466e1717?context=explore))
* calico/pod2daemon-flexvol ([calico/pod2daemon-flexvol:v3.13.4](https://hub.docker.com/layers/calico/pod2daemon-flexvol/v3.13.4/images/sha256-3a12c023e964104ebf8af330bc74fa25831e961c871f8024bd6917c1357a57a6?context=explore))
* calico/cni ([calico/cni:v.3.13.4](https://hub.docker.com/layers/calico/cni/v3.13.4/images/sha256-20a74b0c29e57b7e0a3bfd4474e98d3968f3f0edb1f307c9c789b8ed339971db?context=explore))
* calico/kube-controllers ([calico/kube-controllers:v.3.13.4](https://hub.docker.com/layers/calico/kube-controllers/v3.13.4/images/sha256-49404c910b50bdd93003315d1774c18f445589b1059b24eae2ebaa056c565e8c?context=explore))
* calico/ctl ([calico/ctl:v.3.15.0](https://registry.hub.docker.com/layers/calico/ctl/v3.15.0/images/sha256-09a08c8ef2ef637aadb3d2cc46965b8ba73e0e4cf863c836ad114cc3292822aa?context=explore))

## Prerequisites

## 폐쇄망 설치 가이드
설치를 진행하기 전 아래의 과정을 통해 필요한 이미지 및 yaml 파일을 준비한다.
1. **폐쇄망에서 설치하는 경우** 사용하는 image repository에 CNI 설치 시 필요한 이미지를 push한다. 

    * 작업 디렉토리 생성 및 환경 설정
    ```bash
    $ mkdir -p ~/cni-install
    $ export CNI_HOME=~/cni-install
    $ export CNI_VERSION=v3.13.4
    $ cd $CNI_HOME
    ```

    * 외부 네트워크 통신이 가능한 환경에서 필요한 이미지를 다운받는다.
    ```bash
    $ sudo docker pull calico/node:${CNI_VERSION}
    $ sudo docker save calico/node:${CNI_VERSION} > calico-node_${CNI_VERSION}.tar
    $ sudo docker pull calico/pod2daemon-flexvol:${CNI_VERSION}
    $ sudo docker save calico/pod2daemon-flexvol:${CNI_VERSION} > calico-pod2daemon-flexvol_${CNI_VERSION}.tar
    $ sudo docker pull calico/cni:${CNI_VERSION}
    $ sudo docker save calico/cni:${CNI_VERSION} > calico-cni_${CNI_VERSION}.tar
    $ sudo docker pull calico/kube-controllers:${CNI_VERSION}
    $ sudo docker save calico/kube-controllers:${CNI_VERSION} > calico-kube-controllers_${CNI_VERSION}.tar
    $ sudo docker pull calico/ctl:v3.15.0
    $ sudo docker save calico/ctl:v3.15.0 > calico-ctl_v3.15.0.tar
    ```

    * calico yaml을 다운로드한다. (대역 설정을 위함)
    ```bash
    $ curl -O -I https://docs.projectcalico.org/v3.13/manifests/calico.yaml
    $ curl -O -I https://docs.projectcalico.org/manifests/calicoctl.yaml
    ```


2. 위의 과정에서 생성한 tar 파일들을 폐쇄망 환경으로 이동시킨 뒤 사용하려는 registry에 이미지를 push한다.
    ```bash
    $ sudo docker load < calico-node_${CNI_VERSION}.tar
    $ sudo docker load < calico-pod2daemon-flexvol_${CNI_VERSION}.tar
    $ sudo docker load < calico-cni_${CNI_VERSION}.tar
    $ sudo docker load < calico-kube-controllers_${CNI_VERSION}.tar
    $ sudo docker load < calico-ctl_v3.15.0.tar
    
    $ sudo docker tag calico/node:${CNI_VERSION} ${REGISTRY}/calico/node:${CNI_VERSION}
    $ sudo docker tag calico/calico/pod2daemon-flexvol:${CNI_VERSION} ${REGISTRY}/calico/pod2daemon-flexvol:${CNI_VERSION}
    $ sudo docker tag calico/cni:${CNI_VERSION} ${REGISTRY}/calico/cni:${CNI_VERSION}
    $ sudo docker tag calico/kube-controllers:${CNI_VERSION} ${REGISTRY}/calico/kube-controllers:${CNI_VERSION}
    $ sudo docker tag calico/ctl:v3.15.0 ${REGISTRY}/calico/ctl:v3.15.0
   
    $ sudo docker push ${REGISTRY}/calico/node:${CNI_VERSION}
    $ sudo docker push ${REGISTRY}/calico/pod2daemon-flexvol:${CNI_VERSION}
    $ sudo docker push ${REGISTRY}/calico/cni:${CNI_VERSION}
    $ sudo docker push ${REGISTRY}/calico/kube-controllers:${CNI_VERSION}
    $ sudo docker push ${REGISTRY}/calico/ctl:v3.15.0
    ```


## Install Steps
0. [calico.yaml 수정]
1. [calico 설치]
2. [calicoctl 설치]


## Step0. calico yaml 수정
* 목적 : `calico yaml에 이미지 registry, 버전 정보, pod 대역, IPIP모드 여부를 수정`
* 생성 순서 : 
    * 아래의 command를 수정하여 사용하고자 하는 image 버전 정보를 수정한다.
	```bash
            ////////$ sed -i 's/{kiali_version}/'${KIALI_VERSION}'/g' 2.kiali.yaml
	```
    * pod 대역과 IPIP 모드를 아래와 같이 수정한다. pod 대역은 kubernetes 설치할때 사용했던 kubeadm-config.yaml의 podSubnet 대역과 동일해야 한다. (다를 경우 문제 발생)
	```bash
            - name: CALICO_IPV4_IPPOOL_IPIP
            value: "Never"            
            - name: CALICO_IPV4POOL_CIDR
            value: "10.0.0.0/16" 
	```         

* 비고 :
    * `폐쇄망에서 설치를 진행하여 별도의 image registry를 사용하는 경우 registry 정보를 추가로 설정해준다.`
	```bash
	////////$ sed -i 's/quay.io\/kiali\/kiali/'${REGISTRY}'\/kiali\/kiali/g' 2.kiali.yaml
	```

## Step 1. calico 설치
* 목적 : `calico 설치`
* 생성 순서: calico.yaml 설치  `ex) kubectl apply -f calico.yaml`
* 비고 :
    * calico-kube-controllers-xxxxxxxxxx-xxxxx (1개의 pod)
    * calico-node-xxxxx (모든 노드에 pod)


## Step 2. calicoctl 설치
* 목적 : `calicoctl 설치`
* 생성 순서: calicoctl.yaml 설치  `ex) kubectl apply -f calicoctl.yaml`
* 비고 :
    * calicoctl (1개의 pod)
    * alias calicoctl="kubectl exec -i -n kube-system calicoctl /calicoctl -- "


## calico 장애 해결
* 목적 : `calico에서 calico-node pod이 안뜸`
* 해결 방법 : 
    * calicoctl get node 노드이름 -o yaml
        * bgp:ipv4Address 확인 (노드에 인터페이스 여러개 존재하는 경우 변경 필요)
    ```bash
    calicoctl replace -f -<< EOF
    apiVersion: projectcalico.org/v3
    kind: Node
    metadata:
      name: c1-1
    spec:
      bgp:
        ipv4Address: 172.22.8.106
    EOF
    ```



