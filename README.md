# 🎛️ FPGA Integrated Smart System (Watch & Sensors)

\<img src="[https://img.shields.io/badge/Language-Verilog-blue?style=for-the-badge\&logo=verilog](https://img.shields.io/badge/Language-Verilog-blue?style=for-the-badge&logo=verilog)" /\>
\<img src="[https://img.shields.io/badge/Tool-Vivado-red?style=for-the-badge\&logo=xilinx](https://img.shields.io/badge/Tool-Vivado-red?style=for-the-badge&logo=xilinx)" /\>
\<img src="[https://img.shields.io/badge/Board-Basys3-green?style=for-the-badge\&logo=fpga](https://img.shields.io/badge/Board-Basys3-green?style=for-the-badge&logo=fpga)" /\>

**초음파/온습도 센서 모니터링과 디지털 타이머의 결합**<br>
UART 통신을 통해 모든 기능을 PC에서 원격 제어하고 데이터를 수신하는 FPGA 통합 제어 시스템입니다.

\</div\>

-----

## 📖 프로젝트 개요 (Project Overview)

이 프로젝트는 **Xilinx FPGA**를 활용하여 **스마트 센서 계측(거리, 온습도)** 기능과 **정밀 타이밍(시계, 스톱워치)** 기능을 하나로 통합한 시스템입니다.
[cite_start]물리적인 버튼 제어뿐만 아니라, **UART 시리얼 통신**을 통해 PC 터미널에서 모드를 전환하거나 스톱워치를 제어할 수 있습니다[cite: 161, 166]. [cite_start]안정적인 데이터 전송을 위해 **FIFO 버퍼링** 기술이 적용되었습니다[cite: 112].

-----

## 🚀 주요 기능 (Key Features)

### 1️⃣ ⏱️ UART 디지털 시계 & 스톱워치 (Watch & Stopwatch)

  * [cite_start]**Dual Mode:** 실시간 시계 모드(Watch)와 1/100초 정밀도의 스톱워치(Stopwatch) 모드를 지원하며, 실시간 전환이 가능합니다[cite: 80, 81, 169].
  * [cite_start]**Remote Control:** PC 키보드 입력을 통해 시간 설정(시/분/초) 및 스톱워치 동작(Run/Stop/Clear)을 제어합니다[cite: 192].
  * [cite_start]**Dynamic Display:** 4자리 FND를 시분할 구동(Multiplexing)하여 시각을 표시합니다[cite: 6, 16].

### 2️⃣ 📏 스마트 센서 계측 (Smart Sensing)

  * **초음파 거리 측정 (Ultrasonic):** HC-SR04 센서를 활용하여 물체와의 거리를 측정하고, 측정된 거리 데이터를 UART로 PC에 전송합니다.
  * **온습도 모니터링 (Env Monitor):** DHT11 센서를 통해 주변 환경의 온도와 습도를 실시간으로 계측합니다.
  * **실시간 모니터링:** 계측된 센서값은 FPGA 내부에서 ASCII 코드로 변환되어 터미널 프로그램에 텍스트로 출력됩니다.

### 3️⃣ 📡 견고한 통신 시스템 (Robust Communication)

  * [cite_start]**FIFO Buffering:** 송수신(Rx/Tx) 라인에 8-bit 원형 큐(FIFO)를 적용하여 데이터 유실 없는 안정적인 통신을 보장합니다[cite: 112, 177].
  * [cite_start]**Command Decoder:** 수신된 ASCII 명령어를 해석하여 시스템의 상태(State)와 모드(Mode)를 제어합니다[cite: 182, 192].

-----

## 🛠️ 시스템 아키텍처 (System Architecture)

시스템은 `Top Module`을 중심으로 센서부와 타이머부가 병렬로 구성되어 있으며, UART 모듈이 중앙에서 통신을 중개합니다.

| 모듈 (Module) | 역할 (Role) | 상세 설명 |
| :--- | :--- | :--- |
| **watch\_stopwatch\_top** | 최상위 제어 | [cite_start]버튼 및 UART 입력을 통합하고, 기능별 모듈(시계/센서)을 활성화합니다[cite: 161]. |
| **w\_sw\_uart\_top** | 통신 제어 | [cite_start]UART Rx/Tx, FIFO, Baud Rate 생성, 명령어 해석(Command Unit)을 담당합니다[cite: 175]. |
| **clock\_top** | 타이머 로직 | [cite_start]시계(`watch`)와 스톱워치(`stopwatch`) 로직을 포함하며, FND 출력을 관리합니다[cite: 77]. |
| **sensor\_top** | 센서 제어 | *(통합 예정)* HC-SR04 트리거 제어 및 Echo 펄스 계측, DHT11 프로토콜 처리. |
| **fifo** | 데이터 버퍼 | [cite_start]비동기 데이터 처리를 위한 4-depth 원형 큐 메모리 구조입니다[cite: 112, 116]. |

-----

## 🎮 조작 방법 (Controls)

시스템은 **UART 터미널 명령어**와 **보드 물리 버튼**을 동시에 지원합니다. (`sw[0]` Enable 필수)

### 1\. UART 키보드 명령어 (PC Input)

[cite_start]터미널(9600bps) 연결 후 아래 키를 입력하여 제어합니다[cite: 204].

| Key (ASCII) | Hex | 기능 (Function) | 설명 |
| :---: | :---: | :--- | :--- |
| **M** | `0x4D` | **Mode Change** | 시계 ↔ 스톱워치 (또는 센서모드) 전환 |
| **R** | `0x52` | **Run / Stop** | 스톱워치 시작 및 일시정지 |
| **C** | `0x43` | **Clear** | 스톱워치 리셋 |
| **h** | `0x68` | **Hour Up** | 시계 시간(Hour) 증가 |
| **m** | `0x6D` | **Min Up** | 시계 분(Min) 증가 |
| **s** | `0x73` | **Sec Up** | 시계 초(Sec) 증가 |

### 2\. 물리 버튼 및 스위치 (Basys3 I/O)

[cite_start]`.xdc` 파일 기준 핀 매핑입니다[cite: 97].

  * **SW[0] (Enable):** 전체 시스템 전원 ON/OFF.
  * **SW[1] (Mode):** 시계/스톱워치 모드 선택 토글.
  * **Btn\_R:** 스톱워치 Run/Stop.
  * **Btn\_L:** 스톱워치 Clear / 시계 Hour 설정.
  * **Btn\_U/D:** 시계 Min/Sec 설정.

-----

## 💾 설치 및 실행 (Installation)

1.  **하드웨어 연결:**
      * Basys3 보드를 PC에 연결합니다.
      * **HC-SR04 / DHT11** 센서를 지정된 Pmod 포트(JA/JB 등)에 연결합니다.
2.  **Vivado 프로젝트:**
      * 소스 코드(`.v`)와 제약 파일(`.xdc`)을 프로젝트에 추가합니다.
3.  **합성 및 업로드:**
      * Synthesis & Implementation 후 Bitstream을 생성하여 보드에 올립니다.
4.  **터미널 접속:**
      * TeraTerm 또는 PuTTY를 실행합니다.
      * **Port:** 보드 포트, **Baud Rate:** 9600, **Data:** 8bit, **Parity:** None.

-----

\<div align="center"\>
\<i\>Created with Verilog HDL on Xilinx Vivado\</i\>
\</div\>
