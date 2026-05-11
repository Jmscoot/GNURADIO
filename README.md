# GNU Radio 디지털/아날로그 변조 실습

GNU Radio를 이용한 SSB, BPSK, QPSK, 8-PSK 변조/복조 구현 실습 프로젝트입니다.  
시뮬레이션 환경과 PlutoSDR를 이용한 실제 RF 전송까지 포함합니다.

---

## 목차

- [프로젝트 구조](#프로젝트-구조)
- [구현 항목](#구현-항목)
- [주요 개념](#주요-개념)

---

## 프로젝트 구조

```
.
├── SSB/                  # Single Sideband 변조 (마이크 입력 → BPF → upper/lower 선택)
├── BPSK/                 # BPSK TX 시뮬레이션 (GNU Radio Constellation Modulator)
├── BPSK_직접구현/         # BPSK TX/RX with PlutoSDR (Custom polar NRZ encoding)
├── QPSK/                 # QPSK TX/RX 시뮬레이션 (Symbol Sync + Costas Loop)
└── 8PSK/                 # 8-PSK TX/RX + BER 측정
```

---

## 구현 항목

### SSB (Single Sideband Modulation)
- 마이크 오디오 소스 → BPF(300Hz~3.5kHz) → DSB-SC → upper/lower sideband 선택
- Swap IQ 블록으로 USB/LSB 전환
- tx/rx sideband 불일치 시 복조 불가 확인

### BPSK (Binary Phase Shift Keying)
- GNU Radio Constellation Modulator 사용
- Differential Encoding, RRC 필터(α=0.35), sps=4
- Polyphase Clock Sync + Costas Loop 기반 RX 동기화
- Channel Model(AWGN + freq offset + timing offset) 실험

### BPSK 직접 구현 (with PlutoSDR)
- 이진수 0,1 → polar NRZ(-1, +1) 인코딩 → DSB-SC 변조
- TX RRC(pulse shaping) + RX RRC(matched filter)로 ISI 제거
- Symbol Sync(TED: y[n]y'[n] ML) + Costas Loop로 timing/carrier 동기화
- PlutoSDR 2.4GHz 반송파, samp_rate=1MHz, sps=10

### QPSK (Quadrature Phase Shift Keying)
- Gray coding: 00→0, 01→1, 11→3, 10→2
- Constellation Points: ±0.707±0.707j
- Channel Model로 noise/freq offset/timing offset 주입 후 Symbol Sync 동작 확인
- Eye diagram으로 timing offset 영향 시각화

### 8-PSK
- 8개 constellation point, 3bits/symbol
- BER(Bit Error Rate) 측정 블록 포함

---

## 주요 개념

| 개념 | 설명 |
|---|---|
| **RRC Filter** | ISI 제거용 pulse shaping 필터. TX/RX에 각각 1개씩 배치해 전체적으로 RC filter 특성을 만듦 |
| **Excess BW (α)** | Roll-off factor. 나이퀴스트 최소 대역폭 대비 추가 대역폭 비율. 보통 0.35 사용 |
| **Symbol Sync** | Clock recovery 블록. TED(Timing Error Detector) + PI 제어기로 타이밍 오프셋 보정 |
| **Costas Loop** | Carrier phase synchronization. 주파수 오프셋 및 위상 오프셋 보정 |
| **Differential Encoding** | 위상 모호성(phase ambiguity) 해결을 위해 절대 위상 대신 위상 변화량으로 데이터 인코딩 |
| **ISI** | Inter Symbol Interference. 이전 심볼의 파형이 다음 심볼에 간섭하는 현상 |

---
