개요

GameCatcher Free는 Steam 패치/세일 알리미의 무료 에디션입니다.

단일 기본 채널 사용(길드별 커스텀 무시)

최소 할인율 30% 강제(낮게 설정해도 30% 미만은 무시)

키워드/채널/역할/쿨다운 등 고급 설정은 비활성화

NO_BACKFILL 시딩으로 과거글 백필 방지 지원

요구 사항

Python 3.8+ (venv 권장), discord.py, aiohttp, requests

디스코드 봇 토큰

빠른 설치 (Ubuntu, Free)
# 0) 폴더/가상환경
sudo install -d -o $USER -g $USER -m 0755 /opt/game-notify
python3 -m venv /opt/game-notify/venv
/opt/game-notify/venv/bin/pip install -q --disable-pip-version-check aiohttp requests discord.py

# 1) 앱 코드 배치 (Pro/Free 동일 app.py 사용, TIER로 동작 구분)
#   👉 최신 app.py는 레포지토리의 /opt/game-notify/app.py 로 배치하세요.

# 2) 상태 디렉토리
sudo useradd -r -s /usr/sbin/nologin gamebot 2>/dev/null || true
sudo install -d -o gamebot -g gamebot -m 0755 /var/lib/game-notify

# 3) systemd 유닛(공용)
sudo tee /etc/systemd/system/game-notify@.service >/dev/null <<'UNIT'
[Unit]
Description=GameCatcher (%i)
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=gamebot
Group=gamebot
EnvironmentFile=/etc/game-notify/%i.env
ExecStart=/opt/game-notify/venv/bin/python /opt/game-notify/app.py
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
UNIT

# 4) 환경파일(Free)
sudo install -d -m 0755 /etc/game-notify
sudo tee /etc/game-notify/free.env >/dev/null <<'ENV'
# 필수
DISCORD_TOKEN=여기에_프리_봇_토큰
TIER=free

# 안내: Free는 길드 커스텀을 무시하고 단일 채널만 사용합니다.
# 하나의 채널을 지정하려면 아래 CHANNEL_ID를 채워주세요.
CHANNEL_ID=0  # 예: 123456789012345678

# 권장
STEAM_CC=kr
POLL_SEC=180

# 옵션(초기 기본값)
STEAM_DISCOUNT_PCT=20          # Free라도 실제 동작은 최소 30%로 강제
NEWS_FETCH_COUNT=3
NEWS_LIMIT_PER_RUN=1
POST_COOLDOWN_SEC=1.0
NAME_TTL_DAYS=7
COOLDOWN_MIN=5
NO_BACKFILL=true
ENV

# 5) 기동
sudo systemctl daemon-reload
sudo systemctl enable --now game-notify@free
journalctl -u game-notify@free -f -o cat

환경 변수(Free에서 의미 있는 것)
변수	설명	기본
DISCORD_TOKEN	디스코드 봇 토큰	(필수)
TIER	free 고정	free
CHANNEL_ID	단일 채널 ID	0
STEAM_CC, POLL_SEC	지역/폴링 간격	kr, 180
STEAM_APPIDS	초기 watch appid(공백 구분)	빈값
NEWS_*, POST_COOLDOWN_SEC 등	동작 튜닝	위 예시 참고

Free에서는 길드 커스텀 설정(채널/역할/키워드/쿨다운 등)이 무시됩니다.

슬래시 커맨드(Free에서 허용)

/watch_add appid:12345, /watch_remove appid:12345, /watch_list

/license_status (항상 Free-Only로 표기), /license_refresh

/diag (현재 환경 요약 표시)

동작 규칙

최소 할인율 30% 강제: STEAM_DISCOUNT_PCT를 낮춰도 Free에서는 30% 미만 세일 공지는 나오지 않음

NO_BACKFILL 시딩: 과거 패치/공지의 대량 업로드 방지

단일 채널 사용: CHANNEL_ID만 사용, 기타 채널/역할 설정 무시

트러블슈팅

메시지가 안 옴: CHANNEL_ID가 올바른지, 봇 권한(메시지 보내기/임베드)이 있는지 확인

과거 패치가 몰려옴: NO_BACKFILL=true 확인. 이미 올라왔다면 /watch_remove 후 /watch_add 다시 등록
