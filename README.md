# Vehicle Pedal CAN

센서 인터페이스 노드가 선형 가변저항과 버튼형 제동 입력을 읽어 CAN으로 STM32에 전달하고, STM32가 판단해 필요 시 RC카 측으로 제동 요청을 CAN 송신하는 실험 프로젝트입니다. 센서와 RC카는 CAN 노드가 아니므로 각각의 인터페이스 노드가 필요합니다. 두 노드의 구현/지원 여부는 확인이 필요합니다.

> **안전 범위:** 개발/벤치 검증용입니다. CAN 메시지는 그 자체로 전류를 차단하거나 기계 제동을 만들지 않습니다. 실제 구동 출력은 수신 노드, 회로, 시험 절차를 확인하고 검토한 뒤에만 연결합니다.

## 현재 목표

- 센서 노드가 입력값과 품질/시간 정보를 CAN으로 발행합니다.
- STM32가 CAN 입력을 검증하고 시간 기반 변화율과 제동 우선 여부를 판단합니다.
- STM32가 확정된 CAN 명세에 따라 제동 요청을 송신합니다.
- RC카 측 수신/구동 경로는 모의 노드부터 확인합니다.

## 현재 상태

- 구매/구성 목록: MB102 800핀, 30cm 40핀 암암/암수 듀폰 케이블, STM32F407G-DISC1, ADuM20 판매명 CAN 모듈 2개, 버튼 스위치, DM3117 10k 슬라이드 가변저항, RC카 키트 ([상세 상품명과 상태](hardware/bom.csv))
- 확인/구현 필요: 센서 입력용 MCU/CAN 컨트롤러, ADuM20 모듈의 실제 트랜시버 포함 여부, RC카 측 CAN 컨트롤러/트랜시버 내장 여부
- 미정: 보드/센서 모델, 전압/핀, CAN 속도/ID/스케일, 급가속 기준 및 오류 시 안전 정책

## 저장소 구조

```text
Vehicle_pedal_CAN/
|-- README.md
|-- firmware/                 # STM32 애플리케이션
|-- hardware/                 # BOM, 배선, 사진, 데이터시트
|-- docs/                     # 시스템/회로/소프트웨어/ICD/시험 설계
|-- data/
|   |-- raw/                  # CAN에서 관측한 원시 센서 데이터
|   `-- processed/            # 변화율과 판단 결과
`-- .gitignore                # 빌드 산출물만 제외
```

## 폴더별 기록 위치

| 폴더/파일 | 여기에 둘 것 | 기록할 내용 |
|---|---|---|
| [hardware/bom.csv](hardware/bom.csv) | 부품 목록 | 모델, 수량, 보유/확인 상태, 필요한 추가 노드 |
| [hardware/wiring/](hardware/wiring/) | 회로도와 배선표 | [netlist](hardware/wiring/netlist.csv)의 출발/도착 핀, 신호, 전압, 보호/종단 근거; [커넥터 핀아웃](hardware/wiring/connector-pinout.csv) |
| [hardware/photos/](hardware/photos/) | 실제 조립/배선 사진 | 파일별 대상, 날짜, 개정, 사진 목적을 [사진 대장](hardware/photos/photo-log.csv)에 기입 |
| [hardware/datasheets/](hardware/datasheets/) | 배포 가능한 데이터시트 | 제조사, 부품번호, 개정, URL/파일, 배포 허가를 [자료 대장](hardware/datasheets/datasheet-register.csv)에 기입 |
| [firmware/](firmware/) | STM32CubeIDE 프로젝트와 소스 | `.ioc`, `Core/`, `Drivers/` 및 보드/툴체인/CAN 설정을 [프로젝트 기록](firmware/project-record.csv)에 기입 |
| [data/](data/) | 실행별 측정 파일 | 설정과 장비는 [실행 기록 양식](data/run-record-template.csv)에 복사해 작성 |
| [data/raw/](data/raw/) | 수신한 CAN 센서 프레임 원본 | [raw 샘플 헤더](data/raw/sample-template.csv)를 사용하고 원본을 수정하지 않음 |
| [data/processed/](data/processed/) | STM32 판단 분석 결과 | [processed 샘플 헤더](data/processed/sample-template.csv)를 사용; 단위/알고리즘 버전 기록 |
| [docs/](docs/) | 설계 결정과 검증 기록 | 시스템, 흐름도, 회로 원칙, CAN ICD, SW 설계, 요구사항 및 시험 문서 |

## 실무 설계 문서

- [시스템 아키텍처](docs/system-overview.md)
- [노드 간 데이터 흐름](docs/data-flow.md)
- [회로 및 배선 설계 기록](docs/circuit-design.md)
- [하드웨어/BOM/인터페이스](docs/hardware-and-interface.md) · [BOM CSV](hardware/bom.csv)
- [배선 Netlist](hardware/wiring/netlist.csv) · [커넥터 핀아웃](hardware/wiring/connector-pinout.csv)
- [사진 대장](hardware/photos/photo-log.csv) · [데이터시트 대장](hardware/datasheets/datasheet-register.csv)
- [CAN 인터페이스 명세 (ICD)](docs/can-protocol.md)
- [STM32 소프트웨어 설계](docs/software-design.md)
- [요구사항 및 검증 추적표](docs/requirements-traceability.md)
- [검증 및 시험 계획](docs/test-plan.md)
- [실행 기록 양식](data/run-record-template.csv)
- [STM32 프로젝트 기록](firmware/project-record.csv)

## 권장 진행 순서

1. 센서 CAN 노드와 RC카 수신 노드를 식별하거나 구현합니다.
2. 모든 노드의 CAN 컨트롤러/트랜시버, 전원/레벨, 토폴로지를 회로 문서에 기록합니다.
3. 센서 상태와 제동 요청 양방향 메시지 계약을 ICD에 승인된 근거로 채웁니다.
4. 센서 노드 송신 → STM32 수신/판정 → 모의 수신 노드 제동 요청 순서로 시험합니다.
5. 실제 구동 연결은 승인된 안전 절차와 요구사항 검증 후에만 고려합니다.

측정하지 않았거나 승인되지 않은 값은 임의로 채우지 말고 `미정`으로 남깁니다.
