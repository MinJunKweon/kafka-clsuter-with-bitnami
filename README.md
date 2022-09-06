# 5분만에 Apache Kafka Cluster 만들어보기

## 준비물
- Docker
- Kubernetes (여기서는 Rancher를 이용하였다)
- Helm

## Getting Started
#### 1. 본 Git Repository를 Clone한다
```bash
$ git clone git@github.com:LOG-INFO/kafka-clsuter-with-bitnami.git
```

#### 2. Bitnami Helm Repository를 추가한다
```bash
$ helm repo add bitnami https://charts.bitnami.com/bitnami
```

#### 3. Bitnami kafka Helm Chart를 실행한다
```bash
# 기본 values.yaml이 있긴 하지만, 덮어쓰고 싶은 내용들은 kafka/values.yaml을 참조한다
# 설정값을 변경하고 싶다면 https://github.com/bitnami/charts/tree/master/bitnami/kafka/#installing-the-chart 를 참고한다
$ helm install kafka -f kafka/values.yaml bitnami/kafka
```

#### 4. Kubernetes Pods, Services가 잘 생성되었는지 확인한다
```bash
$ kubectl get pods                                                                                                                                               INT  rancher-desktop kube
NAME                             READY   STATUS    RESTARTS      AGE
kafka-0                          2/2     Running   6             1m
kafka-1                          2/2     Running   6             1m
kafka-2                          2/2     Running   6             1m
kafka-zookeeper-0                1/1     Running   1             1m
kafka-zookeeper-1                1/1     Running   1             1m
kafka-zookeeper-2                1/1     Running   1             1m
kafka-exporter-97b464596-c46zm   1/1     Running   6             1m

$ kubectl get services                                                                                                                                                  ok  rancher-desktop kube
NAME                       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
kafka                      ClusterIP   10.43.175.234   <none>        9092/TCP                     1m
kafka-headless             ClusterIP   None            <none>        9092/TCP,9093/TCP            1m
kafka-0-external           NodePort    10.43.5.144     <none>        9094:30001/TCP               1m
kafka-1-external           NodePort    10.43.83.102    <none>        9094:30002/TCP               1m
kafka-2-external           NodePort    10.43.73.183    <none>        9094:30003/TCP               1m
kafka-zookeeper            ClusterIP   10.43.201.237   <none>        2181/TCP,2888/TCP,3888/TCP   1m
kafka-zookeeper-headless   ClusterIP   None            <none>        2181/TCP,2888/TCP,3888/TCP   1m
kafka-metrics              ClusterIP   10.43.242.108   <none>        9308/TCP                     1m
kafka-jmx-metrics          ClusterIP   10.43.201.19    <none>        5556/TCP                     1m
```

#### 5. Kafka가 잘 동작하는지 확인한다
```bash
$ kafka-console-consumer.sh --bootstrap-server localhost:30001,localhost:30002,localhost:30003 --topic test --from-beginning --group test
```
(다른 bash 추가로 띄운 상태에서)
```bash
$ kafka-console-producer.sh --bootstrap-server localhost:30001,localhost:30002,localhost:30003 --topic test
```

#### 6. 모니터링 지표 수집을 위한 Prometheus, Grafana의 Deployment, Service를 생성한다
```bash
$ kubectl apply -f prometheus,grafana
```

#### 7. Prometheus, Grafana의 Pods, Services가 잘 생성되었는지 확인한다
```bash
kubectl get pods                                                                                                                                               INT  rancher-desktop kube
NAME                             READY   STATUS    RESTARTS      AGE
prometheus-776fd895bc-vg87g      1/1     Running   1             1m
grafana-9bd5bbd6b-x5khn          1/1     Running   1             1m

$ kubectl get services                                                                                                                                                 
NAME                       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
grafana                    NodePort    10.43.194.39    <none>        3000:31000/TCP               1m
prometheus                 NodePort    10.43.224.108   <none>        9090:31001/TCP               1m
```

#### 8. Prometheus, Grafana에 접속한다
- Prometheus: http://localhost:31001/
  - Query 확인
    - ![image](https://user-images.githubusercontent.com/29394651/188572246-c9cd1974-3c93-4c49-8699-22be41b50642.png)
  - Targets 확인
    - ![image](https://user-images.githubusercontent.com/29394651/188572790-32e7db75-ad35-45f4-a3d1-1599ff841d9b.png)
- Grafana: http://localhost:31000/
  - Grafana의 초기 계정은 `admin`/`admin`이다
  - 처음엔 Grafana에 아무 대시보드도 존재하지 않는다
  - ![image](https://user-images.githubusercontent.com/29394651/188572350-7e9c01e7-cd95-4b78-962b-d09b9ef12411.png)

#### 9. Grafana Dashboard 추가
- Kafka JMX Grafana Dashboard
  - https://grafana.com/grafana/dashboards/12483-kubernetes-kafka/
- Kafka Grafana Dashboard
  - 

#### 10. 카프카 클러스터를 종료한다
```bash
$ helm delete kafka
```

#### 11. Prometheus, Grafana도 종료한다
```bash
$ kubectl delete -f prometheus,grafana
```

## 참고
- https://bitnami.com/stack/kafka/helm
