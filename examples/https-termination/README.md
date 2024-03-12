# HTTPS 프록시 예제

이번 예제는 기본 [cafe-example](../cafe-example)에서 HTTPS 터미네이션을 추가하여 기본 HTTP 요청에 대한 HTTPS 리다이렉트를 포함 합니다. 본 예제를 통해 우리는 게이트웨이가 다른 네임스페이스의 Secret을 참조할 수 있도록 ReferenceGrant를 사용 합니다.


## 예제는 아래의 절차에 따라 진행을 합니다.

## 1. NGINX GatewayFabric 배포

1. 먼저 [설치 가이드](https://docs.nginx.com/nginx-gateway-fabric/installation/)를 참고하여 NGINX GatewayFabric을 클러스터에 배포 진행 합니다.

1. 향후 트래픽처리 확인을 위해 NGINX GatewwayFabric 게이트웨이에 대한 공인IP을 shell의 변수로 저장 합니다(DNS가 있다면 DNS에 등록을 해서 진행할 수 있습니다).:

   ```text
   GW_IP=XXX.YYY.ZZZ.III
   ```

1. NGINX Gateway Fabric 접속을 위한 TCP 포트 정보도 동일하게 Shell 변수로 저장을 해 둡니다:

   ```text
   GW_HTTP_PORT=<http port number>
   GW_HTTPS_PORT=<https port number>
   ```
   (예)
      ```text
   GW_HTTP_PORT=80
   GW_HTTPS_PORT=443
   ```

## 2. 테스트를 위한 카페 애플리케이션을 배포

1. 테스트를 위한 기본 카페 애플리케이션을 배포 합니다:

   ```shell
   kubectl apply -f cafe.yaml
   ```

1. `default` 네임스페이스에 배포된 카페 애플리케이션에 대한 배포 상태를 확인 합니다:

   ```shell
   kubectl -n default get pods
   ```
   (예: 아래와 같은 결과를 확인할 수 있습니다)
   ```text
   NAME                      READY   STATUS    RESTARTS   AGE
   coffee-6f4b79b975-2sb28   1/1     Running   0          12s
   tea-6fb46d899f-fm7zr      1/1     Running   0          12s
   ```

## 3. HTTPS 터미네이션 및 라우팅을 설정

1. `certificate`라는 네임스페이스를 생성하고 TLS 인증서 및 키를 통해 Secret을 생성 합니다.:

   ```shell
   kubectl apply -f certificate-ns-and-cafe-secret.yaml
   ```

   TLS 인증서 및 키를 통해 생성한 Secret으로 카페 애플리케이션에 대한 TLS 터미네이션을 수행 합니다.
   > **중요**: 본 예제에 사용한 TLS 인증서 및 키 값은 데모용으로 만들 셀프 인증서이기 때문에 데모용으로만 사용을 합니다. 실제 사용하는 TLS 인증서 및 키 값이 있으면 해당 인증서와 키를 사용하여 생성을 하면 됩니다.

1. ReferenceGrant의 생성:

   ```shell
   kubectl apply -f reference-grant.yaml
   ```

   이 ReferenceGrant는 `default` 네임스페이스의 게이트웨이가 `certificate` 네임스페이스의 `cafe-secret`이름의 Secret을 참조할 수 있도록 허용 합니다.
   

1. 게이트웨이 리소스의 생성:

   ```shell
   kubectl apply -f gateway.yaml
   ```

   이 [Gateway](./gateway.yaml)의 설정:
    - `http` HTTP 트래픽에 대한 리스너
    - `https` HTTPS 트래픽에 대한 리스너. 이 리스너에서 스텝 1에서 생성한 `cafe-secret` Secret을 이용하여 TLS 터미네이션을 수행 합니다.

1. HTTPRoute 리소스의 생성:

   ```shell
   kubectl apply -f cafe-routes.yaml
   ```

   HTTPS 터미네이션을 수행하기 위해 우리는 카페 애플리케이션의 `coffee` 그리고 `tea` [cafe-routes.yaml](./cafe-routes.yaml) HTTPRoute 설정 부분애 [`parentReference`](https://gateway-api.sigs.k8s.io/references/spec/#gateway.networking.k8s.io/v1.ParentReference) 필드를 이용하여 `https` 리스너로 연결을 설정 합니다:

   ```yaml
   parentRefs:
   - name: gateway
     sectionName: https
   ```

   그리고 HTTP 트래픽을 HTTPS로 리다이렉트 하기 위해 HTTP 리스너에 `cafe-tls-redirect` HTTPRoute의 [`HTTPRequestRedirectFilter`](https://gateway-api.sigs.k8s.io/references/spec/#gateway.networking.k8s.io/v1.HTTPRequestRedirectFilter) 설정을 사용 합니다:

   ```yaml
   parentRefs:
   - name: gateway
     sectionName: http
   ```

## 4. 애플리케이션 테스트

해당 애플리케이션에 엑세스하기 위해 우리는  `curl` 명령을 사용하여 요청을 `coffee`와 `tea` 서비스에 전달을 합니다. 첫번째, HTTP로 트래픽을 전달하여 HTTPS가 정상적으로 동작하는가를 확인 합니다. 이후, HTTPS로 직접 접속하여 TLS 터미네이션이 정상적으로 동작하는가를 다시 한번 더 확인을 합니다.

### 4.1 HTTPS 리다이렉트 테스트


NGINX가 정상적으로 HTTPS 리다이렉트를 전송하는지 확인하기 위해 우리는 `coffee`와 `tea`서비스에 HTTP 포트로 접속을 시도 합니다. 응답 해더 부분을 확인하기 위해 curl 명령에서 `--include` 옵션을 함께 사용을 합니다.

결과로 NGINX에서 응답 해더 부분에 `Location` 해더를 통해서 리다이렉트를 전송하는가를 확인할 수 있습니다.

coffee 서비스에 대한 리다이렉트 확인:

```shell
curl --resolve cafe.example.com:$GW_HTTP_PORT:$GW_IP http://cafe.example.com:$GW_HTTP_PORT/coffee --include
```

```text
HTTP/1.1 302 Moved Temporarily
...
Location: https://cafe.example.com/coffee
...
```

tea 서비스에 대한 리다이렉트 테스트:

```shell
curl --resolve cafe.example.com:$GW_HTTP_PORT:$GW_IP http://cafe.example.com:$GW_HTTP_PORT/tea --include
```

```text
HTTP/1.1 302 Moved Temporarily
...
Location: https://cafe.example.com/tea
...
```

### 4.2 coffee와 tea 서비스에 직접 엑세스

이제 우리는 정상적으로 HTTP에 대한 HTTPS 리다이렉트가 동작하는 것을 확인했으며, 직접 HTTPS 리스너를 통해서 coffee 및 tea 서비스에 대한 접속을 확인 합니다. 이 테스트 시 생성한 Secret은 사설 인증서 기반이기 때문에 curl 명령에서 `--insecure` 옵션을 사용하여 인증서 검증을 통과하도록 해야 정상적으로 테스트를 할 수 있습니다.

coffee 서비스에 엑세스:

```shell
curl --resolve cafe.example.com:$GW_HTTPS_PORT:$GW_IP https://cafe.example.com:$GW_HTTPS_PORT/coffee --insecure
```

```text
Server address: 10.12.0.18:80
Server name: coffee-7586895968-r26zn
```

tea 서비스에 엑세스:

```shell
curl --resolve cafe.example.com:$GW_HTTPS_PORT:$GW_IP https://cafe.example.com:$GW_HTTPS_PORT/tea --insecure
```

```text
Server address: 10.12.0.19:80
Server name: tea-7cd44fcb4d-xfw2x
```

### 4.3 ReferenceGrant 설정을 해제 후 시험

`certificate`Step 3에서 생성했던 `certificate` 네임스페이스의 `cafe-secret`에 대한 ReferenceGrant 설정을 제거 후 접속을 시도:

```shell
kubectl delete -f reference-grant.yaml
```

이제 다시 한번 HTTPS 리스너를 통한 접속을 시도 시 우리는 Connection Refused 에러를 확인할 수 있습니다:

```shell
curl --resolve cafe.example.com:$GW_HTTPS_PORT:$GW_IP https://cafe.example.com:$GW_HTTPS_PORT/coffee --insecure -vvv
```

```text
...
curl: (7) Failed to connect to cafe.example.com port 443 after 0 ms: Connection refused
```

그리고 `https` 게이트웨이의 상태 정보를 확인하면 reference가 허용되지 않았음을 직접 체크해볼 수 있습니다: 
```shell
 kubectl describe gateway gateway
```

```text
 Name:                    https
 Conditions:
   Last Transition Time:  2023-06-26T20:23:56Z
   Message:               Certificate ref to secret certificate/cafe-secret not permitted by any ReferenceGrant
   Observed Generation:   1
   Reason:                RefNotPermitted
   Status:                False
   Type:                  Accepted
   Last Transition Time:  2023-06-26T20:23:56Z
   Message:               Certificate ref to secret certificate/cafe-secret not permitted by any ReferenceGrant
   Observed Generation:   1
   Reason:                RefNotPermitted
   Status:                False
   Type:                  ResolvedRefs
   Last Transition Time:  2023-06-26T20:23:56Z
   Message:               Certificate ref to secret certificate/cafe-secret not permitted by any ReferenceGrant
   Observed Generation:   1
   Reason:                Invalid
   Status:                False
   Type:                  Programmed
```