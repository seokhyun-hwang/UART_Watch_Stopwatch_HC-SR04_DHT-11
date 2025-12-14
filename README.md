# 📡 FPGA Integrated UART Control System: Watch, Stopwatch & Sensors

<br>

**Dual-Mode Timing & Multi-Sensor Monitoring System**<br>
FSM 기반의 정밀 타이밍 제어와 FIFO 버퍼링을 적용한 UART 비동기 통신 및 환경 모니터링 시스템

-----

## 📋 1. 프로젝트 개요 (Overview)

본 프로젝트는 **Xilinx Artix-7 FPGA (Basys 3)** 보드를 활용하여 디지털 시계, 스톱워치 기능뿐만 아니라 **HC-SR04(초음파 거리 센서)** 및 **DHT-11(온습도 센서)** 를 통합 제어하는 임베디드 시스템입니다.

단순한 하드웨어 제어를 넘어, **PC 터미널과의 UART 직렬 통신**을 통해 시스템을 원격 제어하고 센서 데이터를 실시간으로 모니터링할 수 있도록 설계되었습니다. 특히 고속 데이터 입출력 시 안정성을 보장하기 위해 **Circular FIFO 구조**를 적용하여 데이터 유실 없는 통신 파이프라인을 구축했습니다.

<br>

-----

## 🛠 2. 개발 환경 (Environment)

| Category | Details |
|:---:|:---|
| **FPGA Board** | Digilent Basys 3 (Artix-7 XC7A35T-1CPG236C) |
| **Software** | Xilinx Vivado Design Suite 202x (WebPACK) |
| **Language** | Verilog HDL |
| **Sensors** | HC-SR04 (Ultrasonic), DHT-11 (Temp/Humidity) |
| **Communication** | UART RS-232 (Baud rate: 9600) |
| **Terminal Tool** | TeraTerm, PuTTY, or Serial Monitor |

<br>

-----

## 🔧 3. 시스템 아키텍처 (System Architecture)

### 3.1 하드웨어 블록 다이어그램 (H/W Block Diagram)

전체 시스템은 **입력 처리부**, **코어 로직(타이머/센서)**, 그리고 \*\*출력 제어부(FND/UART)\*\*로 구성됩니다.

```mermaid
graph TD
    User[User / PC] -->|Tact Switch| Btn[Debounce Logic]
    User -->|UART RX| FIFO_Rx[RX FIFO Buffer]
    
    Btn --> Control_Unit
    FIFO_Rx -->|Command Decoding| Control_Unit[Main Control Unit]
    
    Control_Unit -->|Enable/Mode| Watch[Digital Watch Logic]
    Control_Unit -->|Run/Stop/Clear| SW[Stopwatch Logic]
    Control_Unit -->|Trigger| Sensor_Ctrl[Sensor Controller]

    subgraph Sensors
        Sensor_Ctrl <-->|Trigger/Echo| HC[HC-SR04 Ultrasonic]
        Sensor_Ctrl <-->|1-Wire| DHT[DHT-11 Temp/Humid]
    end
    
    Watch --> MUX[Display Mux]
    SW --> MUX
    Sensor_Ctrl --> MUX
    
    MUX -->|Dynamic Scanning| FND[7-Segment Display]
    Sensor_Ctrl -->|Sensor Data| FIFO_Tx[TX FIFO Buffer]
    FIFO_Tx -->|UART TX| User
```

<br>

### 3.2 클럭 도메인 설계 (Clock Domain Design)

시스템은 $100\text{ MHz}$ 메인 클럭을 분주하여 다양한 서브 클럭을 생성하여 사용합니다.

  * **$100\text{ MHz}$:** 메인 시스템 클럭 (UART Baud Rate 생성 및 FIFO 제어)
  * **$1\text{ MHz}$:** DHT-11 및 초음파 센서 마이크로초($\mu s$) 단위 카운팅
  * **$1\text{ kHz}$:** 7-Segment Dynamic Scanning (잔상 효과 유도)
  * **$100\text{ Hz}$:** 스톱워치 및 시계 카운팅 기준 ($10\text{ ms}$ Resolution)

<br>

-----

## 💻 4. 핵심 기술 및 구현 상세 (Technical Details)

### 4.1 🛡️ UART 통신 및 FIFO 버퍼링 (Robust UART with FIFO)

비동기 통신(Asynchronous Communication)의 특성상 발생할 수 있는 데이터 타이밍 불일치 문제를 해결하기 위해 \*\*FIFO(First-In First-Out)\*\*를 도입했습니다.

  * **구조:** 8-bit Width, 4-Depth의 레지스터 기반 원형 큐(Circular Queue).
  * **RX Path:** `UART_RX` 모듈이 수신 완료(`rx_done`) 신호를 보내면 데이터를 RX FIFO에 `Push`하고, 커맨드 유닛이 준비되었을 때 `Pop`하여 해석합니다.
  * **TX Path:** 센서 데이터(거리, 온습도) 전송 시 TX 모듈이 Busy 상태일 경우, 데이터를 TX FIFO에 대기시켜 \*\*데이터 손실(Data Loss)\*\*을 방지합니다.

<br>

### 4.2 ⏱️ 유한 상태 머신 (FSM) 기반 제어

스톱워치와 센서 제어는 안정적인 상태 전이를 위해 **Moore Machine** 구조의 FSM으로 설계되었습니다.

  * **Stopwatch States:** `STOP` (대기), `RUN` (카운팅), `CLEAR` (초기화)
  * **Hybrid Input:** 물리 버튼(`Btn`)과 UART 명령어(`R`, `C` 등) 신호를 논리합(OR) 연산으로 병합하여, 하드웨어/소프트웨어 양쪽에서 제어가 가능합니다.
  * **Sensor States:** Trigger 신호 발생 $\rightarrow$ Echo 응답 대기 $\rightarrow$ 카운터 값 계산 $\rightarrow$ 데이터 변환 및 전송.

