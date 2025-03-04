# 🚀 Local IP

Local IP는 내부 네트워크(Private Network) 에서만 사용되는 IP 주소를 의미합니다.
이 IP는 인터넷에서 직접 접근할 수 없으며, 같은 네트워크 내에서만 통신할 수 있습니다.

- 내부 네트워크에서만 사용 -> 외부 인터넷에서는 보이지 않음
- 공유기, 사설 네트워크, 기업 내부망에서 사용
- NAT(Network Address Translation) 을 통해 인터넷과 통신 가능

## 🖥️ Local IP 주소 대역

RFC1981 표준에 따라 Local IP(사설 IP) 대역이 저장되어있습니다.

> [What is an RFC1918 Address](https://netbeez.net/blog/rfc1918/)

### IP 대역 주소 범위 사용 예

✅ A 클래스: 10.0.0.0 ~ 10.255.255.255 (대기업, 클라우드 네트워크)

✅ B 클래스: 172.16.0.0 ~ 172.31.255.255	(기업 네트워크)

✅ C 클래스: 192.168.0.0 ~ 192.168.255.255	(가정용 공유기, 소규모 네트워크)

> 📌 이 IP 주소들은 인터넷에서 직접 사용할 수 없으며, 공유기나 NAT를 통해 변환 후 인터넷과
통신합니다.

### ✅ Local IP 사용 사례**

1. **가정용 공유기**

- 공유기에 연결된 모든 기기(PC, 스마트폰)는 192.168.x.x 대역의 Local IP를 사용.
- 공유기가 NAT를 통해 인터넷과 연결.

2. **기업 네트워크**

- 내부 서버, 직원 컴퓨터는 10.x.x.x 또는 172.16.x.x 대역의 Local IP를 사용.
- 인터넷 사용 시 NAT를 통해 공인 IP로 변환.

3. **클라우드 환경 (AWS, GCP, Azure)**

-	VPC(가상 네트워크) 내부에서 **EC2 인스턴스는 Local IP를 사용** (10.0.0.x).
-	외부와 통신할 때 NAT Gateway 또는 Elastic IP 사용.

### 🚀 Local IP 정리

✔️ Local IP는 내부 네트워크에서만 사용되는 사설 IP 주소

✔️ 인터넷에서는 직접 접근할 수 없으며, NAT를 통해 변환해야 인터넷과 연결됨

✔️ IP 대역은 10.x.x.x, 172.16.x.x, 192.168.x.x로 지정됨

✔️ 가정, 기업 네트워크, 클라우드 환경에서 필수적으로 사용됨

### Local IP 확인 방법

🔹 **방법 1: ifconfig 명령어 사용**

```shell
ifconfig | grep "inet "
# 출력 예시: inet 192.168.1.10 netmask 0xffffff00 broadcast 192.168.1.255
```

🔹 **방법 2: ipconfig getifaddr en0**

✔️ en0은 Wi-Fi 인터페이스, en1 또는 en2는 유선 이더넷 인터페이스일 수 있음.

```shell
ipconfig getifaddr en0
# 출력 예시: 192.168.1.10
```

### Java 에서 Local IP 취득 방법

```java
import java.net.InetAddress;
import java.net.NetworkInterface;
import java.net.SocketException;
import java.util.Enumeration;

public class NetworkUtils {

    public static String getLocalIp() {
        try {
            Enumeration<NetworkInterface> interfaces = NetworkInterface.getNetworkInterfaces();
            while (interfaces.hasMoreElements()) {
                NetworkInterface networkInterface = interfaces.nextElement();
                Enumeration<InetAddress> addresses = networkInterface.getInetAddresses();
                while (addresses.hasMoreElements()) {
                    InetAddress inetAddress = addresses.nextElement();
                    if (!inetAddress.isLoopbackAddress() && inetAddress.isSiteLocalAddress()) {
                        return inetAddress.getHostAddress();  // ✅ 사설 IP 반환 (예: 192.168.x.x)
                    }
                }
            }
        } catch (SocketException e) {
            e.printStackTrace();
        }
        return "Unknown";
    }

    public static void main(String[] args) {
        System.out.println("Local IP: " + getLocalIp());
    }
}
```
