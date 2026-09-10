# 006. Java 에서 Local IP 와 NAT IP validation 그리고 Testcode

✔️ Local IP는 사설 네트워크 범위(10.x.x.x, 172.16.x.x \~ 172.31.x.x, 192.168.x.x)

✔️ NAT IP는 사설 IP를 제외한 모든 IPv4 주소

✔️ Java & Spring Boot에서 정규식 검증 적용 가능

## Validation code

```java
import java.util.regex.Pattern;

public class IPValidator {

    // Local IP 정규식
    private static final String LOCAL_IP_REGEX =
            "^(10\\.\\d{1,3}\\.\\d{1,3}\\.\\d{1,3})$|" +  // 10.x.x.x
            "^(172\\.(1[6-9]|2[0-9]|3[0-1])\\.\\d{1,3}\\.\\d{1,3})$|" + // 172.16.x.x ~ 172.31.x.x
            "^(192\\.168\\.\\d{1,3}\\.\\d{1,3})$";  // 192.168.x.x

    // NAT IP (공인 IP) 정규식 (사설 IP, 루프백 제외)
    private static final String PUBLIC_IP_REGEX =
            "^(?!10\\.\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}$)" +
            "(?!172\\.(1[6-9]|2[0-9]|3[0-1])\\.\\d{1,3}\\.\\d{1,3}$)" +
            "(?!192\\.168\\.\\d{1,3}\\.\\d{1,3}$)" +
            "(?!127\\.\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}$)" +
            "(\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}\\.\\d{1,3})$";

    public static boolean isLocalIp(String ip) {
        return Pattern.matches(LOCAL_IP_REGEX, ip);
    }

    public static boolean isPublicIp(String ip) {
        return Pattern.matches(PUBLIC_IP_REGEX, ip);
    }
}
```

## Test code

```groovy
import spock.lang.Specification
import spock.lang.Unroll

class IPValidatorTest extends  Specification {

    @Unroll // 각 테스트 케이스를 개별 실행하여 명확한 결과 확인.
    def "Local IP 검증"() {
        expect:
        IPValidator.isLocalIp(ip) == expected

        where:
        ip               || expected
        "10.0.0.1"       || true   // ✅ A 클래스 사설 IP
        "10.255.255.255" || true
        "172.16.0.1"     || true   // ✅ B 클래스 사설 IP
        "172.31.255.255" || true
        "192.168.0.1"    || true   // ✅ C 클래스 사설 IP
        "192.168.255.255"|| true
        "192.167.1.1"    || false  // ❌ 공인 IP
        "172.15.255.255" || false  // ❌ 172.16~172.31 범위를 벗어남
        "11.0.0.1"       || false  // ❌ 공인 IP
        "8.8.8.8"        || false  // ❌ 공인 IP (Google DNS)
    }

    @Unroll
    def "Public IP 검증"() {
        expect:
        IPValidator.isPublicIp(ip) == expected

        where:
        ip               || expected
        "8.8.8.8"        || true   // ✅ 공인 IP (Google DNS)
        "52.15.192.200"  || true   // ✅ AWS 공인 IP
        "203.0.113.1"    || true   // ✅ 공인 IP
        "192.168.1.1"    || false  // ❌ 사설 IP
        "10.0.0.1"       || false  // ❌ 사설 IP
        "172.16.0.1"     || false  // ❌ 사설 IP
        "127.0.0.1"      || false  // ❌ 루프백 (공인 IP 아님)
    }
}
```
