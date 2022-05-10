# portal-sidecar-summry

### cf-for-k8s variable
- 

### k8s networkpolicy
-

### istio(sidecar)
-

### portal client
- 기본 Sidecar를 배포할 시 portalclient가 미 생성 되있으므로 uaac로 uaa에 접근하여 portalclient를 세팅해야 한다.

```
SYSTEM_DOMAIN=
APP_DOMAIN=
UAA_ADMIN_ID=
UAA_ADMIN_SECRET

uaac target uaa.$SYSTEM_DOMAIN --skip-ssl-validation
uaac token client get $UAA_ADMIN_ID -s $UAA_ADMIN_SECRET

uaac client add portalclient -s clientsecret --redirect_uri "http://portal-web-user.$APP_DOMAIN, http://portal-web-user.$APP_DOMAIN/callback" \
--scope "cloud_controller_service_permissions.read , openid , cloud_controller.read , cloud_controller.write , cloud_controller.admin" \
--authorized_grant_types "authorization_code , client_credentials , refresh_token" \
--authorities="uaa.resource" \
--autoapprove="openid , cloud_controller_service_permissions.read"
```
