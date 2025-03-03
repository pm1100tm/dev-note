# expose vs ports

docker-compose 파일에서 expose와 ports는 모두 컨테이너의 포트를 호스트 머신에 노출시키는데
사용되는 옵션이다. 이 두 옵션 간의 몇 가지 중요한 차이점에 대해서 알아본다.

## 🧶 expose

- 컨테이너 내부의 포트를 다른 컨테이너에게 노출하는 데 사용된다.
- expose로 노출된 포트는 Docker 네트워크 내에서만 접근 가능하며, 호스트 머신에서는 직접 접근 X
- expose로 노출된 포트는 컨테이너 간 통신에 사용될 때 유용하다.

## 🧵 ports

- 컨테이너의 포트를 호스트 머신에 바인딩하여 외부에서 접근할 수 있도록 한다.
- 호스트 머신의 포트와 컨테이너의 포트를 매핑하여 호스트 머신의 특정 포트로 접근하면 해당 컨테이너의
  포트로 연결된다.
- 외부에서 컨테이너에 접근할 때 사용되며, 컨테이너의 외부와의 통신에 사용된다.

### ⚡️ expose 와 ports 의 차이점

- expose

  - 컨테이너 내부의 포트를 다른 컨테이너에게 노출
  - 컨테이너 간 통신에 사용

- ports
  - 컨테이너의 포트를 호스트 머신에 바인딩하여 외부에서 접근할 수 있도록 한다.
  - 외부에서 컨테이너에 접근할 때 사용된다.

### 예제

```docker-compose
version: '3.8'

services:
  db:
    container_name: postgres
    image: postgres:14.11-alpine
    expose:
      - '5432'
    networks:
      - my-netword
    ...

  app:
    image: node-local:0.1
    container_name: node-local
    ports:
      - '8000:80'
    depends_on: db
    networks:
      - my-netword
    ...

networks:
  my-netword:
    driver: bridge
```

서비스 db는 5432 포트를 다른 컨테이너에 노출하며, 외부(호스트 머신)에서의 접근은 불가능하다.
즉, app 과 같이, 같은 네트워크를 사용하는 컨테이너만 통신이 가능하다.

서비스 app은 호스트 8000 포트와 내부 컨테이너 포트 80을 매팅하여, 외부(호스트 머신) 8000 포트에서
들어오는 요청을 내부 컨테이너 80 포트와 연결한다.
