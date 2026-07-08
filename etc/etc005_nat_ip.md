# 005. NAT IP란?

NAT IP(Network Address Translation IP) 는 프라이빗 네트워크(내부 네트워크)에서 퍼블릭 네트워크(인터넷)로 통신할 때 사용되는 IP 주소입니다.

## 📌 주요 특징

* **프라이빗 IP를 퍼블릭 IP로 변환**하여 인터넷과 통신 가능하게 함
* **공인 IP 주소를 절약**하기 위해 사용됨
* **내부 네트워크 보호** 기능을 제공하여 보안성을 높임

## ✅ NAT 동작 방식

NAT(Network Address Translation)는 **라우터 또는 방화벽에서 IP 주소를 변환하는 과정**입니다.

🔹 NAT IP 변환 방식

* **SNAT (Source NAT)** 내부 IP → 공인 IP 변경 (출발지 변환)
* **DNAT (Destination NAT)** 공인 IP → 내부 IP 변경 (목적지 변환)
* **PAT (Port Address Translation)** 여러 내부 IP를 하나의 공인 IP로 변환 (포트 기반)

## ✅ NAT IP의 종류

### 1️⃣ 공인 NAT IP (Public NAT IP)

* 내부 **프라이빗 IP**를 **공인 IP**로 변환하여 인터넷과 통신 가능하게 함
* 클라우드 서비스 (AWS, GCP, Azure)에서 사용됨

### 사설 NAT IP (Private NAT IP)

* 내부 네트워크에서만 사용되는 **프라이빗 IP** (예: 192.168.0.1, 10.0.0.1)
* 내부 서버 간 통신을 위해 사용되며, 인터넷에는 직접 노출되지 않음

## ✅ NAT IP 사용 사례

### 🔹 1️⃣ 기업 네트워크

* 회사 내부에서 **모든 직원이 하나의 공인 IP를 사용**하여 인터넷에 접속
* **SNAT**을 사용하여 여러 사설 IP를 하나의 공인 IP로 변환

### 🔹 2️⃣ 클라우드 환경 (AWS, GCP, Azure)

* **퍼블릭 서브넷 → 공인 IP (인터넷 접속 가능)**
* **프라이빗 서브넷 → NAT 게이트웨이를 통해 인터넷 접속**

📌 예: AWS NAT Gateway

* 프라이빗 서브넷에 있는 서버가 인터넷과 통신할 때 **NAT Gateway**를 사용하여 공인 IP를 할당받음

### 🔹 3️⃣ 가정용 인터넷 공유기

* 집에서 여러 기기가 **공유기(NAT)를 통해 하나의 공인 IP로 인터넷을 사용**
* 공유기가 내부 IP(192.168.x.x)를 공인 IP로 변환

### ✅ NAT IP vs. Elastic IP

| 구분       | NAT IP            | Elastic IP             |
| -------- | ----------------- | ---------------------- |
| 설명       | 내부 IP를 공인 IP로 변환  | 고정된 공인 IP              |
| 사용 목적    | 프라이빗 네트워크의 인터넷 연결 | 특정 서버(EC2 등)에 고정 IP 제공 |
| AWS 사용 예 | NAT Gateway       | EC2, Load Balancer     |

### 🚀 정리

✔️ **NAT IP는 프라이빗 네트워크에서 인터넷과 통신할 때 사용됨**

✔️ **IP 변환 방식에는 SNAT, DNAT, PAT가 있음**

✔️ **기업 네트워크, 클라우드 환경, 가정용 공유기 등에서 널리 사용됨**

✔️ **AWS, GCP, Azure에서는 NAT Gateway를 사용하여 NAT IP를 제공**

### NAT IP 확인 방법

```shell
# AWS API로 확인
curl -s http://checkip.amazonaws.com

# OpenDNS API
dig +short myip.opendns.com @resolver1.opendns.com

# Google의 STUN 서버 사용
curl -s https://domains.google.com/checkip
```

### Java 에서 NAT IP 취득 방법

> Java에서는 공인 IP(NAT IP)는 직접 가져올 수 없으며, 외부 서비스를 통해 확인해야 합니다.

```java
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;

public class NetworkUtils {

    public static String getPublicIp() {
        try {
            URL url = new URL("http://checkip.amazonaws.com"); // ✅ AWS API 사용
            HttpURLConnection connection = (HttpURLConnection) url.openConnection();
            connection.setRequestMethod("GET");
            BufferedReader reader = new BufferedReader(new InputStreamReader(connection.getInputStream()));
            String publicIp = reader.readLine().trim();
            reader.close();
            return publicIp;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return "Unknown";
    }

    public static void main(String[] args) {
        System.out.println("Public (NAT) IP: " + getPublicIp());
    }
}
```
