# Interview

## Python

- [1] Python 언어의 주요 특성
- [2] self
- [3] list vs tuple
- [4] 리스트 컴프리헨션(list comprehension)
- [5] 제너레이터
- [6] 데코레이터
- [7] 파이썬의 내장 데이터 타입
- [8] shallow copy vs deep copy
- [9] Python 이 메모리를 관리하는 방법
- [10] lambda 함수
- [11] GIL(Global Interpreter Lock)
- [12] \_\_init\_\_ vs \_\_new\_\_
- [13] staticmethod vs classmethod
- [14] == vs is
- [15] \_\_slots\_\_
- [16] Context Manager

## Django

- [1] signals
- [2] 정적 파일 또는 미디어 파일들을 다루는 방법
- [3] Django VS FastAPI
- [4] select_related and prefetch_related
- [5] Django의 캐시 전략
- [6] annotate 과 F
- https://velog.io/@qlgks1/Django-Query-Expressions-F-Func-Aggregate-Value-Subquery-%EA%B7%B8%EB%A6%AC%EA%B3%A0-and-or-not

### 1. Python 언어의 주요 특성

- 인터프리터 언어: 런타임 시 코드를 한 줄씩 해석하여 실행한다.
- 동적 타이핑: 변수의 타입을 선언할 필요 없이 값이 할당될 때 자동으로 타입이 결정된다.
- 객체 지향 프로그래밍: 클래스와 객체를 사용한 코드 재사용이 가능하다.

### 2. self

- 클래스의 인스턴스를 나타내는 변수이다.
- 클래스 내의 메서드를 정의할 때 첫 번째 인자로 사용된다.
- 이를 통해, 클래스 내의 다른 멤버 변수나 메서드에 접근할 수 있다.

### 3. list vs tuple

- 변경 가능성
- 사용 목적
- 문법적 차이

리스트는 변경 가능(mutable)한 자료형으로 요소의 추가, 삭제, 수정이 가능하다.
튜플의 경우 변경 불가능(immutable)한 자료형으로, 한 번 생성된 후 수정할 수 없다.
리스트는 주로 데이터가 자주 변경될 때 사용하며, 튜플은 변경되지 않는 데이터를 저장할 때
사용된다.

### 4. 리스트 컴프리헨션(list comprehension)

- 간결하게 리스트를 생성할 수 있는 문법이며, 기존의 리스트를 기반으로 리스트를
  생성하거나 특정 조건을 만족하는 요소들을 필터링할 때 유용하다.
- for 문을 활용한 반복문보다 연산 속도가 빠르다.

### 5. 제너레이터

- 이터레이터의 일종으로 함수 내에서 yield 키워드를 사용하여 값을 하나씩 반환합니다.
  즉, yield 문을 사용하여 값의 시퀀스를 생성하는 함수입니다.
- 값이 필요할 때마다 계산을 수행하므로 메모리 효율적입니다.
- 단일 값을 반환하는 함수와 달리, 제너레이터는 여러 값을 산출할 수 있으며, 각 산출 사이 상태를 일시
  중지했다가 다시 시작할 수도 있습니다.

```python
def count_up_to(max_num: int) -> int:
  count = 1
  while count <= max_num:
    yield count
    count +=1


for n in count_up_to(5):
  print(n)


squares_gen = (x**2 for x in range(10))
print(next(squares_gen))
print(next(squares_gen))
```

### 6. 데코레이터

함수를 수정하지 않고 함수에 추가적인 기능을 부여할 때 사용한다. 다른 함수를 인자로 받아 새로운 함수를
반환하는 함수로 정의된다. 예를 들어, 함수의 실행 시간을 측정하는 데코레이터는 다음과 같이 작성한다.

```python
def deco_calculate_time(func):
    """
    데코레이터: 시간 측정
    """

    def wrapper(*args, **kwargs):
        t1 = time.time()
        ret = func(*args, **kwargs)
        t2 = time.time()
        execution_time = t2 - t1
        print(f"Func[{func.__name__}] execution time: {execution_time:.8f} seconds")

        return ret

    return wrapper
```

#### @functools.wraps(func)

functools 모듈에서 제공하는 @wraps 데코레이터는, 데코레이터로 사용되는 함수의 원래 속성을 보존
하도록 도와줍니다. 예를 들어, 데코레이터가 적용된 함수의 이름, 모듈, 문서화 문자열(docstring) 및
기타 중요한 속성을 원래 함수와 동일하게 유지할 수 있도록 해줍니다.

