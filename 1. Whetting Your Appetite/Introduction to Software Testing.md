# Introduction to Software Testing

#

## 목차

* [Simple Testing](#simple-testing)
  * [Running a Fucntion](#running-a-function)
  * [Debugging a Function](#debugging-a-function)
  * [Checking a Function](#checking-a-function)
* [Automating Test Execution](#automating-test-execution)
* [Generating Tests](#generating-tests)
* [Run-Time verification](#run-time-verification)
* [System Input vs Function Input](#system-input-vs-function-input)
* [The Limits of Testing](#the-limits-of-testing)
* [Lessons Learned](#lessons-learned)

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

## Simple Testing

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

우리가 위에서 my_sqrt(2)를 실행해서 얻은 값이 실제로 올바른 값일까요? 우리는 √x X √x = x라는 성질을 이용해서 증명할 수 있습니다.

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

테스트를 해보면 잘 작동합니다. 이렇게 예상 결과를 알고 있다면 assert문을 반복해서 사용해서 프로그램이 올바르게 작동하는지 확인할 수 있습니다.

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

100개의 값으로 **my_sqrt()** 함수를 테스트하는데 얼마나 걸렸을까요?

**Timer module**을 사용하면 경과시간을 측정할 수 있습니다.

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
우리는 이 테스트에 얼마든지 변화를 주어 **my_sqrt** 함수를 반복해서 테스트할 수 있습니다.

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

이제 우리는 루트를 **my_sqrt_checked()** 함수로 계산할 수 있습니다.

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
2. No
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

음수가 **my_sqrt()** 함수 인자로 들어가게 되면, 무한 루프에 빠지게 됩니다. 이러한 문제를 해결하기 위해 우리는 1초 후에 실행을 중단시키는 ExpectTimeout(1)을 사용할 것입니다.

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

그러나 만약 **sqrt_program()** 함수에 숫자가 아닌 다른 입력값이 들어가면 어떻게 될까요?

#
**_Quiz_**
.  sqrt_program('xyzzy')의 실행 결과는?

1. 0
2. 0.0
3. None
4. An exception
#

직접 해봅시다. 숫자가 아닌 문자열을 변환하려고 하면, 런타임 오류가 발생합니다.

```python
from fuzzingbook.ExpectError import ExpectError
```

```python
with ExpectError():
    sqrt_program("xyzzy")
```

```python
Traceback (most recent call last):
  File "c:\Users\Python\hello.py", line 24, in <module>
    sqrt_program("xyzzy")
  File "c:\Users\Python\hello.py", line 15, in sqrt_program
    x = int(arg)
ValueError: invalid literal for int() with base 10: 'xyzzy' (expected)
```

다음은 잘못된 입력을 확인하는 버전의 코드입니다.
```python
def sqrt_program(arg: str) -> None:
    try:
        x = float(arg)
    except ValueError:
        print("Illegal Input")
    else:
        if x < 0:
            print("Illegal Number")
        else:
            print('The root of', x, 'is', my_sqrt(x))
```

```python
sqrt_program("4")

>>> The root of 4.0 is 2.0
```

```python
sqrt_program("-1")

Illegal Number
```

```python
sqrt_program("xyzzy")

Illegal Input
```

우리는 이제 시스템 수준에서 프로그램이 모든 종류의 입력을 정상적으로 처리할 수 있어야 함을 보았습니다.
이것은 프로그램이 오류가 잘 발생하지 않도록 열심히 코딩을 해야 하는 프로그래머에게 부담입니다.
그러나 프로그램이 모든 종류의 입력을 처리할 수 있다면(잘 정의된 오류 메시지가 포함될 수 있습니다.), 우리는 어떤 입력도 보낼 수 있습니다. 하지만 함수를 호출할 때는 정확한 전제 조건을 알아야 합니다.

#

## The Limits of Testing

테스팅에 최선의 노력을 다했음에도 불구하고, 오류가 발생할 수 있는 테스트되지 않은 입력이 있을 수 있습니다.
예를 들어 √0은 division by zero 오류를 발생시킵니다.

```python
with ExpectError():
    root = my_sqrt(0)
```

```python
Traceback (most recent call last):
  File "c:\Users\Python\hello.py", line 28, in <module>
    root = my_sqrt(0)
  File "c:\Users\Python\hello.py", line 10, in my_sqrt
    guess = (approx + x / approx) / 2
ZeroDivisionError: float division by zero (expected)
```

지금까지 테스트에서, 이 조건은 확인하지 않았습니다. 그러나 0~10000000 범위에서 랜덤 값을 생성했더라도 0이라는 값이 나왔을 확률은 100만 분의 1입니다. 함수의 동작이 몇 개의 값에 따라 바뀌더라도 랜덤 테스트로는 이런 값을 생성할 확률이 낮습니다.

물론 x에 허용된 값을 문서화하고 특수한 경우 **x = 0**을 처리하여 그에 따라 함수를 수정할 수 있습니다.

```python
def my_sqrt_fixed(x):
    assert 0 <= x
    if x == 0:
        return 0
    return my_sqrt(x)
```

이를 통해 이제 √0 = 0을 올바르게 계산할 수 있습니다.

```python
assert my_sqrt_fixed(0) == 0
```

그러나 광범위한 테스트를 통해 프로그램의 정확성에 대해 높은 확신을 얻을 수 있지만, 향후 모든 실행이 정확할 것이라는 보장은 제공하지 않는다는 것을 기억해야 합니다. 심지어 모든 결과를 확인하는 런타임 검사조차도 결과를 생성하는 경우에만 결과가 정확하다는 것을 보장할 수 있습니다. 그러나 향후 실행이 검사 실패로 이어지지 않을 것이라는 보장은 없습니다.

우리는 뉴턴 랩슨 법으로 프로그램이 올바르게 구현되었다고 증명할 수 있습니다. 구현이 간단하고 수학이 잘 이해가 된다면.
이것은 소수의 경우에만 해당됩니다. 다른 방법으로

1. 잘 선택된 여러 입력으로 프로그램을 테스트하고
2. 광범위하게 그리고 자동으로 결과를 확인하는 방법이 있습니다.

본 코스에서 나머지 내용: 프로그램을 철저히 테스트하는데 도움이 되는 기술과 프로그램의 상태를 확인하는 기술을 고안해라.

#

## Lessons Learned

* 테스트의 목적은 프로그램의 버그를 찾는 것입니다.
* 테스트 실행, 테스트 생성, 테스트 결과 확인을 자동화할 수 있습니다.
* 테스트는 불완전합니다. 즉 코드에 오류가 없다는 것을 100% 보장하지 않습니다.

---
#

# Exercises

## Exercise 2: Testing Shellsort

다음은 셸 정렬 함수입니다.

```python
def shellsort(elems):
    sorted_elems = elems.copy()
    gaps = [701, 301, 132, 57, 23, 10, 4, 1]
    for gap in gaps:
        for i in range(gap, len(sorted_elems)):
            temp = sorted_elems[i]
            j = i
            while j >= gap and sorted_elems[j - gap] > temp:
                sorted_elems[j] = sorted_elems[j - gap]
                j -= gap
            sorted_elems[j] = temp

    return sorted_elems
```

첫 번째 테스트는 **shellsort()** 함수가 실제로 잘 동작한다는 것을 보여줍니다.

```python
print(shellsort([3, 2, 1]))

>>> [1, 2, 3]
```

#

### Part 1: Manual Test Cases

다양한 입력으로 **shellsort()** 함수를 철저하게 테스트하십시오.
첫 번째로 수동으로 테스트 케이스를 작성하십시오. 그런 다음 극단적인 테스트 케이스를 선택하고 **assert** 문을 사용하여 두 리스트(정렬 X, 정렬 O)를 비교하십시오.

```python
assert shellsort([9, 7, 3, 1]) == [1, 3, 7, 9]
assert shellsort([50, 24, 9, 3, 5]) == [3, 5, 9, 24, 50]
assert shellsort([-150, -50, -60, -85, -100, -10]) == [-150, -100, -85, -60, -50, -10]
```

테스트 케이스를 모두 통과했습니다.

#

### Part 2: Random Inputs

두 번째로 함수 인자로 들어갈 랜덤 리스트들 만드십시오. 다음 두 함수를 이용해서 함수 실행 결과가 정렬되어 있는지 그리고 본래 값에 대한 순열인지 확인하십시오.

```python
def is_sorted(elems):	# 정렬이 되어있는지 확인하는 함수
    return all(elems[i] <= elems[i + 1] for i in range(len(elems) - 1))

print(is_sorted([1, 2, 3]))

>>> True
```

```python
def is_permutation(a, b):	# 순열인지 확인하는 함수
    return len(a) == len(b) and all(a.count(elem) == b.count(elem) for elem in a)

print(is_permutation([3, 2, 1], [1, 3, 2]))

>>> True
```
랜덤 모듈로 1000개의 리스트를 만들고 위의 두 도우미 함수를 이용해서 결과를 검사하십시오.

#

```python
def makeList():    # 랜덤 리스트를 만드는 함수
    length = random.randint(5, 10)
    unsorted_list = []

    for i in range(length):
        num = random.randint(-100, 100)
        unsorted_list.append(num)

    return unsorted_list


def check(unsorted_list):    # 정렬이 되었는지 순열인지 확인하는 함수
    sorted_list = shellsort(unsorted_list)
    if is_sorted(sorted_list) and is_permutation(unsorted_list, sorted_list):
        return True
    else:
        return False


def test(number):    # 테스트 함수 (입력 - 테스트 횟수)
    for i in range(number):
        unsorted_list = makeList()

        if check(unsorted_list):
            return True
        else:
            return False


result = test(1000)

if result:
    print("Pass!")
else:
    print("Fail!")
```

```python
>>> Pass!
```

테스트에 통과했습니다.

#

## Exercise 3: Quadratic Solver

방정식 **ax² + bx + c = 0**가 주어지면, 주어진 방정식에 대한 해를 찾고 싶습니다. 다음 코드는 방정식이 주어지면 해를 계산해줍니다.

```python
def quadratic_solver(a, b, c):
    q = b * b - 4 * a * c
    solution_1 = (-b + my_sqrt_fixed(q)) / (2 * a)
    solution_2 = (-b - my_sqrt_fixed(q)) / (2 * a)
    return (solution_1, solution_2)
```

그러나 위의 구현은 불완전합니다.

1. 0으로 나누기
2. **my_sqrt_fixed()** 함수의 전제조건 위반
어떻게 오류가 발생하고 어떻게 오류를 예방할 수 있을까요?

#

### Part 1: Find bug-triggering inputs

위의 두 경우에 대해 각각 버그를 유발하는 a, b, c 값을 찾으세요.

```python
# [1] b * b - 4 * a * c = 0이 되는 a, b, c 값

with ExpectError():
    print(quadratic_solver(0, 0, 1))
```

```
Traceback (most recent call last):
  File "c:\Users\Python\hello.py", line 27, in <module>
    print(quadratic_solver(0, 0, 1))
  File "c:\Users\Python\hello.py", line 20, in quadratic_solver
    solution_1 = (-b + my_sqrt_fixed(q)) / (2 * a)
ZeroDivisionError: division by zero (expected)
```

```python
# [2] b * b - 4 * a * c < 0이 되는 a, b, c 값
⇾ x ≥ 0 이라는 전제 조건 위반

with ExpectError():
    print(quadratic_solver(8, 1, 5))
```

```python
Traceback (most recent call last):
  File "c:\Users\Python\hello.py", line 27, in <module>
    print(quadratic_solver(8, 1, 5))
  File "c:\Users\Python\hello.py", line 20, in quadratic_solver
    solution_1 = (-b + my_sqrt_fixed(q)) / (2 * a)
  File "c:\Users\Python\hello.py", line 13, in my_sqrt_fixed
    assert 0 <= x
AssertionError (expected)
```

#

### Part 2: Fix the problem

위의 오류를 처리할 수 있도록 코드를 확장하세요. 존재하지 않는 값에 대해서는 **None**을 반환합니다.

```python
def quadratic_solver_fixed(a, b, c):
    if a == 0:
        if b == 0:
            if c == 0:
                # Actually, any value of x
                return (0, None)
            else:
                # No value of x can satisfy c = 0
                return (None, None)
        else:
            return (-c / b, None)

    q = b * b - 4 * a * c
    if q < 0:
        return (None, None)

    if q == 0:
        solution = -b / 2 * a
        return (solution, None)

    solution_1 = (-b + my_sqrt_fixed(q)) / (2 * a)
    solution_2 = (-b - my_sqrt_fixed(q)) / (2 * a)
    return (solution_1, solution_2)
```

```python
with ExpectError():
    print(quadratic_solver_fixed(3, 2, 1))

>>> (None, None)
```

```python
with ExpectError():
    print(quadratic_solver_fixed(0, 0, 1))

>>> (None, None)
```

#

### Part 3: Odds and Ends

무작위 입력으로 이러한 조건을 발견할 확률이 얼마나 될까요? 초당 10억 개의 테스트를 수행할 수 있다고 가정하면, 버그를 발견할 때 까지 얼마나 기다려야 할까요?
a, b, c의 범위를 32bit 정수로 잡으면, a와 b가 모두 0일 경우의 수는 2³² × 2³² 입니다.

```python
combinations = 2 ** 32 * 2 ** 32
print(combinations)

>>> 18446744073709551616
```

1초당 10억개의 테스트를 한다고 하면, 몇년을 기다려야 할까요?

```python
combinations = 2 ** 32 * 2 ** 32
tests_per_second = 1000000000
seconds_per_year = 60 * 60 * 24 * 365.25
tests_per_year = tests_per_second * seconds_per_year
print(combinations / tests_per_year)

>>> 584.5420460906264
```

무려 584년을 기다려야 합니다. 순수한 랜덤 테스트는 테스팅 전략으로 충분하지 않습니다.

#

## Exercise 4: To Infinity and Beyond

x값을 무한대로 설정하면 어떻게 될까요?

```python
infinity = float('inf')

with ExpectTimeout(1):
	y = my_sqrt_fixed(infinity)
```

```
Traceback (most recent call last):
  File "c:\Users\Python\hello.py", line 34, in <module>
    y = my_sqrt_fixed(infinity)
  File "c:\Users\Python\hello.py", line 16, in my_sqrt_fixed
    return my_sqrt(x)
  File "c:\Users\Python\hello.py", line 7, in my_sqrt
    while approx != guess:
  File "c:\Users\Python\hello.py", line 7, in my_sqrt
    while approx != guess:
  File "C:\Users\Python\Python310\lib\site-packages\fuzzingbook\Timeout.py", line 191, in check_time
    raise TimeoutError
TimeoutError (expected)
```
TimeoutErrorr가 발생합니다.
