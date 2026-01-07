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
#### - Constraint random을 통해 Coverage 향상
#### - Assertion : 실시간 프로토콜 검증
<br>

## 검증 환경 구성
### SystemVerilog의 클래스 기반 검증 환경을 UART_TX, UART_RX, FIFO, Loopback 각 단계별로 구축
#### - Interface: 하드웨어(DUT)와 검증 환경(Testbench) 사이의 신호를 연결하는 통로 역할 수행
#### - Transaction: 검증에 사용되는 데이터 단위로, 랜덤 데이터를 생성하여 다양한 시나리오 테스트
#### - Generator: 제약 조건을 기반으로 유효한 테스트 데이터 생성
#### - Driver: 생성된 데이터를 인터페이스를 통해 설계 모듈(DUT)에 입력 신호로 전달
#### - Monitor: DUT의 출력 신호를 관찰하고 이벤트 수집
#### - Scoreboard: 출력 데이터와 예상 결과값(Golden Model)을 비교하여 Pass/Fail 판별
<br>

## Module별 시나리오 
### UART_TX
<img width="442" height="356" alt="image" src="https://github.com/user-attachments/assets/f5413be5-4b1a-4eff-a25a-c210e8bd5938" />
<br>

#### transaction
- 랜덤 입력 데이터 -> 8bit tx_data
- monitor에서 감시를 위한 데이터 -> start, tx_busy, tx 
- log에서 출력할 display 테스크를 생성

#### generator
- randomize를 통해 랜덤 stimulus 생성
- 해당 랜덤값을 gen2drv를 통해 driver로 전달
- scoreboard에서 검증 완료 gen_next_event를 대기

#### driver
- gen2drv를 통해 generator에서 랜덤값 수신 
- start 신호가 1이 되며 전송 시작 및 dut로 tx_data 전달
- mon_next_event를 통해 monitor의 감시 시점을 제어

#### monitor
- driver의 mon_next_event 신호 대기
- 8bit의 receive_data를 수집한 뒤, 이를 mon2scb를 통해서 scoreboard로 전송

#### scoreboard
- monitor가 수집한 receive_data를 mon2scb를 통해 받은 후, 실제값 tx_data와 비교
- PASS / FAIL 여부를 판단하여 log에 출력
- 다음 tr 값 생성을 위해 gen_next_event를 generator로 전달

#### environment
- gen, drv, mon, scb 4개의 프로세스를 fork-join_any문을  통해 병렬 실행 
- 모든 전송 완료 후 최종 report를 출력
<br>

-------------------------------------------

### UART_RX
<img width="436" height="354" alt="image" src="https://github.com/user-attachments/assets/9db75e82-a939-4471-aa6a-3420fabe1c4e" />
<br>

#### transaction
- 랜덤 입력 데이터 -> 8bit wdata
- monitor에서 감시를 위한 데이터 -> 8bit rdata, rx_done
- log에서 출력할 display 테스크를 생성

#### generator
- randomize를 통해 랜덤 stimulus 생성
- 해당 랜덤값을 gen2drv를 통해 driver로 전달
- 마찬가지로 gen2scb를 통해 scoreboard로 전달
- scoreboard에서 검증 완료 gen_next_event를 대기

#### driver
- gen2drv를 통해 generator에서 랜덤값 수신  
- TX의 역할을 수행하는 drive_one_packet 테스크를 생성
- Task에서 생성한 8bit의 랜덤값 wdata를 dut로 전송

#### monitor
- rx_done의 posedge 신호를 대기
- 8bit의 실제값 rdata를 mon2scb를 통해서 scoreboard로 전송

#### scoreboard
- gen2scb를 통해 generator에서 예상값을 수신
- mon2scb를 통해 monitor에서 실제값을 수신
- 두 값을 비교하여 PASS / FAIL 여부를 판단한 뒤 log에 출력
- 다음 tr 값 생성을 위해 gen_next_event를 generator로 전달

#### environment
- gen, drv, mon, scb 4개의 프로세스를 fork-join_any문을  통해 병렬 실행 
- 모든 전송 완료 후 최종 report를 출력
<br>

