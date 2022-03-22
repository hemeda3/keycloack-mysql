# keycloack-mysql

username: ```admin```
password: ```admin```
normal user of SudaCoin realm:
username: ```user1```
password: ```password```
enable SSL

root@emulator1:~/keycloack-mysql# ```docker exec -it keycloak2 bash```

* bash-4.4$ ```cd opt/jboss/keycloak/bin/```
* bash-4.4$ ```./kcadm.sh config credentials --server http://localhost:8080/auth --realm master --user admin```
* bash-4.4$ ```./kcadm.sh update realms/master -s sslRequired=NONE```
* bash-4.4$ ```./kcadm.sh update realms/SudaCoinRealm -s sslRequired=NONE```



apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-echo
  annotations:
    cert-manager.io/cluster-issuer: keycloack-mapledoum-issuer
    #https://docs.giantswarm.io/guides/advanced-ingress-configuration/#ssl-passthrough
    nginx.ingress.kubernetes.io/ssl-passthrough: "true"
    #By default NGINX uses plain HTTP to reach the services. Adding the annotation
    #nginx.ingress.kubernetes.io/secure-backends: "true" in the Ingress rule changes the protocol to HTTPS.
    nginx.ingress.kubernetes.io/secure-backends: "true"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/ssl-redirect: "true"

spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - keycloak.mapledoum.click
      secretName: keycloak.mapledoum.click
  rules:
  - host: "keycloak.mapledoum.click"
    http:
      paths:
        - pathType: Prefix
          path: /
          backend:
            service:
              name: keycloack
              port:
                number: 8443



---
apiVersion: v1
kind: Service
metadata:
  name: keycloack
  labels:
    app: keycloack
    name: keycloack
spec:

  selector:
    app: keycloack
  ports:

    - port:       8443
      protocol:   TCP
      targetPort: 8443
      name:       https2

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: keycloack
spec:
  replicas: 1
  selector:
    matchLabels:
      app: keycloack
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: keycloack
    spec:
      containers:
        - name: keycloack
          image: jboss/keycloak:16.1.1
          imagePullPolicy: IfNotPresent
          env:
            - name: DB_VENDOR
              value: mysql
            - name: DB_DATABASE
              value: :(
            - name: DB_USER
              value: :(
            - name: DB_PASSWORD
              value: :( 
            - name: DB_ADDR
              value: :(
            - name: DB_PORT
              value: "3306"
            - name: JDBC_PARAMS
              value: enabledTLSProtocols=TLSv1.2
            - name: KEYCLOAK_USER
              value: admin
            - name: KEYCLOAK_PASSWORD
              value: :(
            - name: KEYCLOAK_HTTPS_PORT
              value: "8443"
            - name: KEYCLOAK_ALWAYS_HTTPS
              value: "true"
          volumeMounts:
            - mountPath: "/etc/x509/https"
              name: "keycloak-mapledoum-click"
              readOnly: true
          ports:

            - containerPort: 8443

      volumes:
        - name: "keycloak-mapledoum-click"
          secret:
            secretName: "keycloak.mapledoum.click"
