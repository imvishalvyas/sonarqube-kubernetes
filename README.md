### Sonarqube deployment on Kubernetes GCP.

## 1. Create Postgress DB GCP Cloud SQL.

[Click here](https://console.cloud.google.com/sql) to create postgress db.


## 2. Generate base64 encoded password.
```
echo -n 'yourpasswpod' | base64
```
Copy the password and put it in the secret file.

## 3. Applly secret file.
```
kubectl apply -f secret.yaml
```

## 4. Create PVC storage.
Sonarkube store extension and data. So create 2 pvc.

```
kubectl apply -f data-pvc.yaml 
```

```
kubectl apply -f extension-pvc.yaml
```

## 5. Apply deployment file.
```
kubectl apply -f deployment.yaml
```


## 6. Expose it service to the load balancer.
```
kubectl apply -f service.yaml
```