-------------------------------------------

### FIFO
<img width="441" height="354" alt="image" src="https://github.com/user-attachments/assets/0383e4ae-7ac6-475e-b613-94cc2fc33767" />
<br>

#### transaction
- 랜덤 입력 데이터 -> wr, rd, 8bit wdata
- monitor에서 감시를 위한 데이터 -> 8bit rdata, full, empty
- constraint를 통해 wr 1 : 80% / 0 : 20%로 설정
- log에서 출력할 display 테스크를 생성

#### generator
- rand 변수들을 randomize를 통해 랜덤 stimulus 생성
- 해당 랜덤값을 gen2drv를 통해 driver로 전달
- scoreboard에서 검증 완료 gen_next_event를 대기

#### driver
- gen2drv를 통해 랜덤값 수신 
- dut로 wr, rd, wdata 전달
- mon_next_event를 통해 monitor의 감시 시점을 제어

#### monitor
- driver의 mon_next_event 신호 대기
- driver의 입력 신호 wr, rd, wdata와 dut의 출력 rdata, full, empty를 transaction에 저장하여 감시할 데이터 생성

#### scoreboard
##### - write 동작 처리
- 최대 인덱스가 15인 queue를 선언
- wr 신호가 1이면서, full이 아니면 push_back을 통해 맨 뒤에 값을 추가하며, pass_count++
- display를 통해 data와 size를 log에 출력
##### - read 동작 처리
- rd 신호가 1이면서, empty가 아니면 pop_front를 통해 맨 앞에서부터 데이터를 내보내며, rdata와 expected_data를 비교하여 다를 경우 fail_count++ 
- display를 통해 rdata와 expected_data를 log에 출력

#### environment
- gen, drv, mon, scb 4개의 프로세스를 fork-join_any문을  통해 병렬 실행 
- 모든 전송 완료 후 최종 report를 출력
<br>

-------------------------------------------

### UART-FIFO Loopback
<img width="440" height="356" alt="image" src="https://github.com/user-attachments/assets/5844272d-a883-4eb4-90c9-d55fb4a552f3" />
<br>

#### transaction
- 랜덤 입력 데이터 -> 8bit wdata
- monitor에서 감시를 위한 데이터 -> 8bit rdata
- log에서 출력할 display 테스크를 생성

#### generator
- randomize() 를 통해 random data 값 생성
- 생성된 랜덤값을 gen2drv_mbox를 통해 driver전달
- gen2scb_mbox를 통해 scoreboard로 전달

#### driver
- gen2drv_mbox .get을 통해서 generator로부터 데이터를 받은 후, tr에 저장
- tr.wdata를 uart 프로토콜에 맞는 시리얼 신호로 변환하여 전송

#### monitor
- tx 신호가 1에서 0으로 떨어지는 시점, 즉 시작을 알리는 start bit를 기다림
- 8개의 데이터 비트를 순서대로 샘플링
- 샘플링된 데이터를 mon2scb_mbox에 .put 하여 전달 

#### scoreboard
- top_transaction expected_q[$]; -> transaction을 저장할 공간 선언
- gen2scb_mbox에 .get을 통해 예상 데이터 expected_tr 저장 후 push_back을 통해 맨 뒤에 추가
- expected_q의 저장 공간이 비어 있지 않을 때 비교 시작
- 비교 시작 전 pop_front를 통해 예상값 추출 후 expected_tr에 전달 * expected 예상값과 actual 실제값이 동일 하면 pass 처리, 값이 다르면 fail 처리
- 비교가 끝이 나면 비교가 끝나는 것을 알리기 위해 last_item_compared라는 이벤트를 발생

#### environment
- new() 함수를 통해 mailbox를 각각 생성 및 연결8개의 데이터 비트를 순서대로 샘플링
- gen, drv, mon, scb 4개의 컴포넌트를 병렬로 동시 실행
- last_item_compared 이벤트를 통해 비교를 끝난 것을 확인 후 report를 통해 최종 결과 출력 
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
