---
layout: post
title:  "Basic Shell Command"
date:   2021-05-23 17:00:00 +0900
categories: shell
tag: [linux, unix, shell, command, 리눅스, 명령어] 
---

서버를 관리하면서 자주 사용 하는 shell 명령어 들을 정리해 보자.

## 1. tar (Tape Archiver)
-----  

- 여러개의 파일을 하나로 묶거나 묶인 파일을 여러개의 파일로 풀 때 사용 하는 명령어 이다.
- 엄밀히 말하면 tar은 압축 명령어가 아닌 여러 파일들을 하나로 묶는 용도에 쓰인다.
- 압축을 같이 진행하려면 z 옵션을 활용하자 ( gzip )

### 1.1. 옵션들
        -f  : 대상 tar 아카이브 지정. ( 기본 옵션 )
        -c  : tar 아카이브 생성. 기존 아카이브 덮어 쓰기 
        -x  : tar 아카이브에서 파일 추출
        -v  : 처리되는 과정을 콘솔로 보여줌
        -z  : gzip 압축 적용 옵션
        -t  : tar 아카이브에 포함된 내용 확인
        -j  : bzip2 압축 적용 옵션 
        -C  : 대상 디렉토리 경로 지정
        -k  : tar 아카이브 추출 시, 기존 파일 유지.
        -U  : tar 아카이브 추출 전, 기존 파일 삭제

### 1.2. 사용 예제

- 단일 파일 묶음 

```bash
$ tar cvf text.tar text.txt
```
- 여러 파일 묶음 

```bash
$ tar cvf text.tar text.txt text2.txt # 각각 파일 나열  
$ tar csv text.tar *.txt # 현재 폴더의 모든 txt 파일
```
- 파일 & 폴더 묶음

```bash
$ tar cvf text.tar test_folder text.txt  # 각각 파일 및 폴더를 나열   
```
- 파일 묶음 & 압축 

```bash
$ tar cvfz text.tar.gz text.txt
```
- 파일 추출
 
```bash
$ tar xvf text.tar #현재 폴더에 그대로 추출  
```
- 지정된 경로에 파일 추출 

```bash
$ tar xvf text.tar -C  target_folder
```
- 아카이브 파일 내용 확인 

```bash
$ tar tf text.tar
```


## 2. zip / unzip 
-----  

### 2.1. zip 
- 파일을 압축 할 때 사용

### 2.1.1 옵션들
        -r  : 하위 디렉토리를 포함하는 압축 옵션 
        -x  : 특정 폴더 및 파일들을 제외 
        -1  : 빠른 압축 ( 압축률 저하 ) 
        -9  : 높은 압축률 ( 속도 저하 ) 
        -e  : zip 파일에 압호 설정

- 파일 압축

 ```bash
 $ zip text.zip text.txt test_folder
 ```
- 하위 폴더 압축 

 ```bash
 $ zip -r text.zip text.txt test_folder
 ```

### 2.2. unzip 
- zip 파일의 압축을 해제 할 때 사용 

### 2.2.1 옵션들
        -l  : 압축을 해제 하지 않고 목록만 출력 
        -d  : 특정 폴더에 해제 
        
- 압축 해제 

 ```bash
 $ upzip text.zip
 ```
- 특정 폴더 압축 해제 

 ```bash
 $ upzip text.zip -d target_folder
 ```
- 여러 파일 압축 해제

 ```bash
 $ upzip *.zip -d target_folder
 ```
- 압축 파일 내부 목록 확인

 ```bash
 $ upzip -l text.zip
 ```

