---
layout: single
title: Process Management
categories: Linux
tags: [Linux, Uni]
author_profile: false
search: true
use_tex: false
---

> 프로세스 매니지먼트

<br>

# Programs, Process Threads 의 정의

---

---

### Programs
- 하드디스크에 적재된 실행 가능한 코드

<br>

### Process
- 실행중인 프로그램.
- 바이너리 이미지, 메모리 적재가 되어있음

<br>

### Thread
- 프로세스 내부에서 활동중인 유닛 (실제로 작업을 수행하는 코드)
- 스레드는 프로그램(프로세스) 내에서 실행되는 독립적인 흐름으로, 멀티스레딩 환경에서 여러 스레드가 동시에 실행될 수 있다.
- 각 스레드는 자체 "가상화된 프로세서"를 가지고 있는데, 이는 스레드가 독립적으로 실행 상태를 관리하고, 자신만의 스택, 레지스터, 명령 포인터 등을 가지고 있음을 의미 
- 이러한 특성 덕분에 스레드는 병렬 처리와 동시성을 가능

<br>

# The Process ID

---

---

### Definition
- PID는 각 프로세스를 대표하는 고유 식별자이다. 
- 운영 체제는 이 번호를 사용하여 프로세스를 관리하고, 사용자와 다른 시스템 프로세스는 이 번호를 사용하여 특정 프로세스를 참조한다.
  - 한 시점에 각 PID는 고유합니다. 즉, 시스템 상에서 동시에 두 개 이상의 프로세스가 동일한 PID를 가질 수 없다. 
  - 프로세스가 종료되면, 그 프로세스의 PID는 재사용될 수 있다.

<br>

### The idle process
- 이는 시스템에서 항상 실행 중인 특별한 프로세스로, 다른 실행 가능한 프로세스가 없을 때 커널이 실행하는 프로세스
- 아이들 프로세스는 항상 PID 0을 가지고 있다. 
- 이 프로세스는 시스템이 다른 작업을 수행할 것이 없을 때 실행되며, 일반적으로 시스템의 CPU 사용 시간을 최소화하는 데 사용한다.

<br>

### The init process
- he First Process Executed by the Kernel: init 프로세스는 시스템이 부팅된 후 커널이 처음으로 실행하는 프로세스이다.
- 이는 시스템의 모든 다른 프로세스의 부모 프로세스로 간주되며, PID 1을 가진다.

- Kernel’s Decision on the init Process: 사용자가 커널에게 명시적으로 어떤 프로세스를 실행할 것인지 지시하지 않는 한(예: init 커널 명령줄 매개변수를 통해), 커널은 스스로 적절한 init 프로세스를 식별해야 합니다.

<br>

### Process ID Allocation
- 커널은 프로세스에게 PID를 할당하며, 이전 프로세스가 죽더라도 순차적으로 번호를 지정한다.
- 커널은 최상단에서 감싸지 않는 한 프로세스 ID 값을 재사용하지 않는다. 즉, /proc/sys/kernel/pid_max에 있는 값이 할당될 때까지 이전 값들은 재사용되지 않는다.

<br>

### The Process Hierarchy
#### 부모-자식 관계:
- 프로세스는 '부모'와 '자식'의 관계를 가진다.
- 모든 프로세스(초기 init 프로세스 제외)는 다른 프로세스로부터 생성되며, 이에 따라 부모-자식 관계가 형성된다.
- 이러한 관계는 부모 프로세스 ID(ppid)에 기록되며, 이는 자식 프로세스가 부모 프로세스를 식별할 수 있게 한다.

<br>

#### 사용자와 그룹 소유권:
- 각 프로세스는 특정 사용자와 그룹에 속한다.
- 이 소유권은 프로세스가 시스템 리소스에 접근하는 권한을 제어하는 데 사용된다.
- 자식 프로세스는 부모 프로세스의 사용자와 그룹 소유권을 상속받는다.

<br>

#### 프로세스 그룹:
- 프로세스는 프로세스 그룹의 일원이다.
- 프로세스 그룹은 사용자/그룹 개념과는 별개이다.
- 자식 프로세스는 일반적으로 부모와 동일한 프로세스 그룹에 속한다.
- 셸이 파이프라인을 생성하면, 파이프라인의 모든 명령은 동일한 프로세스 그룹에 속한다.
- 프로세스 그룹은 그룹 내의 모든 프로세스에게 신호를 보내거나 정보를 얻는 것을 용이하게 한다.

<br>

#### 요약:
- 프로세스 계층 구조는 부모-자식 관계와 사용자/그룹 소유권, 프로세스의 개념을 포함한다. 
- 부모 프로세스는 자식 프로세스를 생성하고, 자식 프로세스는 부모의 소유권과  <span style='color:orange'>그룹 소유권을 상속</span>받는다. 
- 프로세스 그룹은 관련 프로세스들을 묶어 신호 전송이나 정보 추출을 쉽게 만들어준다.

<br>

### pid_t
- 추가해야함

<br>

# Running a New Process

---

---

### Executing a Program
- 프로세스는 cpu 자원을 할당받고, i/o 상태이거나 유휴상태일 때 자원을 반납. 준비상태가 되면 다시 자원을 할당받아 프로세스 실

<br>

### Creating a New Process
- A near-duplication its parent process (거의 복제하는 것)
  - 부모프로세스의 영역을 카피해서 프로세스를 생성하니 시간이 오래걸린다!
  - 그래서, 자식프로세스는 부모 프로세스에 접근 권한을 부여하여 효율성 증대
    - 메모리를 공유한다. 프로그램 카운터만 변경
  - 그러다 실제로 복사할 때도 있다: 프로세스를 수정해야 할 때 

<br>

### Exec Family of Calls
- 즉시 실행하는 경우도 있다. 다른 프로세스를 실행하기 위한 execl 시스템 콜을 사용해서 
- 보충해라잉
- 뭔가 중요해 보인다이

<br>

#### Execv

<br>

### The fork() system call
- 자식프로세스에서 부포프로세스를 또 실행하지 않게 하기 위해 if 분기. pid를 이용해 구분?

