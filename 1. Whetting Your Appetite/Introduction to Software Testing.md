# Introduction to Software Testing

#

## 목차

* Simple Testing
  * Running a Fucntion
  * Debugging a Function
  * Checking a Function
* Automating Test Execution
* Generating Tests
* Run-Time verification
* System Input vs Function Input
* The Limits of Testing
* Lessins Learned

#
---

#

## Can I import the code for my own Python projects?

```bash
$ pip install fuzzingbook
```

```python
>>> from fuzzingbook.<notebook> import <identifier>

Ex)
>>> from fuzzingbook.Fuzzer import RandomFuzzer
>>> f = RandomFuzzer()
>>> f.fuzz()
```

#

### Simple Testing

```python
def my_sqrt(x):
    """Computes the square root of x, using the Newton-Raphson method"""
    approx = None
    guess = x / 2
    while approx != guess:
        approx = guess
        guess = (approx + x / approx) / 2
    return approx
```
**sqrt()** 함수와 동일한 기능을 하는 함수

#

### Running a Function

```python
print(my_sqrt(4))
>>> 2.0

print(my_sqrt(2))
>>> 1.414213562373095
```

#

### Debugging a Function

**my_sqrt()** 함수가 어떻게 동작하는지 보는 가장 쉬운 방법은 중요 포인트에 **printf()** 문을 추가하는 것입니다.
예를 들어 반복문이 실행될수록 approx 값이 실제 값에 더 가까워지는지 보기 위해서, **approx** 값을 출력할 수 있습니다.

```python
def my_sqrt_with_log(x):
    """Computes the square root of x, using the Newton–Raphson method"""
    approx = None
    guess = x / 2
    while approx != guess:
        print("approx =", approx)  # <-- New
        approx = guess
        guess = (approx + x / approx) / 2
    return approx

my_sqrt_with_log(9)
print(my_sqrt(9))
```

```python
approx = None
approx = 4.5
approx = 3.25
approx = 3.0096153846153846
approx = 3.000015360039322
approx = 3.0000000000393214
3.0
```

#

### Checking a Function

우리가 위에서 **my_sqrt(2)**를 실행해서 얻은 값이 실제로 올바른 값일까요? 우리는 √x X √x = x라는 성질을 이용해서 증명할 수 있습니다.

```python
result = my_sqrt(2) * my_sqrt(2)
print(result)

>>> 1.9999999999999996
```

반올림 에러가 있긴 하지만 올바른 값이 나온 거 같습니다.
우리는 입력을 주어 프로그램을 실행시키고 결과가 올바른지 올바르지 않은지 체크를 했습니다.
이러한 테스트는 프로그램이 배포되기 전에 최소한의 품질 보증입니다.

#

## Automating Test Execution

우리는 지금까지 위에 프로그램을 수동으로 테스트했습니다. 이 방법은 매우 유연한 테스트 방법입니다. 하지만 장기적으로 보면, 다소 비효율적입니다.

1. 수동으로는 제한된 범위의 숫자만 테스트할 수 있습니다.
2. 프로그램이 수정되면 테스트 과정을 반복해야 합니다.

그래서 자동화된 테스트는 매우 유용합니다. 자동화된 테스트의 한 가지 간단한 방법은 컴퓨터에게 계산을 시키고, 결과 값을 체크합니다.

예를 들어 √4 = 2를 자동화 테스트해봅시다.

```python
result = my_sqrt(4)
expected_result = 2.0
if result == expected_result:
    print("Test passed")
else:
    print("Test failed")
```

```python
>>> Test passed
```

이 테스트의 좋은 점은 반복해서 실행할 수 있고 최소한 4의 제곱근은 올바르게 계산되도록 할 수 있다는 것입니다.
하지만 여전히 몇 가지 문제가 있습니다.

1. 한 번의 테스트를 위해 5줄의 코드가 필요합니다.
2. 반올림 에러를 해결하지 않았습니다.
3. 한 가지 입력에 대한 결과만 체크했습니다.

이 문제들을 하나씩 해결해봅시다.

첫 번째로 테스트를 더 간결하게 만들어 봅시다. 거의 모든 언어는 조건이 참인지 거짓인지 자동으로 체크하고 거짓이면 실행을 멈추는 기능을 가지고 있습니다. 이것을 **assert** 문 이라고 합니다. 이 기능은 테스트에 매우 유용합니다.

파이썬에서, **assert** 문은 조건이 참이면 아무 일도 일어나지 않지만, 거짓이면 테스트가 실패했음을 알려주는 예외를 발생시킵니다.
예시에서, 우리는 **my_sqrt()** 함수가 예상된 값을 반환하는지 안 하는지 체크하는 데 사용할 수 있습니다.

