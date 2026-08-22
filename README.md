# 아타리 조이스틱 BLE 개조 — ZMK 설정

CX40 계열 9핀 조이스틱을 nice!nano v2(호환 클론) + ZMK 기반 무선 방향키 컨트롤러로 개조하는 설정입니다.

**현재 이 저장소의 목적은 "USB 열거 실패"의 원인을 CI 빌드로 좁히는 것입니다.
플래싱 순서와 판정표는 [`DIAGNOSIS.md`](DIAGNOSIS.md) 를 보세요.**

## 빌드

`Actions` 탭 → 워크플로 활성화 승인 → 15~25분 후 Artifacts 에서 `firmware.zip` 다운로드.

`firmware.zip` 안에 6개가 들어 있습니다.

| 파일 | 용도 |
|---|---|
| `atari_joystick-…uf2` | 실제 컨트롤러 펌웨어 |
| `tester_pro_micro-…uf2` | **핵심 대조군** — 공식 shield, 구조 동일, BLE 켜짐 |
| `settings_reset-…uf2` | 설정 초기화 / 최소 대조군 |
| `atari_joystick-usbonly-…uf2` | BLE 끔 |
| `atari_joystick-nodcdchv-…uf2` | 고전압 DC/DC 끔 (클론 보드 대응) |
| `atari_joystick-nosleep-…uf2` | 딥슬립 끔 |

## 플래싱 (macOS)

Finder 드래그는 실패하는 경우가 있으니 터미널을 쓰세요.

```bash
ls /Volumes/                              # 리셋 더블탭 후, NICENANO 확인
cp ~/Downloads/파일명.uf2 /Volumes/NICENANO/
```

`could not copy extended attributes` 경고는 정상입니다.
플래싱 후에는 리셋을 누르지 말고 USB만 다시 꽂으세요.

## 배선

원본 조이스틱은 스위치 5개가 공통 접지를 공유합니다.

| 선 색 | 기능 | 보드 실크 | pro_micro |
|---|---|---|---|
| 노랑 | 공통 (GND) | `GND` | — |
| 갈색 | 상 | `022` | D4 |
| 파랑 | 하 | `024` | D5 |
| 빨강 | 좌 | `100` | D6 |
| 하양 | 우 | `011` | D7 |
| 검정 | 발사 (SPACE) | `104` | D8 |
| — | Fn 택트 | `106` | D9 |

내부 풀업을 쓰므로 외부 저항은 불필요. Fn 스위치가 없으면 `106`은 비워둬도 됩니다.
이 클론 보드에는 물리 리셋 버튼이 없으므로 `RST`–`GND` 사이에 택트 스위치를 납땜해두면 편합니다.

## 키맵

기본: 방향키 4개 + 스페이스

| 조합 | 동작 |
|---|---|
| Fn + 상 / 하 / 좌 | 블루투스 프로필 0 / 1 / 2 |
| Fn + 우 | USB ↔ 블루투스 출력 전환 |
| Fn + 발사 | 현재 프로필 페어링 삭제 |

## 주의 — 버전 세트

`config/west.yml` 의 `revision` 과 `build.yaml` 의 `board` 는 반드시 세트입니다.

| revision | board |
|---|---|
| `main` (현재) | `nice_nano@2.0.0/nrf52840/zmk` |
| `v0.3.0` | `nice_nano_v2` |

HWMv2 마이그레이션(2025-12)이 v0.3.0(2025-08-01) 이후이기 때문입니다.
