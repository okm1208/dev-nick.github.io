---
layout: post
title:  "Basic Shell Command"
date:   2021-05-23 17:00:00 +0900
categories: shell
tags: [linux, unix, shell, command, 리눅스, 명령어] 
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
<br>

## 2. zip / unzip 
-----  

### 2.1. zip 
- 파일을 압축 할 때 사용

#### 2.1.1. 옵션들
        -r  : 하위 디렉토리를 포함하는 압축 옵션 
        -x  : 특정 폴더 및 파일들을 제외 
        -1  : 빠른 압축 ( 압축률 저하 ) 
        -9  : 높은 압축률 ( 속도 저하 ) 
        -e  : zip 파일에 압호 설정

#### 2.1.2. 사용 예제
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

#### 2.2.1. 옵션들
        -l  : 압축을 해제 하지 않고 목록만 출력 
        -d  : 특정 폴더에 해제 

#### 2.2.2. 사용 예제
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
<br>

## 3. find 
-----  
- 파일 및 폴더를 검색 할때 사용하는 명령어

### 3.1. 표현식 (expression)
        -name       : 지정된 문자열 패턴에 해당 하는 파일 검색
        -empty      : 빈 디렉토리 또는 크기가 0인 파일 검색
        -delete     : 검색된 파일 또는 디렉토리 삭제 
        -exec       : 검색된 파일에 대해 지정된 명령 실행
        -type       : 지정된 파일 타입에 해당 하는 파일 검색 ( d : 디렉터리, f : 파일 )
        -mindepth   : 검색을 시작할 하위 디렉토리 최소 깊이
        -maxdepth   : 검색할 하위 디렉토리의 최대 갚이
        -atime      : 파일 점근(access) 시각을 기준으로 파일 검색 
        -ctime      : 파일 내용 및 속성 변경(change) 시각을 기준으로 파일 검색 
        -mtime      : 파일의 데이터 수정(modify) 시각을 기준으로 파일 검색 


### 3.2. 사용 예제

- 특정 파일 확장자 검색

 ```bash
 $ find . -name "*.yml"  # 현재 폴더 기준 하위 폴더 모두 
 $ find . -name -maxdepth 1 "*.yml" # 현재 폴더 기준 1 depth 까지 
 ```        
- 파일 검색 후 삭제 ( -delete 표현식 사용 )
 
 ```bash
 $ find . -name "*.yml" -delete 
 ```      
- 파일 검색 후 삭제 ( -exec 표현식 사용 )

 ```bash
 $ find . -name "*.yml" -exec rm {} + 
 ```      
- 파일 검색 개수 

 ```bash
 $ find . -name "*.yml" | wc -l
 ```      
- 파일 검색 후 수정일 변경 

 ```bash
 $ find . -name "*.yml" -exec touch -d '7 days ago' {} +  # 7일전으로 수정 
 ```      
<br>

## 4. head/tail 
-----  
- 파일의 앞 부분 및 뒷 부분을 읽는다

### 4.1. 옵션들
        -c  --bytes=SIZE    : 앞(뒤)에서 부터 N Kbyte 만큼의 문자를 출력 
        -n  --lines=N       : 앞(뒤)에서 부터 N 개의 라인을 출력한다.
        -q  --quiet         : 파일의 헤더 및 파일이름을 출력하지 않는다.
        -v  --verbose       : 파일의 헤더 및 파일이름을 출력한다.
        -f  --follow        :

### 4.2. 사용 예제

- 특정 파일의 정해진 라인수 만큼 출력 

 ```bash
 $ head -n 10 test.txt  # n 옵션 생략 가능
 $ tail -n 10 test.txt  # n 옵션 생략 가능 
 ```     
- 특정 파일의 정해진 라인수 만큼 계속 감시 하여 내용이 추가 될때 마다 출력

 ```bash
 $ tail -n 10 -f test.txt 
 ```     
<br>

## 5. cat 
-----  
- 파일 또는 표준 입력의 내용을 표준 출력에 출력하는 명령어이다

### 5.1. 옵션들
        -n  : 행 번호 출력  
        
### 5.2. 사용 예제

- 파일의 내용을 전체 출력

 ```bash
 $ cat test.txt 
 ```

- 파일 복사

 ```bash
 $ cat test.txt > copy_test.txt 
 ```

- 파일 병합

 ```bash
 $ cat test.txt test2.txt > merge_test.txt 
 ```

<br>

## 6. split 
-----  
- 하나의 파일을 여러 파일로 분리할때 사용한다.

### 6.1. 옵션들
        -a  : 나누어질 파일 이름의 길이   
        -d  : 숫자로된 파일 이름
        -l  : 정해진 라인 단위로 파일을 나눔 
        -b  : byte 단위로 파일을 나눔
        
