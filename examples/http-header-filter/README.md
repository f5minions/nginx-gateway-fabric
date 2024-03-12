# HTTP Header 필터 예제


이번 예제에서는 NGINX Gateway Fabric과 간단한 에코서버를 배포하고 트래픽을 에코 서버로 라우팅하는 설정 후 HTTPRoute 리소스의 `RequestHeaderModifier` 필터를 통해 요청의 해더를 수정하여 에코서버로 전달하는 내용에 대해서 테스트를 진행 합니다.

## 아래의 절차에 따라서 테스트를 진행

## 1. NGINX Gateway Fabric 배포

1. [설치가이드](https://docs.nginx.com/nginx-gateway-fabric/installation/)에 따라 NGINX Gateway Fabric을 배포 합니다.

1. 실제 애플리케이션 테스트를 위해 NGINX Gateway Fabric을 위한 공인 IP를 shell에 저장을 합니다:

   ```text
   GW_IP=XXX.YYY.ZZZ.III
   ```

1. NGINX Gateway Fabric의 서비스 접속을 위한 TCP 포트도 동일하게 shell 변수로 저장 합니다:

   ```text
   GW_PORT=<port number>
   ```

## 2. 테스트를 위한 카페 애플리케이션 배포
1. 아래의 명령을 통해 기본 카페 애플리케이션을 배포:

   ```shell
   kubectl apply -f headers.yaml
   ```

1. `default` 네임스페이스에 headers라는 기본 테스트 앱이 정상적으로 배포되었는가를 확인 합니다:

   ```shell
   kubectl -n default get pods
   ```

   ```text
   NAME                      READY   STATUS    RESTARTS   AGE
   headers-6f4b79b975-2sb28   1/1     Running   0          12s
   ```

## 3. 카페 애플리케이션을 위한 라우팅 생성

1. Gateway의 배포:

   ```shell
   kubectl apply -f gateway.yaml
   ```

1.  HTTPRoute 리소스의 생성:

   ```shell
   kubectl apply -f echo-route.yaml
   ```

## 4. 애플리케이션 테스트

To access the application, we will use `curl` to send requests to the `headers` Service, including sending headers with
our request.

애플리케이션 테스트를 위해 `curl` 명령을 사용하여 `headers` 라는 이름의 서비스로 전달할 때 HTTPRoute 라우터에 설정한 특정한 해더 값을 함께 전달합니다. 클라이언트가 요청한 Header에 대해서는 아래의 결과와 같이 출력하지만 `No1WebServer` 라는 해더는 클라이언트에서 요청 시 다른 값을 전달했지만 실제로 서버로 전달되는 값는 HTTPRoute에 설정된 값 뿐 입니다.

그리고 기본값으로 전달되는 `User-Agent` 해더는 동일하게 HTTPRoute에서 제거하였기 때문에 서버로 전달되지 않습니다. 

```shell
curl -s --resolve echo.example.com:$GW_PORT:$GW_IP http://echo.example.com:$GW_PORT/headers -H "What-is-NGF:NGINX New Project" -H "No1WebServer:Apache"
```

```text
Headers:
  header 'Accept-Encoding' is 'compress'
  header 'What-is-NFG' is 'NGINX-Gateway-Fabric'
  header 'No1WebServer' is 'NGINX'
  header 'Host' is 'echo.example.com:30922'
  header 'X-Forwarded-For' is '192.168.65.3'
  header 'Connection' is 'close'
  header 'Accept' is '*/*'
  header 'What-is-NGF' is 'NGINX New Project'
```
