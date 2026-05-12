---
title: "Codeql 원리와 사용법 분석"
date: 2026-05-12 22:00:00 +0900
categories: [블로그/기술문서]
tags:
---

미션 목표
codeql의 원리를 학습하고 또한 사용법을 익힌다.

## 개요

CodeQL은 GitHub이 Semmle을 인수한 뒤 공개한 정적 분석(SAST) 플랫폼이다.  
일반적인 SAST 도구가 미리 정의된 룰로 코드를 검사한다면, CodeQL은 코드 자체를 "데이터베이스"로 변환해 SQL과 유사한 쿼리 언어(QL)로 취약점을 찾아내는 방식이 특징이다.  
즉 코드를 검사하는 게 아니라 코드에 대해 "질의(Query)"를 한다는 발상이다.  

## 동작 원리

CodeQL의 동작은 크게 세 단계로 이루어진다.  

첫째, 소스 코드를 빌드하면서 컴파일러가 처리하는 AST(추상 구문 트리), 데이터 흐름, 제어 흐름 정보를 추출해 CodeQL 데이터베이스로 만든다.  
이 단계가 가장 중요한데, 코드를 컴파일하듯 분석해 관계형 DB와 유사한 구조로 저장하기 때문이다.  

둘째, 이 데이터베이스에 대해 QL 언어로 작성된 쿼리를 실행한다.  
QL은 선언형 객체지향 언어로, 함수·변수·메서드 호출 같은 코드 요소를 객체처럼 다룬다.  

셋째, 쿼리 결과로 취약 패턴이나 코드 위치를 리포트한다.  
이 결과는 SARIF 포맷으로 출력돼 GitHub의 Security 탭이나 다른 도구와 연동된다.  

## 지원 언어와 핵심 개념

현재 CodeQL은 C/C++, C#, Go, Java/Kotlin, JavaScript/TypeScript, Python, Ruby, Swift를 지원한다.  

핵심 개념은 세 가지로 정리할 수 있다.  
첫째, 데이터베이스(Database)는 분석 대상 소스코드의 구조 정보 전체를 담은 산출물이다.  
둘째, 쿼리(Query)는 데이터베이스에서 원하는 패턴을 찾는 QL 코드다.  
셋째, 테인트 트래킹(Taint Tracking)은 사용자 입력 같은 소스(source)가 위험한 함수(sink)까지 도달하는 경로를 추적하는 기능이다.  

## 실습: 직접 돌려보기

원리만 이해해서는 감이 안 와서 간단한 실습을 진행했다.  

### 1) CodeQL CLI 설치

GitHub 공식 릴리스 페이지에서 CodeQL CLI 번들을 받아 압축을 풀고 PATH에 추가했다.  

```bash
# macOS / Linux 기준
unzip codeql-bundle.zip -d ~/codeql
export PATH="$HOME/codeql:$PATH"
codeql --version
```

### 2) 분석 대상 코드 작성

취약한 패턴이 있는 Python 코드를 하나 만들었다.  

```python
# vuln.py
import sys

def run(cmd):
    return eval(cmd)   # 사용자 입력을 eval에 그대로 전달

if __name__ == "__main__":
    print(run(sys.argv[1]))
```

### 3) CodeQL 데이터베이스 생성

`codeql database create` 명령으로 분석용 DB를 만든다.  

```bash
codeql database create vuln-db --language=python --source-root=./vuln
```

실행하면 코드 구조와 데이터 흐름 정보가 추출되어 `vuln-db` 디렉터리에 저장된다.  

### 4) QL 쿼리 작성

가장 단순한 예시로 `eval` 호출을 찾는 쿼리를 작성했다.  

```ql
// find-eval.ql
import python

from Call c
where c.getFunc().(Name).getId() = "eval"
select c, "위험한 eval() 호출이 발견됨"
```

`from`은 검사 대상, `where`는 조건, `select`는 출력 결과로 SQL과 매우 유사한 구조를 가진다.  

### 5) 쿼리 실행

작성한 쿼리를 DB에 대해 실행하면 결과가 SARIF 포맷으로 출력된다.  

```bash
codeql database analyze vuln-db find-eval.ql \
    --format=sarif-latest --output=result.sarif
```

결과 파일을 열어보면 `vuln.py`의 `eval(cmd)` 위치와 메시지가 그대로 기록되어 있다.  
GitHub Actions에서 동일 작업을 돌리면 이 결과가 Security 탭에 자동으로 표시된다.  

## 활용 사례

CodeQL은 GitHub Security Lab이 주요 오픈소스 프로젝트의 취약점을 찾는 데 사용하면서 알려졌다.  
Linux 커널, Chromium, Apache 등 대규모 코드베이스에서 다수의 CVE가 CodeQL 쿼리로 발견된 사례가 있다.  
또한 CI/CD 파이프라인에 통합해 시프트-레프트(Shift-Left) 보안 전략을 구현하는 데 자주 활용된다.  

## 배운 점

CodeQL을 정리하면서 가장 인상 깊었던 것은 "코드를 데이터처럼 본다"는 발상이었다.  
패턴 매칭 기반의 단순 SAST와 달리, 데이터 흐름 분석과 테인트 트래킹을 쿼리 한 줄에 압축할 수 있는 점이 특히 강력하게 느껴졌다.  
직접 작은 코드에 쿼리를 돌려보니 `from-where-select`라는 친숙한 구조 덕분에 진입장벽이 생각보다 낮았다.  
다만 DB 생성에 시간이 걸리고 복잡한 테인트 트래킹 쿼리는 학습 곡선이 가팔라, 본격적인 활용은 표준 라이브러리(`semmle.*`)부터 익히는 게 효율적이라는 감을 잡을 수 있었다.