```python
assert my_sqrt(4) == 2
```

이 코드를 실행시켜보면 √4 = 2 이기 때문에 아무 일도 일어나지 않습니다.

두 번째로 반올림 에러를 해결해봅시다. 반올림 에러는 컴퓨터의 부동소수점 계산 때문에 발생합니다. 그래서 우리는 간단하게 두 부동소수점 값을 비교할 수 없습니다.
그럼 어떻게 해야 할까요? 두 값 사이의 절대적 차이가 **엡실론**으로 표시되는 특정 임계값 미만으로 유지되도록 하겠습니다.

```python
EPSILON = 1e - 8
```

```python
assert abs(my_sqrt(4) - 2) < EPSILON
```

이 함수는 두 부동소수점 값을 비교하기 위한 함수입니다. 구체적인 값을 넣어서 테스트를 해봅시다.

```python
def assertEquals(x, y, epsilon=1e-8):
    assert abs(x - y) < epsilon

assertEquals(my_sqrt(4), 2)
assertEquals(my_sqrt(9), 3)
assertEquals(my_sqrt(100), 10)
```

테스트를 해보면 잘 작동합니다. 이렇게 예상 결과를 알고 있다면 assertion문을 반복해서 사용해서 프로그램이 올바르게 작동하는지 확인할 수 있습니다.

#

## Generating Tests

√x X √x = x라는 성질이 있었습니다. 우리는 이 성질을 이용해서 몇 개의 값을 테스트해볼 수 있습니다.

```python
assertEquals(my_sqrt(2) * my_sqrt(2), 2)
assertEquals(my_sqrt(3) * my_sqrt(3), 3)
assertEquals(my_sqrt(42.11) * my_sqrt(42.11), 42.11)
```

테스트를 해보면 프로그램이 잘 작동합니다.

우리는 위의 성질을 이용하면 수천 개의 값도 쉽게 테스트할 수 있습니다.

```python
for n in range(1, 1000):
    assertEquals(my_sqrt(n) * my_sqrt(n), n)
```

100개의 값으로 my_sqrt() 함수를 테스트하는데 얼마나 걸렸을까요?

Timer module을 사용하면 경과시간을 측정할 수 있습니다.

```python
from fuzzingbook.Timer import Timer
```

```python
with Timer() as t:
    for n in range(1, 1000):
        assertEquals(my_sqrt(n) * my_sqrt(n), n)
print(t.elapsed_time())
```

```python
>>> 0.0022703001741319895
```

1000개의 값을 테스트하는데 단 1초도 걸리지 않았습니다.

이번에는 랜덤 하게 10000개의 값을 테스트해봅시다.

```python
import random
from fuzzingbook.Timer import Timer
```

```python
with Timer() as t:
    for i in range(10000):
        x = 1 + random.random() * 1000000
        assertEquals(my_sqrt(x) * my_sqrt(x), x)
print(t.elapsed_time())
```

```python
>>> 0.027606100076809525
```

1초도 안 되는 시간 동안, 10000개의 랜덤 값을 제곱근 값이 올바르게 게산 되었는지 테스트했습니다.
우리는 이 테스트에 얼마든지 변화를 주어 my_sqrt() 함수를 반복해서 테스트할 수 있습니다.

***Note! 랜덤 함수는 랜덤 값을 생성할 때 편향되지 않습니다. 하지만 프로그램의 동작을 크게 바꿀 만큼 특별한 값은 생성할 가능성은 매우 낮습니다. 이 내용에 대해서는 아래에서 나중에 다루겠습니다.***

#

## Run-Time Verification

테스트를 작성하고 실행하는 대신, 검사 코드를 코드를 구현할 때 추가할 수도 있습니다. 이렇게 하면 각각의 모든 함수 호출이 자동으로 확인됩니다.

```python
def my_sqrt_checked(x):
    root = my_sqrt(x)
    assertEquals(root * root, x)
    return root
```

이제 우리는 루트를 my_sqrt_checked() 함수로 계산할 수 있습니다.

```python
print(my_sqrt_checked(2.0))

>>> 1.414213562373095
```

위의 자동화된 런타임 검사는 두 가지를 가정합니다.

