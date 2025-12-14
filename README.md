# ⏱️ FPGA Smart Watch & Sensor System

> **Basys 3 FPGA를 활용한 디지털 시계, 스톱워치 및 UART 기반 센서(HC-SR04, DHT-11) 제어 시스템**

<br>

<div align="center">

![Device](https://img.shields.io/badge/Device-Basys3_(Artix--7)-78C922?style=for-the-badge&logo=microchip&logoColor=white)
![Language](https://img.shields.io/badge/Language-Verilog_HDL-007ACC?style=for-the-badge&logo=visualstudiocode&logoColor=white)
![Tool](https://img.shields.io/badge/Tool-Vivado-FF5252?style=for-the-badge&logo=xilinx&logoColor=white)
![Protocol](https://img.shields.io/badge/Protocol-UART_RS232-FF7F50?style=for-the-badge&logo=arduino&logoColor=white)

</div>

<br>

## 📖 1. 프로젝트 개요 (Project Overview)

이 프로젝트는 **Xilinx Artix-7 기반의 Basys 3 FPGA 보드**를 사용하여 구현된 **다기능 임베디드 시스템**입니다. 단순한 시간 계측을 넘어, 외부 센서 데이터 수집과 비동기식 시리얼 통신(UART)을 결합하여 하드웨어와 소프트웨어(PC) 간의 상호작용을 구현하는 데 중점을 두었습니다.

시스템은 크게 세 가지 핵심 모듈로 구성됩니다:
1.  **Timing Logic:** 100MHz 시스템 클럭을 정밀 분주하여 24시간제 시계와 1/100초 단위 스톱워치를 구동합니다.
2.  **Sensor Interface:** 초음파 거리 센서(HC-SR04)와 온습도 센서(DHT-11)의 고유 통신 프로토콜을 하드웨어 레벨에서 직접 구현하여 데이터를 실시간으로 수집합니다.
3.  **Communication System:** UART(RS-232) 인터페이스를 통해 PC 터미널에서 FPGA의 동작 모드를 원격 제어하거나, 센서 데이터를 PC로 전송합니다. 또한 **Circular FIFO**를 적용하여 데이터 송수신 간의 속도 차이를 완충하고 데이터 무결성을 확보했습니다.

<br>

## 🚀 2. 주요 기능 및 기술적 특징 (Key Features)

### 🕒 Dual-Mode Timekeeping (시간 관리)
* **Digital Watch:** 시(Hour), 분(Min), 초(Sec) 단위의 리얼타임 시계 (24H 포맷).
* **Precision Stopwatch:** 10ms(1/100초) 해상도의 정밀 타이머.
* **FSM Control:** `IDLE`, `RUN`, `PAUSE`, `CLEAR` 상태를 관리하는 유한 상태 머신(Finite State Machine) 설계로 안정적인 동작 제어.

### 📡 UART Remote Control (PC 통신)
* **Remote Commands:** PC 터미널(Tera Term, Putty 등)에서 키보드 입력을 통해 FPGA 제어 가능.
    * `M`: 모드 전환 (Watch ↔ Stopwatch)
    * `R`: 스톱워치 시작/정지 (Run/Stop)
    * `C`: 스톱워치 초기화 (Clear)
* **Data Buffering:** `fifo.v`를 통해 수신된 UART 데이터를 버퍼링하여 고속 데이터 처리 시 발생할 수 있는 데이터 유실 방지.

### 🌡️ Environment Sensing (센서 인터페이스)
* **HC-SR04 (Ultrasonic):** Trigger 신호 생성 및 Echo 펄스 폭 측정을 통해 거리 계산 로직 구현.
* **DHT-11 (Temp/Humidity):** 단일 선(Single-wire) 양방향 통신 프로토콜을 준수하여 온도 및 습도 데이터 파싱.
* **Visual Output:** 측정된 센서 값은 4-Digit 7-Segment Display(FND)를 통해 실시간으로 시각화.

<br>

## 🛠️ 3. I/O 맵핑 (Pin Configuration)

`.xdc` 제약 파일에 정의된 I/O 매핑입니다.

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

## 📂 4. 프로젝트 발표 자료 (Presentation)

프로젝트의 상세 설계 과정, 블록 다이어그램 및 시뮬레이션 결과는 아래 보고서에서 확인하실 수 있습니다.

<div align="center">

[![PDF Report](https://img.shields.io/badge/📄_PDF_Report-View_Document-FF0000?style=for-the-badge&logo=adobeacrobatreader&logoColor=white)](https://github.com/seokhyun-hwang/files/blob/main/UART_watch_stopwatch_HC-SR04_DHT-11.pdf)

</div>

<br>

## 📂 5. 프로젝트 디렉토리 구조 (Directory Structure)

```text
📦 FPGA-Watch-Sensor-System
 ┣ 📂 src
 ┃ ┣ 📜 watch_stopwatch_top.v   # [Top] 최상위 모듈 (System Wrapper)
 ┃ ┣ 📜 w_sw_uart_top.v         # [Sub] UART 서브시스템 (RX/TX/FIFO 통합 관리)
 ┃ ┣ 📜 clock_top.v             # [Sub] 시계 및 스톱워치 코어 로직
 ┃ ┣ 📜 stopwatch_cu.v          # [FSM] 스톱워치 제어 상태 머신
 ┃ ┣ 📜 stopwatch.v             # 스톱워치 데이터패스
 ┃ ┣ 📜 watch.v                 # 시계 데이터패스
 ┃ ┣ 📜 hc_sr04.v               # 초음파 센서 구동 드라이버
 ┃ ┣ 📜 dht11.v                 # 온습도 센서 구동 드라이버
 ┃ ┣ 📜 fifo.v                  # 데이터 버퍼링을 위한 원형 큐 (Circular FIFO)
 ┃ ┣ 📜 uart_rx.v               # UART 수신 모듈 (Serial to Parallel)
 ┃ ┗ 📜 uart_tx.v               # UART 송신 모듈 (Parallel to Serial)
 ┣ 📂 sim
 ┃ ┗ 📜 tb_test.v               # 전체 시스템 검증을 위한 Testbench
 ┗ 📂 constrs
   ┗ 📜 Basys-3-Master.xdc      # 물리적 핀 매핑 및 타이밍 제약 파일
````

<br>

## 🚀 6. 시작 가이드 (Getting Started)

1.  **환경 설정:** Xilinx Vivado (2018.x 이상 권장) 설치 및 Digilent Basys 3 보드 드라이버 설정.
2.  **프로젝트 생성:** Vivado에서 'RTL Project' 생성 후 `src` 폴더 내의 모든 Verilog 파일을 소스로 추가.
3.  **핀 매핑:** `Basys-3-Master.xdc` 파일을 Constraints에 추가 (보유한 센서의 핀 연결에 맞춰 수정 필요).
4.  **합성 및 구현:** `Generate Bitstream` 실행 후 USB를 통해 보드에 업로드.
5.  **기능 확인:**
      * **Stand-alone Mode:** `sw[0]`을 켜고 보드의 버튼을 눌러 시계/스톱워치 동작 확인.
      * **UART Mode:** PC와 USB 연결 후 터미널 프로그램(Baudrate: 9600)을 열고 `M`, `R`, `C` 키를 입력하여 원격 제어 확인.

<br>

-----

Copyright ⓒ 2025. SEOKHYUN HWANG. All rights reserved.

```
