# Spring Boot Uygulamasını Kubernetes'te Helm ile En Ucuz Şekilde Deploy Etme Rehberi

## En Ekonomik Deployment Seçenekleri (liste halinde):

1. **Managed Kubernetes Hizmetleri (Ücretsiz Katman)**
   - **Google Kubernetes Engine (GKE)**: Aylık $0 - Ücretsiz katmanda 1 kontrol düzlemi
   - **Digital Ocean**: Ayda $10'dan başlayan küçük kümeler
   - **Linode Kubernetes**: Ayda $10'dan başlayan fiyatlar

2. **Self-Hosted Çözümler**
   - **K3s**: Hafif Kubernetes dağıtımı (düşük kaynak tüketimi)
   - **MicroK8s**: Tek düğümlü basit kurulum için ideal
   - **minikube**: Geliştirme ortamında test için

3. **Serverless Kubernetes**
   - **Knative**: Yalnızca kullanıldığında ölçeklenen ve ücretlendirilen altyapı

## Adım Adım En Ucuz Deployment Planı:

### 1. Uygulamanızı Optimize Edin
- JVM bellek ayarları: `-Xmx` değerini düşük tutun (ör. 256m-512m)
- Distroless veya Alpine tabanlı imajlar kullanın
- Multi-stage builds ile imaj boyutunu küçültün

```Dockerfile
FROM eclipse-temurin:17-jdk-alpine AS builder
WORKDIR /app
COPY . .
RUN ./mvnw package -DskipTests

FROM eclipse-temurin:17-jre-alpine
COPY --from=builder /app/target/*.jar app.jar
ENTRYPOINT ["java", "-Xmx256m", "-jar", "/app.jar"]
```

### 2. Helm Chart'ınızı Kaynak Verimli Hale Getirin

```yaml
resources:
  requests:
    memory: "256Mi"
    cpu: "100m"
  limits:
    memory: "512Mi"
    cpu: "500m"
```

### 3. En Uygun Maliyetli Deployment Seçeneği

**Seçenek A: Digital Ocean Kubernetes + Spot Instances**
- $10/ay temel küme ücreti
- Spot instance'lar düzenli fiyattan %60-70 daha ucuz
- Örnek komut:
  ```bash
  doctl kubernetes cluster create spring-cluster --size s-1vcpu-2gb --count 1 --region fra1 --tag prod --spot
  ```

**Seçenek B: Self-Hosted K3s (VPS üzerinde)**
- Hetzner Cloud: ~€5/ay (2GB RAM, 1 vCPU)
- Oracle Cloud Free Tier: Tamamen ücretsiz (4GB RAM, 2 vCPU)
- K3s kurulum:
  ```bash
  curl -sfL https://get.k3s.io | sh -
  ```

### 4. Yatay Pod Autoscaling (HPA) ile Kaynak Optimizasyonu

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: spring-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: spring-app
  minReplicas: 1
  maxReplicas: 3
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

### 5. Monitoring ve Kaynak İzleme için Hafif Alternatifler
- Prometheus + Grafana yerine **Metrics-Server** + **kube-prometheus-light**
- Log yönetimi için ElasticSearch yerine **Loki**

## Uygulamada İzlenecek Adımlar:

1. Uygulamanızı konteynerize edin (optimize edilmiş Dockerfile ile)
2. Helm chart oluşturun (resource limitleri ayarlı)
3. En ucuz K8s platformunu seçin ve kurun
4. Helm ile deploy edin:
   ```bash
   helm install spring-app ./spring-app-chart -f values-production.yaml
   ```
5. İzleme sistemi kurun ve kaynak kullanımını optimize edin

Daha spesifik yardıma ihtiyacınız var mı? (Ör: Mevcut Helm chart'ınızı optimize etmek, özel bir cloud provider için yapılandırma yapmak gibi)