### 6.2. 사용 예제

- 파일을 100라인 단위로 분리 

 ```bash
 $ split -l 100 test.txt # 분리하게 되면 해당 파일명은 임의의 앏파벳으로 구성 된다. 
 $ split -l 100 text.txt split_ # 접두사 지정 
 ```
<br>

## 7. ps 
-----  
- 현재 실행중인 프로세스의 목록을 출력 한다. 보통 grep과 많이 사용된다.

### 7.1. 옵션들
        -e  : 커널 프로세스를 제외한 모든 프로세스를 출력한다.
        -f  : 풀 포맷으로 보여준다. ( UID PID PPID C STIME TTY TIME CMD )
        -p  : 특정 PID를 지정할 수 있다. 
        -r  : 현재 실행 중인 프로세스를 보여준다.
        -u  : 특정 사용자의 프로세스를 확인 한다.

### 7.2. 항목 
        - PID       : 프로세스 ID 
        - UID       : 프로세스 소유자의 이름 
        - PPID      : 부모 프로세스 ID 
        - TIME      : 총 CPU 시간 
        - TTY       : 프로세스와 연결된 터미널 
        - COMMAND   : 프로세스의 실행 명령어
        - STIME     : 프로세스가 시작된 시간 
        - STAT      : 프로세스의 상태 코드  
        
### 7.3. 사용 예제

- 특정 프로세스 검색 

 ```bash
 $ ps -ef | grep '프로세스명'  # 대소문자를 구분 한다
 $ ps -ef | grep -i '프로세스명' # 대소문자를 구분 하지 않는다.
 ```
- 특정 PID 검색 

 ```bash
 $ ps -fp 'PID'
 ```
<br>

## 8. ls 
-----  
- 해당 디렉토리 내의 파일 혹은 디렉토리 목록을 출력 한다.

### 8.1. 옵션들
        -R  : 지정한 디렉토리 아래에 있는 하부디렉토리와 파일들을 모두 출력한다.
        -a  : 경로 안의 모든 파일들을 나열한다.
        -l  : 파일들을 나열 할때 자세한 정보를 출력한다.
        -h  : 파일 사이즈를 용량단위(MB,GB)를 붙여서 출력한다. 
        -d  : 지정한 디렉토리 내에 있는 파일들을 제외하고 디렉토리만 출력 한다.
        -S  : 파일 사이즈 정렬. 사이즈가 큰것 부터 출력한다.
        -t  : 파일의 수정 시간 정렬. 최근 수정된 것 부터 출력한다.

<br>

## 9. nohup 
-----
- 프로세스를 실행한 터미널의 세션이 끊기더라도 지속적으로 동작할 수 있게 해주는 명령어.

- 터미널에서 프로세스를 실행 시킨 후 세션을 종료하게 되면 해당 프로세스들에게 **HUP Signal** 이 전달되어 프로세스를 종료하게 된다.
이 Hup Signal을 무시하도록 해주는 명령어이다. 

- 해당 명령어는 **표준 출력(standard output)** 을 **nohup.out** 파일로 **재지향(redirection)** 하는 특징이 있다.


### 9.1. 사용 예제

- 특정 프로세스 벡그라운드 실행 
 ```bash
 $ nohup '실행할 프로세스' &
 $ nohup '실행할 프로세스' 1>/dev/null 2>&1 &  # 표준출력(1)과 표준에러(2) 를 /dev/null 로 재지향(redirection) 해준다.
 ``` 

<br>

## 10. kill / pkill 
-----  
- 프로세스를 종료하는 명령어.

- kill : PID 인자를 받아 프로세스를 종료 시킨다.

- pkill : 문자열 패턴을 이용하여 프로세스를 종료 시킨다.

### 10.1. 옵션들
        -l (kill)  : 시그널의 종류 출력 
        -f (pkill) : 지정한 패턴을 명령어뿐 아니라 경로명, 옵션, 아규먼트 등도 비교
        -n (pkill) : 패턴과 일치하는 프로세스의 가장 최근에 실행된 프로세스 하나만 종료
        -x (pkill) : 패턴과 정확하게 일치하는 프로세스만 종료
### 10.2. 사용 예제

- 특정 PID의 프로세스를 종료 

 ```bash
 $ kill -9 PID  # 프로세스를 강제로 종료 ( SIGKILL )
 $ kill -15 PID # 소프트웨어 종료 시그넝 ( SIGTERM )
 ``` 
- 특정 이름을 가진 프로세스를 종료 

 ```bash
 $ pkill -x 'mcp_ftp_bulk'
 ``` 

<br>

## 11. 그외 
-----  

- 폴더별 용량 산정  

 ```bash
 $ du -h --max-depth=1
 $ du -sh * 
 ```     