* 이러한 런타임 검사를 공식화할 수 있어야 합니다. 하지만 공식화가 가능하더라도 상황에 따라 공식화 하여 테스트 하는 것이 매우 복잡해질 수 있습니다. Ex) 빅데이터에서 데이터를 가져와 쓰기 작업을 하는데, 이 작업이 제대로 수행되는지 테스트 해야 할 경우 공식화는 가능하더라도 테스트하는 것이 매우 복잡해질 수 있습니다.
또한 런타임 검사는 지역 변수 뿐만 아니라 프로그램의 종류나 상태에 따라서 달라질 수 있습니다.
* 런타임 검사는 비용이 합리적이어야 합니다. 검사 비용이 많이 들지 않더라도 큰 데이터 구조를 검사해야 하는 경우에는 검사 비용이 어마어마 해질 수 있습니다. 실제로 런타임 검사는 프로그램 제작 동안에는 비활성화됩니다. 반면에 포괄적인 런타임 검사는 오류를 찾고 신속하게 디버깅하기 좋은 방법입니다.

런타임 검사의 중요한 한계는 검사 된 결과가 있을 경우에만 정확성을 보장합니다. 즉 예시에서 처럼 음수나 문자열이 입력으로 들어가면 계산 결과가 나오지 않기 때문에 정확성을 보장하기 힘듭니다. 하지만 symbolic verification의 경우 입력이 들어갔을 때 그와 관련된 모든 동작을 분석하고 검증하기 때문에 이런 예외 상황에서도 정확성을 보장합니다.

#
**_Quiz_**
.  런타임 검사가 항상 올바른 결과가 나올 것이라고 보장합니까?

1. Yes
1. No
#

## System Input vs Function Input

이 시점에서 우리는 my_sqrt()를 다른 프로그래머가 사용할 수 있도록 할 수 있으며, 다른 프로그래머는 이를 코드에 포함할 수 있습니다. my_sqrt 함수는 third-party부터 오는 입력을 처리해야 할 것입니다. 즉, 프로그래머에 의해 제어되지 않습니다.

입력이 third-party 제어 하에 있는 문자열인 프로그램 sqrt_program()로 이 시스템 입력을 시뮬레이션 해보겠습니다.

```python
def sqrt_program(arg: str) -> None:
    x = int(arg)
    print('The root of', x, 'is', my_sqrt(x))
```

```python
sqrt_program("4")

>>> The root of 4 is 2.0
```

무엇이 문제일까요? 외부 입력값이 유효한지 유효하지 않은지 검증하지 않습니다. 예를 들어 sqrt_program(-1)이 실행되면 무슨 일이 발생할까요?

음수가 my_sqrt() 함수 인자로 들어가게 되면, 무한 루프에 빠지게 됩니다. 이러한 문제를 해결하기 위해 우리는 1초 후에 실행을 중단시키는 ExpectTimeout(1)을 사용할 것입니다.

```python
from fuzzingbook.ExpectError import ExpectTimeout
```

```python
with ExpectTimeout(1):
    sqrt_program("-1")
```

```python
Traceback (most recent call last):
  File "c:\Users\Python\hello.py", line 20, in <module>
    sqrt_program("-1")
  File "c:\Users\Python\hello.py", line 16, in sqrt_program
    print('The root of', x, 'is', my_sqrt(x))
  File "c:\Users\Python\hello.py", line 10, in my_sqrt
    guess = (approx + x / approx) / 2
  File "c:\Users\Python\hello.py", line 10, in my_sqrt
    guess = (approx + x / approx) / 2
  File "C:\Users\AppData\Local\Programs\Python\Python310\lib\site-packages\fuzzingbook\Timeout.py", line 191, in check_time
    raise TimeoutError
TimeoutError (expected)
```

위의 메시지는 문제가 발생했음을 나타내는 오류 메시지입니다. 오류가 발생했을 때 활성 상태였던 함수 및 라인의 호출 스택을 나열합니다. 맨 아래줄은 마지막으로 실행된 줄입니다. 위의 라인은 함수 호출을 나타냅니다.

우리는 코드가 예외로 종료되는 것을 원치 않습니다. 따라서 외부 입력을 받을 때, 입력값이 적절하게 검증되었는지 확인해야 합니다.
예를 들어

```python
def sqrt_program(arg: str) -> None:
    x = int(arg)
    if x < 0:
        print("Illegal Input")
    else:
        print('The root of', x, 'is', my_sqrt(x))
```

```python
sqrt_program("-1")

>>> Illegal Input
```

그러나 만약 sqrt_program() 함수에 숫자가 아닌 다른 입력값이 들어가면 어떻게 될까요?

#
**_Quiz_**
.  sqrt_program('xyzzy')의 실행 결과는?

1. 0
2. 0.0
3. None
4. An exception
#

직접 해봅시다. 숫자가 아닌 문자열을 변환하려고 하면, 런타임 오류가 발생합니다.
