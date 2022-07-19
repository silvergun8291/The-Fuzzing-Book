# Fuzzing: Breaking Things with Random Inputs

#

## 목차

* Synopsis
* A Testing Assignment
* A Simple Fuzzer
* Fuzzing External Programs
	* Creating Input Files
	* Invoking External Programs
	* Long-Running Fuzzing
* Bugs Fuzzers Find
	* Buffer Overflows
	* Missing Error Checks
	* Rogue Numbers
* Catching Errors
	* Generic Checkers
	* Program-Specific Checkers
	* Static Code Checkers
* A Fuzzing Architecture
	* Runner Classes
	* Fuzzer Classes
* Lessons Learned
* Exercises
	* Exercise 1: Simulate Troff
	* Exercise 2: Run Simulated Troff
	* Exercise 3: Run Real Troff

#
---
#

## Synopsis

이 챕터에서 제공하는 코드를 사용하기 위해서 추가하세요.

```python
>>> from fuzzingbook.Fuzzer import <identifier>
```

이 챕터는 A Fuzzing Architecture에서 소개된 두가지 중요한 클래스를 제공합니다.

* fuzzer의 기본 클래인 Fuzzer
* programs의 기본 클래스인 Runner

#

## Fuzzers

Fuzzer는 fuzzers의 기본 클래스이며, 간단화 인스턴스로 RandomFuzzer를 사용합니다.
Fuzzer 객체의 fuzz() 메서드는 생성된 입력이 포함된 문자열을 반환합니다.

```python
>>> random_fuzzer = RandomFuzzer()
>>> random_fuzzer.fuzz()
'%$<1&<%+=!"83?+)9:++9138 42/ "7;0-,)06 "1(2;6>?99$%7!!*#96=>2&-/(5*)=$;0$$+;<12"?30&'
```

RandomFuzzer()는 생성자를 사용하면 다음과 같은 키워드 인수를 지정할 수 있습니다.

```python
>>> print(RandomFuzzer.__init__.__doc__)
Produce strings of `min_length` to `max_length` characters
           in the range [`char_start`, `char_start` + `char_range`)

>>> random_fuzzer = RandomFuzzer(min_length=10, max_length=20, char_start=65, char_range=26)
>>> random_fuzzer.fuzz()
'XGZVDDPZOOW'
```

<img src="https://velog.velcdn.com/images/silvergun8291/post/a5c4d2d3-5e5d-429c-907b-afd62601b289/image.JPG">