<br>

### 4.3 📟 다이내믹 디스플레이 (Dynamic Multiplexing)

  * **Digit Splitter:** 2진수(Binary) 시간 데이터 및 센서 값을 10진수(BCD)의 각 자릿수로 변환 (`% 10`, `/ 10` 연산 활용).
  * **Scanning:** 4개의 Anode(`fnd_com`)를 $1\text{ kHz}$ 속도로 순차 점멸시켜, 시각적으로 4자리가 동시에 켜진 것처럼 보이게 구현했습니다.

<br>

-----

## 🎮 5. 사용자 인터페이스 (User Interface)

### 5.1 UART 원격 제어 프로토콜

PC 터미널(TeraTerm, PuTTY) 접속 정보: **Baud Rate 9600, Data 8-bit, Parity None, Stop 1-bit**

| Command (ASCII) | Hex Code | Description | 동작 설명 |
|:---:|:---:|:---|:---|
| **M** | `0x4D` | **Mode Toggle** | 시계 ↔ 스톱워치 ↔ 센서 모드 전환 |
| **R** | `0x52` | **Run / Stop** | 스톱워치 시작 및 정지 토글 |
| **C** | `0x43` | **Clear** | 스톱워치 시간 초기화 (00:00) |
| **h** | `0x68` | **Hour Adj** | 시계 시간(Hour) +1 증가 |
| **m** | `0x6D` | **Min Adj** | 시계 분(Minute) +1 증가 |
| **s** | `0x73` | **Sec Adj** | 시계 초(Second) +1 증가 |

<br>

### 5.2 FPGA 물리 버튼 및 핀 매핑 (Pin Assignment)

`.xdc` 제약 파일에 정의된 I/O 매핑입니다. (센서는 PMOD 포트 사용 가정)

| Type | Pin Name | FPGA Pin | Function |
|:---:|:---:|:---:|:---|
| **Switch** | `sw[0]` | V17 | **System Enable** (Must be ON) |
| **Switch** | `sw[1]` | V16 | Mode Select (0: SW, 1: Watch) |
| **Button** | `Btn_R` | T17 | Stopwatch Run/Stop |
| **Button** | `Btn_L` | W19 | Stopwatch Clear / Watch Hour+ |
| **Button** | `Btn_U` | T18 | Watch Minute+ |
| **Button** | `Btn_D` | U17 | Watch Second+ |
| **Port** | `RsRx` | B18 | UART Receive (PC $\rightarrow$ FPGA) |
| **Port** | `RsTx` | A18 | UART Transmit (FPGA $\rightarrow$ PC) |
| **PMOD** | `JXADC` | (User Def) | HC-SR04 Trigger/Echo |
| **PMOD** | `JB` | (User Def) | DHT-11 Data Line |

<br>

-----

## 📄 6. 프로젝트 발표 자료 (Project Presentation)

프로젝트 발표 자료는 아래 버튼을 클릭하여 확인하실 수 있습니다.

[![PDF Report](https://img.shields.io/badge/📄_PDF_Report-View_Document-FF0000?style=for-the-badge&logo=adobeacrobatreader&logoColor=white)](https://github.com/seokhyun-hwang/files/blob/main/UART_watch_stopwatch_HC-SR04_DHT-11.pdf)

<br>

-----

## 📂 7. 프로젝트 디렉토리 구조 (Directory Structure)

```text
📦 FPGA-Watch-Sensor-System
 ┣ 📂 src
 ┃ ┣ 📜 watch_stopwatch_top.v   # [Top] 최상위 모듈
 ┃ ┣ 📜 w_sw_uart_top.v         # [Sub] UART 서브시스템 (RX/TX/FIFO 포함)
 ┃ ┣ 📜 clock_top.v             # [Sub] 시계/스톱워치/센서 로직 통합
 ┃ ┣ 📜 stopwatch.v             # 스톱워치 모듈
 ┃ ┣ 📜 watch.v                 # 디지털 시계 모듈
 ┃ ┣ 📜 hc_sr04.v               # 초음파 센서 드라이버
 ┃ ┣ 📜 dht11.v                 # 온습도 센서 드라이버
 ┃ ┣ 📜 fifo.v                  # Circular FIFO 버퍼
 ┃ ┣ 📜 uart_rx.v / uart_tx.v   # UART 드라이버
 ┃ ┗ 📜 stopwatch_cu.v          # FSM 제어기
 ┣ 📂 sim
 ┃ ┗ 📜 tb_test.v               # 시뮬레이션 Testbench
 ┗ 📂 constrs
   ┗ 📜 Basys-3-Master.xdc      # 물리적 핀 매핑 제약 파일
```

<br>

-----

## 🚀 8. 시작 가이드 (Getting Started)

1.  **환경 설정:** Xilinx Vivado 설치 및 Digilent Basys 3 보드 드라이버 설정.
2.  **프로젝트 생성:** Vivado에서 'RTL Project' 생성 후 `src` 폴더 내의 Verilog 파일 로드.
3.  **핀 매핑:** `Basys-3-Master.xdc` 파일을 Constraints에 추가 (센서 핀 번호 확인 필수).
4.  **합성 및 구현:** `Generate Bitstream` 실행 후 보드에 업로드.
5.  **기능 확인:**
      * **HW:** `sw[0]`을 켜고 버튼을 눌러 스톱워치 및 센서 값(FND 표시) 확인.
      * **SW:** PC 터미널을 열고 키보드 `M`, `R`, `C` 키를 입력하여 원격 제어 확인.

<br>

-----

Copyright ⓒ 2025. SEOKHYUN HWANG. All rights reserved.