데코레이터를 사용하면 원래 함수의 메타데이터가 데코레이터로 매핑된 함수로 덮어씌워지기 때문에, 원래 함수의
중요한 정보가 사라질 수 있는데, @wraps는 이러한 문제를 방지합니다. 아래의 코드를 실행해보면
say_hello 함수의 메서드명과 문서의 정보가 wrapper 함수로 덮어씌워진 것을 볼 수 있다.

```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        """Wrapper function."""
        print("Something is happening before the function is called.")
        result = func(*args, **kwargs)
        print("Something is happening after the function is called.")
        return result

    return wrapper


@my_decorator
def say_hello():
    """This function says hello."""
    print("Hello!")


print(say_hello.__name__) # 출력: wrapper
print(say_hello.__doc__) # 출력: Wrapper function
```

### 7. 파이썬의 내장 데이터 타입

- Numbers: int(정수), float(실수), complex(복소수)
- Sequences: list, tuple, range
- Text: str
- Sets: set, frozenset
- Mappings: dict
- Booleans: bool
- Binary: bytes, bytearray, memoryview

### 8. shallow copy vs deep copy

#### shallow copy(얕은 복사)

새로운 개체를 만들되 원본의 참조값을 삽입합니다. 즉, 복사된 객체의 변경사항이 원본 객체에 반영됩니다.

```python
import copy

original = [[1, 2, 3], [4, 5, 6]]
shallow = copy.copy(original)
shallow[0][0] = 'X'
print(original)  # [['X', 2, 3], [4, 5, 6]]
```

#### deep copy(깊은 복사)

원본의 모든 값을 재귀적으로 복사하여, 복사된 객체는 원본과 완전히 독립적입니다.

```python
import copy
original = [[1, 2, 3], [4, 5, 6]]
deep = copy.deepcopy(original)
deep[0][0] = 'X'

print(original)  # [[1, 2, 3], [4, 5, 6]]
```

### 9. Python 이 메모리를 관리하는 방법

파이썬은 Private Heap, Memory Manager, GC(가비지 컬렉터)를 조합하여 메모리를 관리합니다.
자세한 설명은 다음과 같습니다.

#### Private Heap

파이썬 객체와 데이터 구조체는 Private Heap 에 저장됩니다. 이 힙 공간은 파이썬 메모리 관리자가
관리하며, 메모리 할당과 비할당을 담당합니다. 사용자는 이 Private Heap에 직접 접근할 수 없으며,
대신 파이썬 객체와 데이터 구조를 통해 이 힙과 상호작용합니다.

#### Memory Manager

파이썬 메모리 관리자는 파이썬 힙 관리를 포함해 동적 메모리 할당을 처리합니다.
객체에 대한 메모리 할당을 처리하고 더 이상 사용하지 않는 메모리를 회수합니다.

#### Object-specific Allocators

특정 유형의 객체(정수, 리스트, 딕셔너리 등)의 경우 파이썬은 이러한 객체 유형에 대해 메모리를 효율적
으로 관리하는 특수 할당자를 사용합니다.

#### Garbage Collection (GC)

파이썬의 GC는 더 이상 사용하지 않는 메모리를 자동으로 할당 해제하는데 도움이 됩니다.
참조 카운팅(객체의 참조 값 갯수가 0이 될 때)과 순환 GC 조합을 사용하여 메모리를 관리합니다.

sys 모듈을 사용하면 참조 카운팅 시스템과 상호 작용할 수 있습니다. 예를 들어 sys.getrefcount(object)는 주어진 객체에 대한 참조 카운트를 반환합니다.

#### Cyclic Garbage Collector

참조 카운팅만으로든는 참조 주기(2개 이상의 객체가 서로 참조하여 루프를 형성하는 경우)를 처리할 수
없습니다. 이 때 파이썬의 순환 GC가 등장합니다.

순환 GC는 프로그램 내 다른 참조를 통해서가 아니라 서로를 통해서만 도달할 수 있는 객체 그룹을
식별하여 이러한 주기를 끊고 메모리를 회수합니다.

gc 모듈은 가비지 컬렉터에 대한 인터페이스를 제공합니다.

```python
import gc

# Forcing garbage collection
gc.collect()

# Getting the number of unreachable objects found
print(gc.collect())
```

