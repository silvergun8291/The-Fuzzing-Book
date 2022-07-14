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

첫 번째로 테스트를 더 간결하게 만들어 봅시다. 거의 모든 언어는 조건이 참인지 거짓인지 자동으로 체크하고 거짓이면 실행을 멈추는 기능을 가지고 있습니다. 이것을 가정 **설정문(assert)**이라고 합니다. 이 기능은 테스트에 매우 유용합니다.
