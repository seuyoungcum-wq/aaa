# 지뢰찾기 (Minesweeper) — Python / tkinter

간단한 데스크톱 지뢰찾기 게임(Windows 포함 모든 플랫폼에서 동작).  
Python의 표준 GUI 라이브러리인 tkinter만 사용하므로 별도 외부 라이브러리 설치 없이 실행할 수 있습니다.

## 주요 기능
- 가변 보드 크기(행/열) 및 지뢰 개수 설정
- 첫 클릭은 항상 안전: 첫 클릭한 칸과 주변 8칸엔 지뢰가 배치되지 않음
- 좌클릭: 칸 열기 (숫자 0이면 자동으로 주변 확장)
- 우클릭: 깃발 토글(플래그)
- Chording: 열린 숫자 칸에서 주변 깃발 수와 같으면 주변 칸 자동 오픈
- 타이머 표시 및 남은 지뢰 수 표시
- 게임 승/패 처리 및 전체 오픈/플래그 상태 표시
- New Game 버튼으로 재시작 가능

## 요구사항
- Python 3.7 이상
- tkinter (대부분의 Python 배포판에 포함)
  - Windows: 기본적으로 포함
  - Linux (Debian/Ubuntu 등): `sudo apt install python3-tk` 필요할 수 있음

## 사용법
1. 저장: 제공된 소스코드를 `minesweeper.py`로 저장합니다.
2. 실행:
   - Windows (명령 프롬프트 / PowerShell):
     ```
     python minesweeper.py
     ```
   - 또는 Python이 `python3`로 호출되는 환경:
     ```
     python3 minesweeper.py
     ```
3. 실행 후 상단에서 Rows / Cols / Mines 값을 조정하고 `New Game`을 누르거나 기본값으로 시작합니다.
4. 조작:
   - 좌클릭: 칸 열기
   - 우클