- 참조 카운팅

  - 각 객체에는 객체에 대한 새 참조가 만들어지면 참조 수가 증가하고 참조가 삭제되면 참조수가 감소합니다.
  - 참조 카운트가 0으로 떨어지면 객체는 할당 해제됩니다.

- 순환 가비지 컬렉션

  - 파이썬에는 두 개 이상의 객체가 서로 참조하여 참조 카운팅만으로는 해결할 수 없는 주기를 만드는
    순환 참조를 감지하고 정리하는 가비지 컬렉터가 있습니다.

### 10. lambda 함수

- 익명 함수이고 단일 문으로 표현하는 함수이며, 작은 일회성 인라인 함수 객체를 만드는데 사용합니다.
- 공식적으로 정의할 필요없어 코드를 간결하게 가독성있게 만들 수 있습니다.
- 그러나 복잡한 처리를 할 수 없고, 디버깅의 어려움과 재사용성이 떨어진다는 단점이 있습니다.

```python
add = lambda x, y: x + y
add(1, 2)
```

람다 함수는 수명이 짧은 연산에 자주 사용되기 때문에 GC가 더 빨리 이루어질 수 있어, 단기적으로 더
효율적인 메모리 사용으로 이루어질 수 있습니다.

### 11. GIL(Global Interpreter Lock)

GIL은 파이썬 객체에 대한 엑세스를 보호하는 뮤텍스로, 여러 네이티브 스레드가 한 번에 파이썬 바이트
코드를 실행하지 못하게 방지합니다. 이것은 파이썬의 메모리 관리가 스레드 세이프하지 않기 때문에
필요합니다. 그 결과 멀티 스레드 파이썬 프로그램에서도 한 번에 하나의 스레드만 파이썬 코드를 실행할 수
있으므로 CPU를 사용하는 멀티 스레드 프로그램에는 제한이 될 수 있습니다.

### 12. \_\_init\_\_ vs \_\_new\_\_

- new

  - 인스턴스를 생성하는 메서드입니다.
  - 이 메서드는 init 보다 먼저 호출되며 클래스의 새 인스턴스를 반환하는 역할을 합니다.

- init

  - 파이썬의 초기화 메서드입니다.
  - 인스턴스가 생성된 후에 호출됩니다.
  - 인스턴스 속성을 초기화하는데 사용됩니다.

```python
class MyClass:
    def __new__(cls, *args, **kwargs):
        print("Creating instance")
        return super(MyClass, cls).__new__(cls)

    def __init__(self, value):
        print("Initializing instance")
        self.value = value

obj = MyClass(10)
# Output:
# Creating instance
# Initializing instance
```

### 13. staticmethod vs classmethod

#### staticmethod

- 정적 메서드는 암시적 첫 번째 인자를 받지 않습니다.
- 일반 함수처럼 동작하지만, 클래스의 네임스페이스에 속합니다.

```python
class MyClass:
    @staticmethod
    def static_method():
        print("This is a static method")

MyClass.static_method()  # Output: This is a static method
```

#### classmethod

- 클래스 메서드는 클래스를 암시적 첫 번째 인자로 받습니다.
- 클래스의 모든 인스턴스에 적용되는 클래스 상태를 수정할 수 있습니다.
- 클래스의 객체도 클래스 메서드를 호출할 수 있습니다.

```python
class MyClass:
    _count = 0

    @classmethod
    def increment_count(cls):
        cls._count += 1
        print(f"Count is now {cls._count}")

MyClass.increment_count()  # Output: Count is now 1
MyClass.increment_count()  # Output: Count is now 2
```

### 14 == vs is

- ==: 두 객체의 값을 비교하여 동일한지 확인합니다.
- is: 두 객체의 ID를 비교하여 메모리에서 동일한 객체인지 확인합니다.

```python
a = [1, 2, 3]
b = [1, 2, 3]
c = a

print(a == b)  # Output: True (values are equal)
print(a is b)  # Output: False (different objects in memory)
print(a is c)  # Output: True (same object in memory)
```

### 15. \_\_slot\_\_

slot 은 인스턴스 속성의 생성을 고정된 속성 집합으로 제한하는데 사용되므로, 인스턴스별 dict 생성을
피하여 메모리를 절약할 수 있습니다.

