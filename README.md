# Домашнее задание к занятию «Конфигурация приложений»

### Цель задания

В тестовой среде Kubernetes необходимо создать конфигурацию и продемонстрировать работу приложения.

------

### Чеклист готовности к домашнему заданию

1. Установленное K8s-решение (например, MicroK8s).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым GitHub-репозиторием.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/concepts/configuration/secret/) Secret.
2. [Описание](https://kubernetes.io/docs/concepts/configuration/configmap/) ConfigMap.
3. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

------

### Задание 1. Создать Deployment приложения и решить возникшую проблему с помощью ConfigMap. Добавить веб-страницу

1. Создать Deployment приложения, состоящего из контейнеров nginx и multitool.
2. Решить возникшую проблему с помощью ConfigMap.
3. Продемонстрировать, что pod стартовал и оба конейнера работают.
4. Сделать простую веб-страницу и подключить её к Nginx с помощью ConfigMap. Подключить Service и показать вывод curl или в браузере.
5. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

| Deployment |
| :---: |
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-configmap
  labels:
    app: deploy-configmap
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: deploy-configmap
  template:
    metadata:
      labels:
        app: deploy-configmap
      namespace: default
    spec:
      containers:
      - name: nginx
        image: nginx:1.25.4
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
        volumeMounts:
          - name: nginx-conf
            mountPath: /usr/share/nginx/html
      - name: multitool
        image: wbitt/network-multitool
        imagePullPolicy: IfNotPresent
        env:
          - name: HTTP_PORT  
            valueFrom:
              configMapKeyRef:
                name: configmap-nginx
                key: key1
      volumes:
      - name: nginx-conf
        configMap: 
            name: configmap-nginx
            items: 
            - key: 'index.html'
              path: 'index.html'
```

| ConfigMap |
| :---: |
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: configmap-nginx
  namespace: default
data: 
  key1: '8081'
  index.html: |
    <!DOCTYPE html>
    <html>
     <head>
      <title>Empty Site</title>
      <meta charset="utf-8">
      <meta name="viewport" content="width=device-width, initial-scale=1">
     </head>
     <body>
      <h1>Empty page</h1>
     </body>
    </html>
```

| Service |
| :---: |
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-configmap
  namespace: default
spec:
  ports:
    - name: nginx
      port: 9001
      nodePort: 30080
      targetPort: 80
    - name: multitool
      port: 9002
      targetPort: 8081
  selector:
    app: deploy-configmap
  type: NodePort
```

<p align="center">
    <img width="1200 height="600" src="/img/configmap-curl.png">
</p>

<p align="center">
    <img width="1200 height="600" src="/img/configmap-web.png">
</p>

------

### Задание 2. Создать приложение с вашей веб-страницей, доступной по HTTPS 

1. Создать Deployment приложения, состоящего из Nginx.
2. Создать собственную веб-страницу и подключить её как ConfigMap к приложению.
3. Выпустить самоподписной сертификат SSL. Создать Secret для использования сертификата.
4. Создать Ingress и необходимый Service, подключить к нему SSL в вид. Продемонстировать доступ к приложению по HTTPS. 
4. Предоставить манифесты, а также скриншоты или вывод необходимых команд.

| Secret |
| :---: |
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-configmap
  namespace: default
type: kubernetes.io/tls
data:
  tls.crt: | 
    LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURDVENDQWZHZ0F3SUJBZ0lVSzBxUUp3eE44
    NGpPL094NTBCSUZ5NG85Z2Qwd0RRWUpLb1pJaHZjTkFRRUwKQlFBd0ZERVNNQkFHQTFVRUF3d0pi
    RzlqWVd4b2IzTjBNQjRYRFRJME1ESXlNakl3TVRreE5Wb1hEVEkxTURJeQpNVEl3TVRreE5Wb3dG
    REVTTUJBR0ExVUVBd3dKYkc5allXeG9iM04wTUlJQklqQU5CZ2txaGtpRzl3MEJBUUVGCkFBT0NB
    UThBTUlJQkNnS0NBUUVBc2ZlSWE2Y29yMkVHcjh5ZGtsd2NQQWVIVTAwdnQycXgyK2tmVDlJZCtE
    M3MKZ2xHWUFpakpTamI2Y1VnQnhTYkhNMWtGaGxzSFpoVjdBOHVXb3BDOXNpbFhGTWRwUFpIUnF1
    dVFGWmdpMUwyZAo3UlZTcXdNNnFBWC92b1pZZGxvUm9xajg5YlhSN3ByU01Jd2lQQitEbVAzMzBy
    ZW9NOU5mbDFQdytlZFo2dkZ5CmpxZUVGZWc1eTAybHJvMFBqZVB3TjYwdm50OE1SZ3pXdlIxdHdG
    cHF4QXhITnFyb0JXd3gvSHV5Y0RyN2NEd2sKQTlXZ2N5YXFVMkVQc1RXMGl4OFUrVXpkaHFQYjBX
    dDAvNGJOTmNZb29JZFhseGt4dkt6M3V3eDBRYUkvY1I3YwpqQUJVUGZzRXZlSUU4TE5Vc2V0eUN3
    TTM0VHhLNkxCeDhzeFhnZzlWbHdJREFRQUJvMU13VVRBZEJnTlZIUTRFCkZnUVUyTEZCdTJ5R2Ja
    Z2RVVk5XV3dYOERodlZ6bTh3SHdZRFZSMGpCQmd3Rm9BVTJMRkJ1MnlHYlpnZFVWTlcKV3dYOERo
    dlZ6bTh3RHdZRFZSMFRBUUgvQkFVd0F3RUIvekFOQmdrcWhraUc5dzBCQVFzRkFBT0NBUUVBZFpz
    dwpia2xNcGEyT3pxeGZjZUYvMmFLbWdpL21kZzZESkYyVmZTSU1aNUtVMjJhUXhNVWV0aWZ5SlRF
    ZGV2dFBzcmdDCkR3KzhYdWJRcFJGUWh6amVyY0FIU0dqNjBNSHVWLzVlZXh5TG9WS3ZFa3BMZmxY
    MFV1L0hidW8vRlBwMkVPZ20KWHd5Yy9UV2pzVVo5aWlUSldpbitSd3AzM09NMlUxRFpKa2U1SFNU
    em15ZjdTRnQvcUlxTWJsMDFIU3hMbjVRVwpkNWpLaTJtMGRUUDNNeHRwZVZDODFUcUxQaXlSODNi
    SWthRE9iMHdWdHIrdmRHS1BWaUNYLzVBa20ydGxnQU8vCkwvb3BXaDdsU1RvL21uMkNpQUJlRVhP
    SzZOVzdkYWVOSnhvUC9KU0xEdWFuaEpGcE1aSkZNNUpsM3psM2ovc0kKdUNObEpKeEdZd3dmb3pt
    bGxnPT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
  tls.key: |
    LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tCk1JSUV2QUlCQURBTkJna3Foa2lHOXcwQkFRRUZB
    QVNDQktZd2dnU2lBZ0VBQW9JQkFRQ3g5NGhycHlpdllRYXYKekoyU1hCdzhCNGRUVFMrM2FySGI2
    UjlQMGgzNFBleUNVWmdDS01sS052cHhTQUhGSnNjeldRV0dXd2RtRlhzRAp5NWFpa0wyeUtWY1V4
    Mms5a2RHcTY1QVZtQ0xVdlozdEZWS3JBenFvQmYrK2hsaDJXaEdpcVB6MXRkSHVtdEl3CmpDSThI
    NE9ZL2ZmU3Q2Z3owMStYVS9ENTUxbnE4WEtPcDRRVjZEbkxUYVd1alErTjQvQTNyUytlM3d4R0RO
    YTkKSFczQVdtckVERWMycXVnRmJESDhlN0p3T3Z0d1BDUUQxYUJ6SnFwVFlRK3hOYlNMSHhUNVRO
    MkdvOXZSYTNULwpoczAxeGlpZ2gxZVhHVEc4clBlN0RIUkJvajl4SHR5TUFGUTkrd1M5NGdUd3Mx
    U3g2M0lMQXpmaFBFcm9zSEh5CnpGZUNEMVdYQWdNQkFBRUNnZ0VBU2RmVXdoNXM4a0JISHdDK3pQ
    RHRRamM1Zm1ZRGk0NTQyQytscjJBVzBWOFkKV0laMGxVakpKTU1sTFlYY1BpcTE4dWRZTklSbTBJ
    UFBOQ2J3ak9tVDNHM3MxUkZjNkpBdHVFYmYxU1g0SmQrNwp2SmpoWVZZSXE1azVvWnRxNzBpMkVw
    RWR6UEl4ZGxqRktDR3RQdGN3cW5XT3M5OUNxcVprL294MDY2eUVFY2lFClZNdXlnZHE2NUdpcmJF
    RE1KcFdyWWhyVmdEWGpXa2xFVlhuTXFQSFc5OW9ZZ2xCMTQ5UUgxbHBiMXkzQnExL0YKeWVhOElq
    N3pyMEx5d3NjbUVVM0Z4dHdWMjEyTjQvZk5XQVhGSFJVdXZRQ2t3ajBZZHlSaGtBeHBwVXNyV24x
    NQo2NnpDZTh6MFZkUW1YUHRlcnFoWFhNTWkzMXFWZ1NQY29wemhZdjAyQVFLQmdRRDNoNHdlaVgz
    dEZWdzdNNFFJClZ2T09jMHNURnA4QVp0aVJ0dUc2MEZ5aHVZL1cxa1pIbFJpTFdTTzBGR3pJQUJv
    QVJSWmJIZ1FyenU0RGhsdFYKbmRBMnRzWTFXMHdtbUhvRUk3aDdaeUNYZDhmRGY1WXQvVkJ6R1lt
    ZmtUQlVBd3h3a1pmSTZuVzVVL3lmc3dmRgpTMkVpakFpU3NaVGVmNVhKbWt3WTNmNTJsd0tCZ1FD
    NERwZE5xTldkUnFsNzFPa1NsNzhzbTNuL3U5cm5pbmg5CkNDSTBQbVc4MHdIM1puNW1uVGJmK2V5
    VjFsVzlDTmlkQUw2K1NVV2x2b25VWkJMRFpCUTNrUG5KSWpBcWVLSU0KVHExQmNsMjRPZTlnWms5
    VllwWUYwVmhEUXZxRGIwVG4vV1ZMZTc1T1BXMmxLK0UrTmx1MXlpTWpuVFBkbS81bwpZWm1Bd2dq
    NUFRS0JnQmhtS0EycWg5c2l5K0NhQjEyN0ZHN3FObkEvUHBVUGpqRnUwWGxVcUl6WWViRTNsZDNn
    CmVIYmo1bjBOdGx0UWh6K1hqOGlUZ04zQW0vMkU5T1BQbG9LT0thT0F5RlRWbXRGbG8vMm1BTFJ0
    ZmlkcklDYVEKWGFtNnpySUg0YmVtUlVlalVrN2ZyWk1ERUZlOWtmcUVuNktFSXlReWxQWUpwWDRs
    MDNKd0QzRXBBb0dBUUJEMApJWmdQSXZ1aHF2VGxYQTl0Ly96dWJsSFpWSmNpY1lNUFJOZ2pXYUtw
    SUpDUWx1OUtWcFFNQWV2bFZETnNFdHBiCmlxaStrWDdOUXh6Q1d6ak93TGk1K1lUbzl4K2VhR0pL
    ZEdsMnJkV2N1UlZqci9qczk0RnpFNWFRMUljNm1QWGUKK0hOT1ByV3JJTDh4WEJKWHdlTm1iOU1j
    WnBzUjV3dHgveHVMUEFFQ2dZQmlmVzdnTVZ0Yno1c1h2NnJiTVdsdQo0WDFUemFhL1BseUY5ZlYv
    cVMxeWRhenF1YlRnTEtLV1E5UGwvWXpTbmxjSEd0d29kdEpBSnNOTFBYK2dPVlYyClVIRTlNZllT
    MXljU1luUG1naWFtKzJOeXRVUHpoVVlOYzB3VkNPb2l2SmE4azNYelBCOVQxaUlOenZoUzg4RHkK
    UHRBd2lrcU55cW9pV3k5UXhwVytLdz09Ci0tLS0tRU5EIFBSSVZBVEUgS0VZLS0tLS0K
```

| Ingress |
| :---: |
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-configmap
  namespace: default
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: "nginx"
  rules:
  - host: localhost
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: svc-configmap
            port:
              name: nginx
      - path: /api
        pathType: Exact
        backend:
          service:
            name: svc-configmap
            port:
              name: nginx
  tls:
    - hosts:
        - localhost
      secretName: secret-configmap
```

<p align="center">
    <img width="1200 height="600" src="/img/configmap-curl-tls.png">
</p>

<p align="center">
    <img width="1200 height="600" src="/img/configmap-web-tls.png">
</p>

------

### Правила приёма работы

1. Домашняя работа оформляется в своём GitHub-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl`, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

------
