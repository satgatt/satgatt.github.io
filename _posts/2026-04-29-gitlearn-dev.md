---
title: "Git 배우기:
date: 2026-04-29 22:00:00 +0900
categories: [개발]
tags:
---

[Learn Git Branching](https://learngitbranching.js.org/?locale=ko) 사이트를 끝까지 풀고, 등장한 명령어를 정리한다.  

## 기본

`git commit` — 새 커밋을 만든다.  

`git branch <이름>` — 지금 자리(HEAD)에 새 브랜치를 만든다. 만들기만 하고 그 브랜치로 옮겨가지는 않는다.  

`git checkout <대상>` — 다른 브랜치나 커밋으로 자리를 옮긴다. 커밋 해시로 직접 옮기면 어떤 브랜치에도 속하지 않은 상태가 된다(detached HEAD).  

`git checkout -b <이름>` — 새 브랜치를 만들고 곧바로 그 브랜치로 옮겨가는 단축형.  

`git merge <대상>` — 다른 브랜치를 지금 브랜치로 합친다. 두 흐름이 만나는 자리에 부모가 둘인 merge commit이 생긴다.  

## 커밋 옮기기

`git rebase <대상>` — 지금 브랜치의 커밋들을 떼어내 대상 브랜치 위에 다시 붙인다. 히스토리가 깔끔하게 한 줄로 정리된다. 다만 커밋이 새로 만들어지는 형태라, 다른 사람과 공유 중인 브랜치에는 쓰지 않는 게 좋다.  

`git cherry-pick <커밋> [<커밋> ...]` — 원하는 커밋만 골라서 지금 자리 뒤에 가져다 붙인다.  

## 상대 참조

`HEAD^` — 부모 커밋으로 한 칸 위로 간다. `HEAD~` 와 같다.  

`HEAD~N` — N칸 위로 한 번에 간다.  

`HEAD~~` — `HEAD~2` 와 같다. `~` 를 여러 번 쓴 형태.  

`HEAD^2` — 머지 커밋의 두 번째 부모로 간다. 머지 커밋만 부모가 둘이라서 이때만 쓸 수 있다.  

## 브랜치 강제 이동

`git branch -f <브랜치> <대상>` — 브랜치 위치를 원하는 자리로 강제로 옮긴다.  

## 되돌리기

`git reset HEAD~N` — 브랜치를 N칸 뒤로 되돌린다. 혼자 쓰는 로컬 브랜치에서만 쓴다. 이미 다른 사람과 공유한 브랜치에는 쓰지 않는다.  

`git revert <커밋>` — 해당 커밋의 변경을 취소하는 새 커밋을 추가한다. 히스토리는 늘어나지만 공유 중인 브랜치에서도 안전하다.  

## 커밋 수정

`git commit --amend` — 방금 만든 커밋을 새로 쌓지 않고 그 자리에서 고친다.  

`git rebase -i HEAD~N` — 최근 N개 커밋을 순서를 바꾸거나, 합치거나(squash), 빼버릴 수 있는 인터랙티브 모드로 연다.  

## 태그

`git tag <태그명> <커밋>` — 특정 커밋에 고정된 이름표를 붙인다. 브랜치와 달리 새 커밋이 쌓여도 자리를 옮기지 않는다.  

`git describe <ref>` — 지정한 위치에서 가장 가까운 태그를 기준으로 어디쯤에 있는지 표시해준다. 결과는 `<태그>-<거리>-g<해시>` 형태로 나온다.  

## 원격

`git clone` — 원격 저장소를 로컬에 통째로 복사해 온다.  

`o/main` (= `origin/main`) — 원격 추적 브랜치. 마지막으로 확인한 원격의 위치를 기억해두는 로컬 쪽 표시다. 실제 원격 자체와는 별개로 움직인다.  

`git fetch` — 원격에 새로 올라온 커밋을 받아와서 원격 추적 브랜치만 갱신한다. 내 로컬 작업 브랜치는 건드리지 않는다.  

`git pull` — fetch와 merge를 한 번에 처리한다. 받아온 변경이 곧바로 내 로컬 브랜치에 반영된다.  

`git fakeTeamwork [N]` — 사이트 전용 명령. 원격에 다른 사람이 N개의 커밋을 푸시한 상황을 만들어준다.  

`git push` — 내 로컬 커밋을 원격에 올린다.  

`git branch -u origin/main <브랜치>` — 로컬 브랜치의 upstream을 지정한 원격 브랜치로 설정한다. 이렇게 해두면 그 브랜치에서 `git push` 만 쳐도 알아서 그곳으로 올라간다.  

## Refspec

push와 fetch는 `<source>:<destination>` 이라는 공통 문법을 쓴다.  

`git push origin <source>:<destination>` — 로컬의 source를 원격의 destination 자리에 보낸다.  

`git push origin main^:foo` — 로컬의 `main^` 커밋을 원격의 `foo` 브랜치 자리에 보낸다.  

`git push origin :foo` — 보낼 쪽(source)을 비우면 원격의 `foo` 브랜치가 삭제된다.  

`git fetch origin <source>:<destination>` — 원격의 source를 가져와서 로컬의 destination에 반영한다. 그 자리에 브랜치가 없으면 새로 만든다.  

`git fetch origin :bar` — source를 비우면 로컬에 빈 `bar` 브랜치가 만들어진다.  

`git pull origin <source>:<destination>` — fetch와 같은 인자 형식을 쓰고, 가져온 변경을 지금 브랜치에 머지까지 한다.