```python
class MyClass:
    __slots__ = ['x', 'y']

    def __init__(self, x, y):
        self.x = x
        self.y = y


class MyClass2:
    x, y = 0, 0

    def __init__(self, x, y):
        self.x = x
        self.y = y


obj = MyClass(1, 2)
print(obj.x, obj.y)  # Output: 1 2
# obj.z = 3  # Raises AttributeError: 'MyClass' object has no attribute 'z'

obj = MyClass2(1, 2)
print(obj.x, obj.y)  # Output: 1 2
obj.z = 3
print(obj.z)  # Output: 3
```

### 16. Context Manager

Context Manager는 원하는 시점에 정확하게 리소스를 할당하고 해제할 수 있는 Python의 구조체입니다.
File Stream, Database Connection or Lock 과 같은 리소스를 관리하는 가장 일반적인 방법은
Context Manager를 사용하는 것입니다.

#### with 문

with 문은 리소스 할당 및 해제 관리를 간소화합니다. with 문을 사용하면 블록이 입력될 때 리소스가 할당
되고 블록이 종료될 때 해제됩니다.

Context Manager는

- 클래스의 메직 메서드 enter, exit 메서드
- contextlib 모듈의 contextmanager 데코레이터

중 하나를 사용하여 구현할 수 있습니다.

```python
class FileManager:
    def __init__(self, filename, mode):
        self.filename = filename
        self.mode = mode

    def __enter__(self):
        self.file = open(self.filename, self.mode)
        return self.file

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.file.close()
        # Handle exceptions if necessary
        if exc_type:
            print(f"An error occurred: {exc_val}")
        return True  # Suppress the exception


# Usage
with FileManager('example.txt', 'w') as f:
    f.write('Hello, World!')


from contextlib import contextmanager

@contextmanager
def open_file(filename, mode):
    file = open(filename, mode)
    try:
        yield file
    finally:
        file.close()


# Usage
with open_file('example.txt', 'w') as f:
    f.write('Hello, World!')
```

Django에서 Context Mangaer는 데이터베이스 트랜잭션, 파일 작업 및 사용자 정의 미들웨어에서 자주
사용됩니다. 장고의 ORM은 컨텍스트 관리자를 사용하여 데이터베이스 트랜잭션을 처리하여 원자적인 작업을
보장합니다.

즉, 리소스 관리, 예를 들어 파일이나 네트워크 연결과 잠금과 같은 리소스를 자동으로 획득하고 해제하여,
오류가 발생하더라도 정리 작업이 잘 수행되도록 도와줍니다.

컨텍스트 관리자를 사용하면 더 깔끔하고 가독성이 높으며 유지 관리가 쉬운 코드를 만들 수 있습니다.

---

### 1. signals

Django 프레임워크 내에서 특정 이벤트가 발생했을 때 다른 부분에서 해당 이벤트에 반응할 수 있도록
하는 기능을 제공합니다. 이는 주로 애플리케이션의 다양한 부분 간 느슨한 결합을 유지하면서 상호작용을
가능하게 합니다. 예를 들어, 모델 인스턴스가 저장될 때 특정 작업을 실행하거나, 사용자가 로그인할 때
특정 작업을 수행하고자 할 때 유용합니다.

#### 주요 개념

- Signal: 이벤트의 일종으로, 신호를 보내는 부분입니다. 내장 신호를 제공하지만, 사용자 정의 시그널도
  생성할 수 있습니다.
- Receiver: 특정 신호를 수신하여 작업을 수행하는 함수입니다. 신호가 발생하면 자동으로 호출됩니다.
- Sender: 신호를 보내는 객체입니다. 특정 모델이나 기타 객체일 수 있습니다.

```python
from django.db.models.signals import post_save
from django.dispatch import receiver
from .models import Author


@receiver(post_save, sender=Author)
def author_saved(sender, instance, **kwargs):
    print(f"Author {instance.name} has been saved.")
```

### 2. 정적 파일 또는 미디어 파일들을 다루는 방법

settings.py 에 STATIC_URL 및 STATICFILES_DIRS 값을 등록합니다.

```python
STATIC_URL = '/static/'
STATICFILES_DIRS = [BASE_DIR / "static"]
```

그 후 URL 패턴을 구성합니다.

