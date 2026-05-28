---
title: "Linux 기본 명령어 정리"
date: 2026-05-28 09:00:00 +0900
categories: [이어드림스쿨, Linux]
tags: [linux, terminal, cli, 명령어]
---

# 1. 현재 위치 확인

## 현재 디렉터리 확인

- 현재 내가 어느 위치에 있는지 절대 경로로 출력

```bash
pwd
```

## 바탕화면으로 이동

- `cd` 명령어로 원하는 디렉터리로 이동

```bash
cd ~/Desktop
```

## 현재 위치의 파일/폴더 목록 보기

- 현재 디렉터리 안의 파일과 폴더를 나열

```bash
ls
```

- 숨김 파일 포함, 상세 정보까지 보기

```bash
ls -al
```

---

# 2. 폴더(디렉터리) 만들기 및 이동

## 새로운 폴더 만들기

- `mkdir`로 새로운 폴더 생성

```bash
mkdir linux_practice
```

- 중첩 폴더를 한번에 만들기 (`-p` 옵션)

```bash
mkdir -p linux_practice/src/utils
```

## 만든 폴더로 이동

```bash
cd linux_practice
```

## 상위 폴더로 이동하기

```bash
cd ..
```

---

# 3. 파일 만들기 및 내용 확인

## 빈 파일 만들기 (touch)

- `touch`로 빈 파일 생성

```bash
touch hello.txt
```

- `>`는 덮어쓰기, `>>`는 내용 추가
- 가급적이면 `>`는 사용 금지

```bash
echo "Hello, Linux!" > hello.txt
echo "두 번째 줄 추가" >> hello.txt
```

## 파일 내용 확인

- 파일 전체 내용을 터미널에 출력

```bash
cat hello.txt
```

## vi 편집기 기본 단축키

> 개발자 필수 / 앞으로 마크다운 작업할 때 vi 편집기 사용 추천!

| 키 | 동작 |
| --- | --- |
| `i` | 삽입 모드 진입 (현재 커서 앞) |
| `ESC` | 일반 모드로 복귀 |
| `:w` | 저장 |
| `:q` | 종료 |
| `:wq` | 저장 후 종료 |
| `:q!` | 저장 없이 강제 종료 |

---

# 4. 파일/폴더 복사, 이동, 삭제

## 파일 복사 (cp)

- `cp 원본 복사본` 형식

```bash
cp hello.txt hello_backup.txt
```

- 폴더 전체 복사 (`-r` 옵션)

```bash
cp -r linux_practice/ linux_practice_backup/
```

## 파일/폴더 이동 및 이름 변경 (mv)

- 같은 위치에서 이름만 바꾸면 이름 변경

```bash
mv hello_backup.txt hello_copy.txt
```

- 폴더 생성 후 파일 이동

```bash
mkdir archive
mv hello_copy.txt archive/
```

## 파일 삭제 (rm)

- 파일 삭제

```bash
rm 파일명.txt
```

- 폴더 전체 삭제 (`-r` 옵션)

```bash
rm -r archive
```

- 강제 삭제 (`-rf`) ⚠️ 주의! 복구 불가

```bash
rm -rf 폴더/파일/...
```
