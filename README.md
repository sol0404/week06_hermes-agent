# Week 06 — Hermes Agent 설치 및 디스코드 연동

[Hermes Agent](https://github.com/NousResearch/hermes-agent)(NousResearch)를 Windows에 설치하고 디스코드 봇으로 연결한 과정을 정리한 기록입니다.

## 환경

- Windows 11, PowerShell (WSL 없이 네이티브 설치)
- Hermes Agent v0.23.2
- 모델: `upstage/solar-mini4:free` (Nous Portal 무료 모델)
- 메신저: Discord

## 1. 설치

```powershell
iex (irm https://hermes-agent.nousresearch.com/install.ps1)
```

- `%LOCALAPPDATA%\hermes`에 설치되고, 사용자 PATH에 `hermes` 명령이 추가됩니다. 관리자 권한은 필요 없습니다.
- Python, Node.js, pnpm, ripgrep, FFmpeg, Chromium 등을 함께 받기 때문에 시간이 꽤 걸립니다(이번 설치는 20분 이상).
- 설치가 끝나면 새 터미널을 열고 `hermes doctor`로 점검합니다.
- Windows Defender가 `uv.exe`를 격리하면 오탐입니다. 관리자 PowerShell에서 아래처럼 폴더를 예외로 추가합니다.

  ```powershell
  Add-MpPreference -ExclusionPath "$env:LOCALAPPDATA\hermes\bin"
  ```

## 2. 디스코드 패키지 설치

디스코드 연동용 Python 패키지(`discord.py`)는 기본 설치에 들어 있지 않습니다.

```powershell
hermes pm install --extra discord
```

## 3. 디스코드 봇 만들기

[Discord Developer Portal](https://discord.com/developers/applications)에서 진행합니다.

1. **New Application**으로 앱을 만듭니다.
2. **Bot** 메뉴의 Privileged Gateway Intents에서 아래 두 개를 켜고 저장합니다. 꺼져 있으면 봇이 접속하지 못합니다.
   - **Server Members Intent**
   - **Message Content Intent**
3. 같은 화면에서 **Reset Token**으로 봇 토큰을 발급해 복사합니다. 토큰은 한 번만 보여 주고, 비밀번호처럼 다뤄야 합니다.
4. 디스코드 앱에서 **설정 → 고급 → 개발자 모드**를 켜고, 내 프로필의 **사용자 ID 복사**로 숫자 ID를 확인합니다. 나만 봇을 쓸 수 있게 허용 목록에 넣는 용도입니다.

## 4. Hermes 설정

```powershell
hermes setup
```

이번에 고른 항목:

| 질문 | 선택 |
|---|---|
| How would you like to set up Hermes? | 1. Quick Setup (Nous Portal) |
| Select default model | `upstage/solar-mini4:free` |
| Select terminal backend | Local (Keep current) |
| 메신저 선택 | 6. Discord만 체크 → 봇 토큰, 사용자 ID 입력 |
| Install the gateway service (자동 실행) | Yes |
| Open the UAC prompt now? | Yes (Windows 예약 작업 등록) |
| Help improve Hermes? | No thanks |

## 5. 봇을 서버에 초대

`YOUR_APP_ID`를 개발자 포털의 **애플리케이션 ID**로 바꿔서 엽니다.

```
https://discord.com/oauth2/authorize?client_id=YOUR_APP_ID&scope=bot+applications.commands&permissions=309237763136
```

서버에 초대해야 DM도 주고받을 수 있습니다.

- DM에서는 모든 메시지에 답합니다.
- 서버 채널에서는 `@봇이름`으로 불러야 답합니다.

## 자주 쓰는 명령

| 하고 싶은 일 | 명령 |
|---|---|
| 게이트웨이 상태 확인 | `hermes gateway status` |
| 게이트웨이 재시작 | `hermes gateway restart` |
| 자동 실행 해제 | `hermes gateway uninstall` |
| 모델 변경 | `hermes model` |
| 진단 | `hermes doctor` |
| 업데이트 | `hermes update` |

로그 위치: `%LOCALAPPDATA%\hermes\logs\gateway.log`