```python
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
  # other url patterns
] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

### 3. Django VS FastAPI

- Django

  - Django는 웹 개발에 필요한 다양한 기능을 내장하고 있으며, ORM, 인증 시스템, 관리자 인터페이스,
    폼 처리, 템플릿 엔진 등 다양한 도구를 제공합니다.
  - 일체형(monolithic) 구조로 설계되어 있어, 프로젝트의 모든 부분이 하나의 프레임워크 안에서 통합되고
    일관되게 작동합니다.
  - MTV 패턴: Django 는 Model-Template-View 패턴을 따릅니다.

- FastAPI
  - 경량 프레임워크로, 필요한 기능만 선택적으로 사용할 수 있습니다. 주로 빠른 API 개발을 목표로 합니다.
  - 비동기 프로그래밍을 기본적으로 지원합니다. 이는 고성능 비동기 API를 쉽게 작성할 수 있게 합니다.
  - ASGI(Asynchronous Server Gateway Interface) 표준을 따릅니다. 이는 비동기 처리를
    기본으로 하며, Django의 WSGI(Web Server Gateway Interface) 방식과는 차이가 있습니다.
  - Pydantic 모델 사용: FastAPI는 데이터 검증 및 설정을 위해 Pydantic 모델을 사용합니다.

### 4. select_related and prefetch_related

#### select_related

- 외래 키나 일대일 관계를 미리 로드하여 데이터베이스 쿼리 수를 줄이는 데 사용됩니다.
- 한 번의 쿼리로 관련된 객체를 함께 가져와 성능이 향상될 수 있습니다.
- 다중 관계 및 중첩된 관계도 지원합니다.
- 외래 키나 일대일 관계에서만 사용할 수 있으며, 다대다 관계나 역참조 관계에서는 prefetch_related를
  사용해야 합니다.

#### prefetch_related

- 역방향 참조나 다대다 관계에서 사용합니다.
- select_related 와 같이 조인하는 것이 아니라 필터링한다는 것이 가장 큰 차이점입니다.

#### ORM의 N + 1 문제

select_related 는 주로 1:N의 관계에서 N쪽의 모델이 사용할 수 있습니다.

만약 Post 와 Comment 모델이 있고, 관계가 1:N 인 경우, 아래의 코드는 총 6개의 쿼리를 발생시킵니다.

여기서의 문제는 Comment 와 Post 테이블을 한 번 조인하는 쿼리만 날리면 됐는데, 포스트 개수(5개) 만큼
쿼리를 더 날린 것입니다. (총 6번)

```python
for comment in Comment.objects.all():
    print(comment.post.title)
```

이 문제를 아래와 같이 하면 해결할 수 있습니다.

```python
for comment in Comment.objects.all().select_related("post"):
  print(comment.post.title)
```

즉, select_related 는 Lazy Loading 이 아닌 Eager Loading을 통해, 미리 1번만 쿼리하여,
N+1 문제를 해결할 수 있습니다.

<!-- https://bio-info.tistory.com/174 -->
<!-- https://chatgpt.com/c/21fc6709-e841-42d7-8f3f-bc63044a663f -->

#### select_related 를 사용했는데 left join 이 된다. 어떻게 해결해야 할까?

Django는 기본적으로 LEFT OUTER JOIN 을 사용합니다. 이는 관련 객체가 존재하지 않을 경우에도 결과를
반환하기 위해서입니다. 강제로 INNER JOIN을 구현하는 방식은 Django에서 지원하지 않습니다.
따라서, 요구사항을 달성하기 위해서는 아래와 같은 방법을 선택합니다.

- Raw 쿼리를 사용하여 inner join 하는 방법입니다.

```python
from django.db import connection


def get_books_with_authors():
    with connection.cursor() as cursor:
        cursor.execute("""
            SELECT book.*, author.*
            FROM book
            INNER JOIN author ON book.author_id = author.id
        """)
        results = cursor.fetchall()
    return results
```

- null 조건을 조정하여 마치 INNER_JOIN 처럼 사용하는 방법입니다.

```python
from django.db import models


class BookQuerySet(models.QuerySet):
    def with_authors(self):
        return self.filter(author__isnull=False).select_related('author')


class BookManager(models.Manager):
    def get_queryset(self):
        return BookQuerySet(self.model, using=self._db)

    def with_authors(self):
        return self.get_queryset().with_authors()


class Book(models.Model):
    title = models.CharField(max_length=100)
    author = models.ForeignKey(Author, on_delete=models.CASCADE)

    objects = BookManager()
```

- Subquery를 사용하는 방법입니다.

Subquery와 OuterRef를 사용하여 비슷한 효과를 낼 수 있습니다. 하지만 이 방법은 다소 복잡하고,
상황에 따라 성능이 저하될 수 있습니다.

```python
from django.db.models import OuterRef, Subquery


