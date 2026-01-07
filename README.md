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

## 검증 결과
<img width="1546" height="772" alt="image" src="https://github.com/user-attachments/assets/8e1823d9-596b-47c6-bdb7-72a5343bfa7b" />

### UART_TX/RX
- 모든 가능한 데이터 값(0~255)을 최소 1번씩 관찰하여 Coverage를 확보했으며, Start/Stop 비트가 정상적으로 동작함을 Waveform으로 확인

### FIFO
- 공간이 가득 찬 상태(Full)와 비어 있는 상태(Empty)에서의 동작 검증
- 가득 찬 상태에서 쓰기 시도 시 "Queue is full" 메시지를 출력하며 데이터 오버플로우를 방지

### Top Simulation
- 보드 제어 : 버튼 입력을 통해 Up/Down 모드 변경, 현재값 Clear, 전체 초기화 기능이 정상 작동 확인
- UART 제어: PC에서 ASCII 명령어('r', 'c', 'm')를 전송하여 카운터를 원격 제어하는 Loopback 동작을 성공적으로 시뮬레이션함

### 결론: 총 200번 이상의 다양한 랜덤 테스트케이스를 수행하여 100% 성공(Pass)
<br>

## 트러블슈팅
### 문제점: UART_TX 검증 단계에서 테스트벤치와 모니터 간의 동기화 문제로 인해 출력 데이터를 정확하게 캡처하지 못하는 현상 발생
#### 원인 분석: 모니터링 시점이 실제 데이터 변화 시점보다 빠르거나 늦어 이벤트 시퀀스가 어긋나는 타이밍 이슈
#### 해결 방안: UART_TX 테스트벤치에 mon_next_event를 추가하여 모니터가 다음 유효 신호를 기다리도록 동기화 로직을 보강
#### FIFO 예외 처리: FIFO가 Full인 상태에서 Write 신호가 들어올 경우(wr=1 & full=1), 무시하고 예외 메시지를 출력하도록 제약 조건을 설정하여 시스템 안정성을 높였습니다.
