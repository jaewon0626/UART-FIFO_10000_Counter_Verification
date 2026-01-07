# UART-FIFO_10000_Counter_Verification
### SystemVerilog를 통합 언어(설계+검증)로 사용하여 UART 통신 및 FIFO 모듈의 기능을 객체 지향 프로그래밍(OOP) 기반의 검증

## 프로젝트 개요
### SystemVerilog Verification의 특징
#### - 통합 언어 : 설계 + 검증을 하나의 언어로 가능함
#### - OOP(객체지향 프로그래밍)
- 캡슐화 : 데이터(속성)와 메서드(기능)을 한 번에 관리 -> 유지보수 용이
- 추상화 : 내부의 디테일한 기능을 몰라도 사용 가능(=모듈화)
- 다형성 : 상위 개념(부모 class)를 재정의하여 대입
- 상속 : 재사용성 + 새로운 기능 -> 확장성이 좋아짐
#### Constraint random을 통해 Coverage 향상
#### Assertion : 실시간 프로토콜 검증
<br>

## 검증 환경 구성
### SystemVerilog의 클래스 기반 검증 환경을 UART_TX, UART_RX, FIFO, Loopback 각 단계별로 구축
#### Interface: 하드웨어(DUT)와 검증 환경(Testbench) 사이의 신호를 연결하는 통로 역할 수행
#### Transaction: 검증에 사용되는 데이터 단위로, 랜덤 데이터를 생성하여 다양한 시나리오 테스트
#### Generator: 제약 조건을 기반으로 유효한 테스트 데이터 생성
#### Driver: 생성된 데이터를 인터페이스를 통해 설계 모듈(DUT)에 입력 신호로 전달
#### Monitor: DUT의 출력 신호를 관찰하고 이벤트 수집
#### Scoreboard: 출력 데이터와 예상 결과값(Golden Model)을 비교하여 Pass/Fail 판별
<br>

## Module별 시나리오 
### UART_TX
<img width="442" height="356" alt="image" src="https://github.com/user-attachments/assets/f5413be5-4b1a-4eff-a25a-c210e8bd5938" />
<br>
transaction
- 랜덤 입력 데이터 -> 8bit tx_data
- monitor에서 감시를 위한 데이터 -> start, tx_busy, tx 
- log에서 출력할 display 테스크를 생성

<br>

### UART_RX
<img width="436" height="354" alt="image" src="https://github.com/user-attachments/assets/9db75e82-a939-4471-aa6a-3420fabe1c4e" />
<br>

### FIFO
<img width="441" height="354" alt="image" src="https://github.com/user-attachments/assets/0383e4ae-7ac6-475e-b613-94cc2fc33767" />
<br>

### UART-FIFO Loopback
<img width="440" height="356" alt="image" src="https://github.com/user-attachments/assets/5844272d-a883-4eb4-90c9-d55fb4a552f3" />
<br>

## Top Block Diagram
<img width="2118" height="1047" alt="image" src="https://github.com/user-attachments/assets/2b6dde43-9128-4a64-892e-e5c175ea08f1" />
<br>

## 특징
### 1. 동작 방식
#### 모든 명령어가 정확히 하나의 클록 사이클에 완료된다.
#### Fetch → Decode → Execute → Memory → Write Back 단계가 한 사이클 내에 순차적으로 진행된다.
<img width="928" height="209" alt="image" src="https://github.com/user-attachments/assets/6fe3d7a3-19eb-4a4f-bbdf-f23036e27153" />
<br>

### 2. 주요 특징
#### 클록 주기 : 가장 느린 명령어(일반적으로 load)의 실행 시간에 맞춰 클록 주기가 결정된다.
#### 하드웨어 구조 : 각 단계마다 별도의 하드웨어 유닛이 필요하다. (예: 별도의 명령어 메모리와 데이터 메모리, 여러 개의 ALU)
#### CPI (Cycles Per Instruction) : 항상 1이다.
#### 제어 유닛 (Control Unit)의 특징 :
##### Combinational Logic: 멀티 사이클 프로세서가 FSM(Finite State Machine)을 사용하는 것과 달리, 싱글 사이클은 현재 명령어의 Opcode와 Funct 필드만 보고 즉시 모든 제어 신호(ALUOp, RegWrite, MemRead 등)를 생성하는 조합 회로로 구성된다. -> 단순하고 직관적임
#### 데이터패스(Datapath) :
##### - IF (Instruction Fetch): PC(Program Counter)가 가리키는 주소의 명령어 메모리에서 명령어를 가져옵니다. 동시에 PC는 PC + 4로 업데이트된다.
##### - ID (Instruction Decode): 가져온 명령어를 해석하여 레지스터 파일에서 소스 레지스터(rs1, rs2) 값을 읽고, Control Unit이 제어 신호를 생성한다.
##### - EX (Execute): ALU가 연산을 수행하거나, 주소 계산(Load/Store의 경우), 분기 조건 비교(Branch의 경우)를 수행한다.
##### - MEM (Memory Access): Load/Store 명령어인 경우 데이터 메모리에 접근하여 값을 읽거나 쓰기 동작을 수행한다. (R-type 등은 이 단계 패스)
##### - WB (Write Back): 연산 결과나 메모리에서 읽은 값을 레지스터 파일(rd)에 쓴다.
<br>

### 3. 장단점
#### [장점]
##### 구현이 단순하고 이해하기 쉽다.
##### 제어 로직이 간단하다.
##### 예측 가능한 타이밍을 가진다.

#### [단점]
##### 클록 주기가 길어 전체 성능이 낮다.
##### 하드웨어 활용도가 낮다. (각 사이클마다 일부 유닛만 사용)
##### 자원 낭비가 심함. (ALU, 메모리 등이 중복 배치)