author_subquery = Author.objects.filter(id=OuterRef('author_id')).values('id')
books = Book.objects.filter(author_id__in=Subquery(author_subquery))


for book in books:
  print(book.title, book.author.name)
```

### 5. Django의 캐시 전략

#### In-Memory Caching (Local Memory Cache)

- 빠르고 간단하지만 단일 프로세스로 제한됩니다. 프로세스가 여러개라면 1/프로세스 개수 번 만큼 요청해야,
  의도한 값을 처리할 수 있습니다.
- 개발 또는 소규모 애플리케이션에 유용합니다.
- django.core.cache.backends.locmem.LocMemCache

#### File-Based Caching

- 파일 시스템에 캐시 데이터를 저장합니다.
- 메모리가 제한되어 있거나 공유 환경에 유용합니다.
- django.core.cache.backends.filebased.FileBasedCache

#### Database Caching

- 데이터베이스를 사용하여 캐시 데이터를 저장합니다.
- 기존 데이터베이스 인프라가 있는 공유 환경에 적합합니다.
- django.core.cache.backends.db.DatabaseCache

#### Memcached

- 고성능 분산 메모리 객체 캐싱 시스템입니다.
- 대규모 애플리케이션에 적합합니다.
- django.core.cache.backends.memcached.MemcachedCache

#### Redis Cache

- 데이터베이스, 캐시 및 메시지 브로커로 사용되는 인메모리 데이터 구조 저장소입니다.
- 복잡한 캐싱 시나리오와 대규모 애플리케이션에 적합합니다.
- django_redis.cache.RedisCache

#### Memcached VS Redis Cache

- Memcached는 2003년에 만들어졌습니다.
- Memcached는 HTML같은 같은, 정적 자원을 캐싱할 때 효율적입니다.
- 직렬화된 형태로 데이터를 저장하도록 되어 제한적입니다.
- 멀티 쓰레드 기능을 지원합니다. 따라서, Redis에 비해 스케일링에 유리합니다.

- Redis는 Memcached의 교훈에 따라 2009년에 만들어졌습니다.
- 다양한 자료구조 및 용량을 지원합니다.
- Memcached는 key 이름을 250 byte까지 제한하고, 단순히 string만 사용합니다.
- Redis는 key, value 이름을 512mb까지 지원하며, hash, set, list, string 등 다양한 데이터
  구조가 있어, 캐싱 및 캐시된 데이터 조작의 편리성을 제공합니다.
- Redis는 다양한 삭제 정책을 지원합니다.

### 6. annotate 과 F

#### annotate

annotate() 메서드를 사용하면 쿼리 집합에 계산된 필드를 추가할 수 있습니다. 이는 합계나 카운트와 같은
집계 값을 쿼리 결과에 직접 추가할 때 특히 유용합니다.

- 데이터 집계: 합, 평균, 카운트 등을 쿼리 결과에 포함해야 할 때
- 사용자 지정: 모델 필드를 기반으로 계산된 필드를 포함해야 하는 경우
- 복잡한 쿼리: 동적 계산을 사용하여 복잡한 DB처리를 수행해야 하는 경우
- 예시: 작가의 이름, 집필한 책의 총개수, 책의 평균 평점

#### F

F expression 을 사용하여, 쿼리에서 모델 필드 값을 직접 참조할 수 있어, 데이터를 Python 객체로
가져올 필요없이 DB에서 계산되도록 할 수 있습니다.

- 데이터베이스 수준 계산: 더하기, 빼기 등의 연산을 수행해야 하는 경우
  이를 수행한 결과로 필터 조건으로 지정할 수도 있습니다.
- 동적 필드 참조: 쿼리에서 필드 값을 동적으로 참조해야 하는 경우
- race condition 제어

---

### 1. 시리얼라이즈 vs 디시리얼라이즈

- 시리얼라이즈: Python 객체를 JSON, XML 등의 형식으로 변환하는 과정
- 디시리얼라이즈: JSON 등의 형식의 데이터를 Python 객체로 변환하는 과정

FastAPI에서 데이터 변환은 주로 Pydantic 모델을 사용하여 처리됩니다. Pydantic은 데이터 검증 및
설정을 위한 Python 라이브러리이며, 요청 데이터/응답 데이터/경로 및 쿼리 매개변수 변환 등의 작업을
수행합니다.
