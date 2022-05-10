# portal-sidecar-summry

### cf-for-k8s variable
- 편안한 Portal APP manifest 설정을 위한 manifest/sidecar-values.yml 의 변경이 필요 하다.
```
cf_admin_password: ******

cf_db:
  admin_password: ******
  
capi:
  database:
    password: ******
    
uaa:
  database:
    password: ******
  admin_client_secret: ******
  login_secret: ******

```

### k8s networkpolicy
- 현재는 networkpolicy를 전부 해제하고 테스트 중인데 이후 테스트는 networkpolicy를 유지한 채 테스트 진행해봐야함 --> 테스트 필요

### istio(sidecar)
- cf-workloads가 다른 k8s service와 통신을 하려면 k8s sidecar의 설정을 약간 변경해줘야 한다.
```
$ vi portal-sidecar.yml
---
apiVersion: networking.istio.io/v1alpha3
kind: Sidecar
metadata:
  name: portal-db
  namespace: cf-workloads
spec:
  workloadSelector:
    labels:
      cloudfoundry.org/org_name: system   # portal app 이 배포될 org name 
      cloudfoundry.org/space_name: dev    # portal app 이 배포될 space name
  egress:
  - hosts:
    - cf-db/*    # portal-app 이 cf-db와 통신 가능하게 설정
    - cf-workloads/* # portal-container-infra가 외부 VM 이고 service를 cf-workloads에 설정했을 경우
    - ......     # 필요 시 추가 설정
```

- 최종적으로는 Portal DB와 Storage가 POD로 작동될 예정 이지만 테스트 용으로 외부 infra와 통신하게 설정하였다.
```
$ vi storage-service.yml
apiVersion: v1
kind: Service
metadata:
  name: storage-service
  namespace: cf-workloads
spec:
  ports:
    - port: 15001
      targetPort: 15001
      name : keystone
    - port: 10008
      targetPort: 10008
      name : proxy
    - port: 13306
      targetPort: 13306
      name : mariadb
---
apiVersion: v1
kind: Endpoints
metadata:
  name: storage-service
  namespace: cf-workloads
subsets:
- addresses:
    - ip: 10.0.2.120
  ports:
    - port: 15001
      name : keystone
    - port: 10008
      name : proxy
    - port: 13306
      name0 : mariadb
      
## 설정 후 sidecar의 egress 변경 필요
```



### portal client
- 기본 Sidecar를 배포할 시 portalclient가 미 생성 되있으므로 uaac로 uaa에 접근하여 portalclient를 세팅해야 한다.

```
SYSTEM_DOMAIN=********
APP_DOMAIN=********
UAA_ADMIN_ID=********
UAA_ADMIN_SECRET********

uaac target uaa.$SYSTEM_DOMAIN --skip-ssl-validation
uaac token client get $UAA_ADMIN_ID -s $UAA_ADMIN_SECRET

uaac client add portalclient -s clientsecret --redirect_uri "http://portal-web-user.$APP_DOMAIN, http://portal-web-user.$APP_DOMAIN/callback" \
--scope "cloud_controller_service_permissions.read , openid , cloud_controller.read , cloud_controller.write , cloud_controller.admin" \
--authorized_grant_types "authorization_code , client_credentials , refresh_token" \
--authorities="uaa.resource" \
--autoapprove="openid , cloud_controller_service_permissions.read"
```
