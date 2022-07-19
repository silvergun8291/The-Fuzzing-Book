# Fuzzing: Breaking Things with Random Inputs

#

## 목차

* Synopsis
* A Testing Assignment
* A Simple Fuzzer
* Fuzzing External Programs
	- Creating Input Files
	- Invoking External Programs
	- Long-Running Fuzzing
* Bugs Fuzzers Find
	- Buffer Overflows
	- Missing Error Checks
	- Rogue Numbers
* Catching Errors
	- Generic Checkers
	- Program-Specific Checkers
	- Static Code Checkers
* A Fuzzing Architecture
	- Runner Classes
	- Fuzzer Classes
* Lessons Learned
* Exercises
	- Exercise 1: Simulate Troff
	- Exercise 2: Run Simulated Troff
	- Exercise 3: Run Real Troff

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

이 과제를 완수하고 fuzz generator를 만들어 봅시다. 아이디어는 임의의 문장을 생성하여 버퍼 문자열 변수에 추가한 후 문자열을 반환하는 것입니다.

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

여기에 실제 **fuzzer()** 함수가 있습니다.

```python
def fuzzer(max_length: int = 100, char_start: int = 32, char_range: int = 32) -> str:
    """A string of up to `max_length` characters
       in the range [`char_start`, `char_start` + `char_range`)"""
    string_length = random.randrange(0, max_length + 1)
    out = ""
    for i in range(0, string_length):
        out += chr(random.randrange(char_start, char_start + char_range))
    return out
```

**fuzzer()** 함수는 기본 인수를 사용하여 다음과 같은 임의의 문자열을 반환합니다.

```python
print(fuzzer())

>>> !7#%"*#0=)$;%6*;>638:*>80"=</>(/*:-(2<4 !:5*6856&?""11<7+%<%7,4.8,*+&,,$,."
```

바트 밀러는 이러한 무작위 비정형 데이터의 이름으로 "fuzz"라는 용어를 만들었습니다. 이제 이 "fuzz" 문자열이 특정 입력 형식(예를 들어 쉼표로 구분된 값 목록 또는 이메일 주소)의 데이터가 들어올 것이라고 예상하는 프로그램의 입력이라고 상상해 보십시오. 프로그램이 이러한 입력을 문제없이 처리할 수 있을까요?


Fuzzing은 쉽게 다른 종류의 입력값도 만들도록 설정할 수 있습니다. 예를 들어 우리는 fuzzer()가 일련의 소문자를 생성하도록 할 수 있습니다.

```python
# max length: 1000, start: 'a', range: 26 ('a' ~ 'z')
fuzz = fuzzer(1000, ord('a'), 26)
print(fuzz)
```

```python
alzbcutzsbsxwuftbioecunclfnpvljmnfpmivwboohvtpezhqfngeqjrzovkywaankfvxultmtzrkldpbybiixrzolfyzcrxoclvgkhmgfxfdggsvqcygqhbzzskscocrxllosagkvaszlngpysurezehvcqcghygphnhonehczraznkibltfmocxddoxcmrvatcleysksodzlwmzdndoxrjfqigjhqjxkblyrtoaydlwwisrvxtxsejhfbnforvlfisojqaktcxpmjqsfsycisoexjctydzxzzutukdztxvdpqbjuqmsectwjvylvbixzfmqiabdnihqagsvlyxwxxconminadcaqjdzcnzfjlwccyudmdfceiepwvyggepjxoeqaqbjzvmjdlebxqvehkmlevoofjlilegieeihmetjappbisqgrjhglzgffqrdqcwfmmwqecxlqfpvgtvcddvmwkplmwadgiyckrfjddxnegvmxravaunzwhpfpyzuyyavwwtgykwfszasvlbwojetvcygectelwkputfczgsfsbclnkzzcjfywitooygjwqujseflqyvqgyzpvknddz
```

프로그램이 식별자를 입력으로 예상하면, 이 정도로 긴 식별자가 입력으로 들어올 것이라고 예상할 수 있을까요?

#

**_Quiz_**
.  다음중 임의의 긴 10진수 문자열을 생성할 수 있는 코드는?

1. **fuzzer(100, 1, 100)**
2. **fuzzer(100, 100, 0)**
3. **fuzzer(100, 10, ord('0'))**
4. **fuzzer(100, ord('0'), 10)**

#

**정답**

```python
fuzz = fuzzer(100, ord('0'), 10)
print(fuzz)

>>> 050199092904721615267546627640773972382632848750065259698551700448752187153
```

#

## Fuzzing External programs

외부 프로그램에 실제로 fuzzed 입력을 넣으면 어떻게 되는지 봅시다. 이를 위해 두 단계로 진행하겠습니다. 첫 번째로 우리는 fuzzed 테스트 데이터로 입력 파일을 만든 다음, 이 입력 파일을 선택한 프로그램에 넣습니다.

#

## Creating Input Files

파일 시스템을 흐트러뜨리지 않도록 임시 파일 이름을 얻읍시다.

