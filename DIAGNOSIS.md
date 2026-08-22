# 진단 절차 — USB 열거 실패 원인 좁히기

`build.yaml` 은 **6개 펌웨어**를 한 번에 빌드합니다. 전부 구울 필요는 없고 아래 순서대로만.

## 상태 확인 방법

```bash
ls /Volumes/          # NICENANO 보이면 부트로더
```
시스템 리포트 → 하드웨어 → USB (창 닫았다 다시 열기)

| 표시 | 상태 |
|---|---|
| `Nice Keyboards` / `0x239a` | 부트로더 |
| `ZMK Project` / `0x1D50` | **앱 정상 실행** |
| 없음 | 열거 실패 |

플래싱 후 리셋 누르지 말고 USB만 재연결.

## 순서

### ① `tester_pro_micro` — 가장 중요

ZMK 공식 shield. `kscan-gpio-direct` + physical layout + matrix transform + BLE 켜짐 —
`atari_joystick` 과 구조가 **동일**하지만 100% 공식 코드입니다.

| 결과 | 결론 |
|---|---|
| 정상 | shield 구조는 문제없음 → ② |
| **실패** | shield 정의 문제가 **아님**. BLE 켜진 ZMK 앱이 이 클론 보드에서 못 뜨는 것 → ③ |

> `settings_reset` 만으로는 이 구분이 안 됩니다. `CONFIG_ZMK_BLE=n` 이라
> BLE 도 kscan 도 physical layout 도 타지 않기 때문입니다.

### ② `atari_joystick`

정상이면 → 로컬 빌드 환경 문제였음 (CI 산출물 그대로 사용).
실패면 → shield 정의 문제.

### ③ `atari_joystick-usbonly` (BLE 끔)

정상이면 → BLE 스택/설정 파티션 문제 → ④.
실패면 → BLE 무관. 보드/부트로더 문제.

### ④ `atari_joystick-nodcdchv` (고전압 DC/DC 끔)

ZMK `main` 은 nice!nano v2 에서 HV DC/DC 를 기본 활성화합니다. 알리 클론은 `DCCH`
인덕터 미실장 개체가 있어 전류 폭증·불안정 보고가 있습니다 ([zmk#2990](https://github.com/zmkfirmware/zmk/issues/2990)).

정상이면 → `config/atari_joystick.conf` 에 `CONFIG_SOC_DCDC_NRF52X_HV=n` 추가하고 마무리.

### ⑤ `atari_joystick-nosleep` (딥슬립 끔)

부팅 직후 잠들어 USB가 안 잡히는 경우 배제.

## Plan B — v0.3.0 안정판

전부 실패하면 마지막 정식 릴리스로. **두 파일을 세트로** 바꿔야 합니다.

- `config/west.yml` → `revision: v0.3.0`
- `build.yaml` → `board: nice_nano_v2`

v0.3.0(2025-08-01)은 HWMv2 마이그레이션(2025-12) **이전**이라
`nice_nano@2.0.0/nrf52840/zmk` 를 쓰면 보드를 못 찾고 빌드가 실패합니다.
문제의 HV DC/DC 기본값 변경도 v0.3.0 에는 없으므로 클론 보드엔 이쪽이 안전합니다.

## 설정 파일 검토 결과

최신 ZMK 문서 기준 대조 결과 **부팅을 막을 문법·구조 오류는 없었습니다.**

- overlay — 공식 `tester_pro_micro.overlay` 와 구조 일치
- physical layout 의 `kscan` 생략 — `chosen` 에 있으면 생략 가능(문서 명시)
- transform 6칸 / keys 6개 / input-gpios 6개 — 개수 일치
- `ZMK_KEYBOARD_NAME` 14자 — 16자 제한 이내

그래서 ①을 1순위로 둡니다.
