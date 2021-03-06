
# Console 설치 가이드

## 구성 요소
* hypercloud-console ([tmaxcloudck/hypercloud-console](https://hub.docker.com/r/tmaxcloudck/hypercloud-console/tags))

## Prerequisites
* Kubernetes, HyperCloud4 Operator, Grafana, Istio(Kiali, Jaeger), Prometheus가 설치되어 있어야 합니다.

## Install Steps
1. [Step 1. Infra 세팅](#step-1-infra-세팅)
2. [Step 2. Secret (TLS) 생성](#step-2-secret-tls-생성)
3. [Step 3. Service (Load Balancer) 생성](#step-3-service-load-balancer-생성)
4. [Step 4. Deployment (with Pod Template) 생성](#step-4-deployment-with-pod-template-생성)
5. [Step 5. 동작 확인](#step-5-동작-확인)

## Step 1. Infra 세팅
* 목적 : console에서 사용할 Namespace, ResourceQuota, ServiceAccount, ClusterRole, ClusterRoleBinding을 생성한다.
* 순서 : 
    * 작업 폴더에 [1.initialization.yaml](https://raw.githubusercontent.com/tmax-cloud/hypercloud-console/hc-dev/install-yaml/1.initialization.yaml) 파일을 생성하고, `@@NAME_NS@@`들을 모두 원하는 문자열로 교체합니다.
	    * 이 과정에서 `@@NAME_NS@@` 대신 기입하는 문자열은 console이 설치될 namespace의 이름이 됩니다.
    * `kubectl create -f 1.initialization.yaml` 을 실행합니다.

## Step 2. Secret (TLS) 생성
* 목적 : console이 https를 지원하게 한다.
* 순서 : 
    * 작업 폴더 하위의 tls 폴더에 crt, key 파일을 준비합니다.
	    * 발급받은 인증서가 없는 경우
		    * 작업 폴더 하위에 tls 폴더를 생성하고 진입한 후, 다음을 한 줄씩 실행합니다.
			    * `openssl genrsa -out tls.key 2048`
			    * `openssl req -new -key tls.key -out tls.csr`
			    * `openssl x509 -req -days 3650 -in tls.csr -signkey tls.key -out tls.crt`
	    * 발급받은 인증서가 있는 경우
		    * 발급받은 인증서에 암호가 걸려있는 경우, `openssl rsa -in xxxxxx.key(원본파일명) -out tls.key` 을 실행하고 암호를 해제합니다.
		    * 루트 인증서 및 체인 인증서를 적용하려는 경우, crt 파일 내용 아래에 체인 인증서 내용을, 그 아래에 루트 인증서 내용을 차례로 이어붙인, 새로운 crt 파일을 만듭니다.
    * 작업 폴더로 이동하고, `kubectl create secret tls console-https-secret --cert=./tls/tls.crt --key=./tls/tls.key -n console-system(Step 1에서 @@NAME_NS@@ 대신 기입한 이름)` 을 실행합니다.

## Step 3. Service (Load Balancer) 생성
* 목적 : 브라우저를 통해 console에 접속할 수 있게 한다.
* 순서 : 
    * 작업 폴더에 [2.svc-lb.yaml](https://raw.githubusercontent.com/tmax-cloud/hypercloud-console/hc-dev/install-yaml/2.svc-lb.yaml) 파일을 생성하고, `@@NAME_NS@@`를 원하는 문자열로 교체합니다.
	    * `@@NAME_NS@@` 대신 기입하는 문자열은 Step 1에서와 같아야 합니다.
    * `kubectl create -f 2.svc-lb.yaml` 을 실행합니다.

## Step 4. Deployment (with Pod Template) 생성
* 목적 : console 웹서버를 호스팅할 pod를 생성한다.
* 순서 : 
    * 작업 폴더에 [3.deployment-pod.yaml](https://raw.githubusercontent.com/tmax-cloud/hypercloud-console/hc-dev/install-yaml/3.deployment-pod.yaml) 파일을 생성하고, 다음의 문자열들을 교체해줍니다.
    
    | 문자열 | 상세내용 | 형식예시 |
    | ---- | ---- | ---- |
    | `@@NAME_NS@@` | namespace의 이름 (Step 1에서와 같게) | `hypercloud-console` |
    | `@@HC4@@` | `kubectl get svc -n hypercloud4-system hypercloud4-operator-service` 에서 CLUSTER-IP와 PORT(S) 확인하여 입력 (포트는 `:` 왼쪽 값 사용) | `10.x.x.x:28677` |
    | `@@PROM@@` | `kubectl get svc -n monitoring prometheus-k8s` 에서 CLUSTER-IP와 PORT(S) 확인하여 입력 (포트는 `:` 왼쪽 값 사용) | `10.x.x.x:9090` |
    | `@@GRAFANA@@` | `kubectl get svc -n monitoring grafana` 에서 CLUSTER-IP와 PORT(S) 확인하여 입력 (포트는 `:` 왼쪽 값 사용) | `10.x.x.x:3000` |
    | `@@KIALI@@` | `kubectl get svc -n istio-system kiali` 에서 CLUSTER-IP와 PORT(S) 확인하여 입력 (포트는 `:` 왼쪽 값 사용) | `10.x.x.x:20001` |
    | `@@JAEGER@@` | `kubectl get svc -n istio-system tracing` 에서 CLUSTER-IP와 PORT(S) 확인하여 입력 (포트는 `:` 왼쪽 값 사용) | `10.x.x.x:80` |
    | `@@HDC_FLAG@@` | HCDC 모드로 설치하려는 경우 `true` 입력 (아닌 경우 행 삭제) | `true` |
    | `@@PORTAL@@` | HCDC 모드로 설치하려는 경우 tmaxcloud portal 로그인 페이지 URL 입력 (아닌 경우 행 삭제) | `https://tmaxcloud.com/#!/sign-in` |
    | `@@VER@@` | hypercloud-console 이미지 태그 입력 | `1.1.x.x` |
    
    * `kubectl create -f 3.deployment-pod.yaml` 을 실행합니다.
* 비고
    * HCDC 모드로 설치할 경우 DNS 서버 세팅이 필요하고, console과 portal이 같은 도메인의 서브도메인을 사용해야 합니다. (포트는 둘 다 https 기본 포트인 443 사용)


## Step 5. 동작 확인
* 목적 : console이 정상 동작하는지 확인한다.
* 순서 : 
    * `kubectl get po -n console-system(Step 1에서 @@NAME_NS@@ 대신 기입한 이름)` 을 실행하여 pod가 running 중인지 확인합니다.
    * `kubectl get svc -n console-system(Step 1에서 @@NAME_NS@@ 대신 기입한 이름)` 을 실행하여 EXTERNAL-IP를 확인합니다.
    * `https://EXTERNAL-IP` 로 접속하여 동작을 확인합니다.
	    * 단, HCDC 모드에서 테스트하려 하는 경우에는 IP가 아니라 Domain Name을 통해 접속해야 합니다.