```python
import os
import tempfile
```

```python
basename = "input.txt"
tempdir = tempfile.mkdtemp()
FILE = os.path.join(tempdir, basename)
print(FILE)

>>> C:\Users\swlab\AppData\Local\Temp\tmp1p7w1ho4\input.txt
```

이제 쓰기 작업을 위해 이 파일을 열 수 있습니다. 파이썬 open() 함수는 임의의 내용을 쓸 수 있는 파일을 엽니다. 일반적으로 with 문과 함께 사용되어 파일이 더 이상 필요하지 않으면 즉시 닫을 수 있습니다.

```python
data = fuzzer()
with open(FILE, "w") as f:
    f.write(data)
```

파일 내용을 읽으면 파일이 실제로 생성되었는지 확인할 수 있습니다.

```python
contents = open(FILE).read()
print(contents)
assert(contents == data)

>>> !.:),08>84-$3<-02-6/ 5<6>1'+;<;06,2>5&$4560-+"
```

#

### Invoking External Programs

[bc download](https://ftp.jaist.ac.jp/pub/GNU/bc/)

이제 입력파일도 있으니, **bc** 계산기 프로그램을 테스트 해봅시다.

**bc**를 호출하려면 파이썬 **subprocess** 모듈을 사용하십시오. 작동 방식은 다음과 같습니다.

```python
import os
import subprocess
```

```python
program = "bc"
with open(FILE, "w") as f:
    f.write("2 + 2\n")
result = subprocess.run([program, FILE],
                        stdin=subprocess.DEVNULL,
                        stdout=subprocess.PIPE,
                        stderr=subprocess.PIPE,
                        universal_newlines=True)  # Will be "text" in Python 3.7
```

우리는 **결과**로 부터 프로그램 출력을 확인할 수 있습니다. **bc**의 경우, 출력은 산술식을 계산한 결과입니다.

```python
print(result.stdout)

>>> '4\n'
```

우리는 또한 상태를 확인할 수 있습니다. 값이 0이면 프로그램이 올바르게 종료되었음을 나타냅니다.

```python
print(result.returncode)

>>> 0
```

모든 오류 메시지는 **result.stderr**를 통해 볼 수 있습니다.

```python
print(result.stderr)

>>> ''
```

**bc** 대신, 당신이 좋아하는 어떤 프로그램도 넣을 수 있습니다. 그러나 프로그램이 시스템을 변경하거나 손상시킬 수 있는 경우 fuzzed 입력에 이러한 작업을 수행하는 데이터 또는 명령어가 포함될 수 있으니 주의해야 합니다.

#

**_Quiz_**
.  재미삼아 파일 제거 프로그램(예: **rm -rf FILE**)을 테스트한다고 상상해 보십시오. 여기서 **FILE**은 fuzzer()에 의해 생성된 문자열입니다. **fuzzer()** (기본 인수 포함)가 **FILE** 인수를 생성하여 모든 파일을 삭제할 가능성은 얼마나 됩니까?

1. About one in a billion
2. About one in a million
3. About one in a thousand
4. About one in ten

#

사실 그 가능성은 당신이 생각하는 것보다 높습니. 예를 들어 **/** (루트 디렉터리)를 제거하면 전체 파일 시스템이 사라집니다. **~** (홈 디렉터리)를 제거하면 모든 파일이 사라집니다. **.** (현재 디렉터리)를 제거하면 현재 디렉터리의 모든 파일이 사라집니다. 이 중 하나를 만들기 위해서는 문자열 길이가 1 (100개 중 1개)과 이 세글자 중 하나 (32개 중 3개)가 필요합니다. 실제로 1000분의 1확률입니다.

```python
1/100 * 3/32

>>> 0.0009375
```

그러나 두 번째 문자가 공백인 한 어떤 문자열도 실제로 처리할 수 있습니다. 즉 **rm -rf / WHATEVER**는 먼저 /를 처리하고 그 다음 이어지는 문자열을 처리할 수 있습니다. 첫 번째 문자는 32개 중 3개, 공백은 32개 중 1개입니다. 그래서 우리는 300개 중 1개입니다.

```python
3/32 * 1/32

>>> 0.0029296875
```

퍼징 테스트가 일반적으로 수백만 번 실행된다는 점을 고려할 때 이 위험을 감수하고 싶지 않을 것입니다. 도커 컨테이너와 같이 원하는 대로 재설정할 수 있는 안전한 환경에서 퍼저를 실행하세요.

#

### Long-Running Fuzzing

이제 테스트한 프로그램에 많은 수의 입력을 제공하여 프로그램이 충돌하는지 여부를 확인합시다. runs 변수는 모든 결과를 입력 데이터와 실제 결과의 쌍으로 저장합니다. (참고: 이 작업을 실행한느 데 시간이 걸릴 수 있습니다.)

```python
trials = 100
program = "bc"

runs = []

for i in range(trials):
    data = fuzzer()
    with open(FILE, "w") as f:
        f.write(data)
    result = subprocess.run([program, FILE],
                            stdin=subprocess.DEVNULL,
                            stdout=subprocess.PIPE,
                            stderr=subprocess.PIPE,
                            universal_newlines=True)
    runs.append((data, result))
```

<span style="color: red">
이제 일부 통계에 대한 런을 쿼리할 수 있습니다.예를 들어, 실제로 통과된 실행 수를 쿼리할 수 있습니다. 즉, 오류 메시지가 없습니다.여기서는 목록 이해를 사용합니다. 조건이 참인 경우 목록의 요소 양식 식은 각 요소가 목록에서 가져온 계산된 식 목록을 반환합니다.(실제로 목록 이해는 목록 생성기를 반환하지만, 우리의 목적을 위해 생성기는 목록처럼 작동합니다.)여기에서는 조건이 유지되는 모든 요소에 대해 1이라는 식을 가지고 있으며, 목록의 모든 요소를 합하기 위해 sum()을 사용합니다.
</span>

```python
sum(1 for (data, result) in runs if result.stderr == "")

>>> 6
```

대부분의 입력은 유효하지 않으며, 무작위 입력에 유효한 산술 식이 포함될 가능성이 낮기 때문에 크게 놀랄 일은 아닙니다.

첫 번째 에러 메시지를 봐봅시다.

```python
errors = [(data, result) for (data, result) in runs if result.stderr != ""]
(first_data, first_result) = errors[0]

print(repr(first_data))
print(first_result.stderr)
```

```python
'47&&,/8/\'(9-(!#%.7%7=+%0((<!2=>*=,7(>?67$1" ?668>&97(>\'0! 1=01 $;\'050 !.#;;\'2&= ?\'8'
/tmp/tmpfw0cxb3p/input.txt 1: syntax error
/tmp/tmpfw0cxb3p/input.txt 1: illegal character: '
(standard_in) 1: syntax error
```

잘못된 문자, 구문 분석 오류 또는 구문 오류 이외의 메시지가 포함된 실행이 있습니까? (예를 들어, 충돌이나 치명적인 버그를 발견했는가?) 많지 않음:

```
errors = [result.stderr for (data, result) in runs if
 result.stderr != ""
 and "illegal character" not in result.stderr
 and "parse error" not in result.stderr
 and "syntax error" not in result.stderr]

 print(errors)


 >>> []
 ```

아마도 충돌은 bc가 그냥 충돌하는 것으로 나타날 것입니다.안타깝게도 반환 코드는 0이 아닙니다.

```python
result = sum(1 for (data, result) in runs if result.returncode != 0)
print(result)

>>> 0
```

위의 bc 테스트를 좀 더 진행하면 어떨까요? 테스트가 진행되는 동안, 1989년의 최첨단 기술이 어땠는지 살펴봅시다.

#

## Bugs Fuzzers Find

1989년 Miller와 그의 학생들이 첫 퍼저를 실행했을 때 놀라운 결과를 얻었습니다. 퍼징한 UNIX 유틸리티의 약 3분의 1이 문제가 있었습니다. 퍼징 입력이 들어갔을 때 충돌, 중단 또는 실패가 발생했다는 것입니다. [Miller et al, 여기에는 위의 bc 프로그램도 포함되어 있습니다. (분명히, 이제 버그가 수정되었습니다!)

이러한 UNIX 유틸리티 중 많은 부분이 네트워크 입력을 처리하는 스크립트에 사용되었음을 고려하면, 이는 놀라운 결과였습니다. 프로그래머들은 신속하게 자체 퍼저를 구축하고 실행했으며, 보고된 오류를 수정하기 위해 서둘렀고, 더 이상 외부 입력을 신뢰하지 않는 것을 배웠습니다.

밀러의 실험 결과 어떤 문제가 발견되었나요? 프로그래머들이 1990년에 저지른 실수는 오늘날에도 여전히 같은 실수라는 것이 밝혀졌습니다.

#

### Buffer Overflows

많은 프로그램에는 입력 및 입력 요소에 대한 최대 길이가 내장되어 있습니다. C와 같은 언어에서는 프로그램(또는 프로그래머)이 눈치채지 못하게 이 길이를 초과하기 쉬우며, 이른바 버퍼 오버플로(buffer overflow)를 유발합니다. 예를 들어, 다음 코드는 입력이 8자를 초과하더라도 입력 문자열을 weekday 문자열로 복사합니다.

```c
char weekday[9]; // 8 characters + trailing '\0' terminator
strcpy (weekday, input);
```

아이러니하게도, 입력이 "Wednesday"(9자)이면 이 작업은 실패합니다. 초과 문자('y'와 '\0' 문자열 종결자)는 weekday 이후 메모리 공간으로 복사되어 임의 동작을 트리거합니다. <span style아마도 'n'에서 'y'로 설정된 부울 문자 변수일 수 있습니다.퍼징을 사용하면 임의의 긴 입력 및 입력 요소를 매우 쉽게 생성할 수 있습니다.
