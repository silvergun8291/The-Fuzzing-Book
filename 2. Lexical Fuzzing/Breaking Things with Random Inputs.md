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

* **fuzzer**의 기본 클래인 Fuzzer
* **programs**의 기본 클래스인 Runner

#

## Fuzzers

**Fuzzer**는 fuzzers의 기본 클래스이며, 간단화 인스턴스로 **RandomFuzzer**를 사용합니다.
**Fuzzer** 객체의 **fuzz()** 메서드는 생성된 입력이 포함된 문자열을 반환합니다.

```python
>>> random_fuzzer = RandomFuzzer()
>>> random_fuzzer.fuzz()
'%$<1&<%+=!"83?+)9:++9138 42/ "7;0-,)06 "1(2;6>?99$%7!!*#96=>2&-/(5*)=$;0$$+;<12"?30&'
```

**RandomFuzzer()**는 생성자를 사용하면 다음과 같은 키워드 인수를 지정할 수 있습니다.

```python
>>> print(RandomFuzzer.__init__.__doc__)
Produce strings of `min_length` to `max_length` characters
           in the range [`char_start`, `char_start` + `char_range`)

>>> random_fuzzer = RandomFuzzer(min_length=10, max_length=20, char_start=65, char_range=26)
>>> random_fuzzer.fuzz()
'XGZVDDPZOOW'
```

![diagram](https://velog.velcdn.com/images/silvergun8291/post/a5c4d2d3-5e5d-429c-907b-afd62601b289/image.JPG)


## Runners

퍼저는 PASS, FAIL, UNRESOLVED로 Runner와 페어링 될 수 있습니다. **Print Runner**는 주어진 입력을 출력하고 PASS 결과를 반환합니다.

```python
>>> print_runner = PrintRunner()
>>> random_fuzzer.run(print_runner)
EQYGAXPTVPJGTYHXFJ

('EQYGAXPTVPJGTYHXFJ', 'UNRESOLVED')
```

**ProgramRunner**는 생성된 입력을 외부 프로그램에 공급합니다. 결과는 프로그램의 상태(완료된 프로레스 인스턴스)와 결과(PASS, FAIL, UNRESOLVED)의 쌍입니다.

```python
>>> cat = ProgramRunner('cat')
>>> random_fuzzer.run(cat)
(CompletedProcess(args='cat', returncode=0, stdout='BZOQTXFBTEOVYX', stderr=''),
 'PASS')
 ```
 
![diagram2](https://velog.velcdn.com/images/silvergun8291/post/480dc1c6-2f25-4dda-a6e9-4cc7e2680a88/image.JPG)

#

## A Testing Assignment

퍼징은 "1988년 가을의 어둡고 폭풍우가 몰아치는 밤"에 발명되었습니다. [Takanen et al, 2008]. 매디슨 위스콘신에 있는 아파트에 있었던, 바튼 밀러 교수는 1200보의 전화선을 통해 그의 대학 컴퓨터에 접속했습니다. 천둥번개는 회선에 노이즈를 발생시켰고, 이 노이즈는 UNIX 명령어들이 잘못된 입력을 받아 충돌하는 원인이 되었습니다. 잦은 충돌로 인해 그는 놀랐고, 프로그램이 저 노이즈보다 더 견고해야 한다고 생각했습니다. 과학자로서, 그는 문제의 범위와 원인을 조사하기를 원했습니다.그래서 위스콘신-매디슨 대학에서 그의 학생들에게 프로그래밍 과제를 냈습니다. 그 과제는 학생들이 최초의 퍼저를 만들게 하는 것이었습니다.

과제는 다음과 같습니다.

> 이 프로젝트의 목표는 예측할 수 없는 입력 스트림이 주어졌을 때 다양한 UNIX 유틸리티 프로그램의 견고성을 평가하는 것입니다. [...] 첫 번째로, fuzz generator를 만드세요. 이것은 랜덤 문자 스트림을 출력하는 프로그램입니다. 둘째, fuzz generator를 사용하여 가능한 한 많은 UNIX 유틸리티를 공격하세요.

이 과제는 퍼징의 본질을 다룹니다: 무작위 입력을 만들고 그 입력이 프로그램에 오류를 일으키는지 확인하세요.

#

## A Simple Fuzzer

이 과제를 완수하고 fuzz generator를 만들어 봅시다. 아이디어는 임의의 문잘르 생성하여 버퍼 문자열 변수에 추가한 후 문자열을 반환하는 것입니다.

이 구현에서는 다음과 같은 파이썬 기능을 사용합니다.

* **random.randrange(start, end)** – return a random number  [  **start, end**  )
* **range(start, end)** – create a list with integers in the range  [  **start, end**  ) . Typically used in iterations.
* **for elem in list: body** – execute **body** in a loop with **elem** taking each value from **list**.
* **for i in range(start, end): body** – execute **body** in a loop with **i** from **start** to **end**  −  1.
* **chr(n)** – return a character with ASCII code n

우리는 랜덤한 숫자를 사용하기 위해서, 각각의 모듈을 import 해야합니다.

```python
import random
```









