개요

GameCatcher Pro는 Steam 게임의 패치 노트/공지와 세일(가격 인하) 를 감지해 디스코드 채널로 보내는 봇입니다.

Pro 전용: 길드(서버)별 상세 설정, 최소 할인율, 키워드 필터, 알림 채널/역할, 쿨다운, Premium Apps(앱 구독) Entitlement 체크.

백필 방지: NO_BACKFILL이 켜져 있으면 신규 watch 추가/부팅 시 과거 글을 “본 것으로 시딩”해, 과거 패치가 한꺼번에 올라오지 않음.

주요 기능

패치/공지 임베드 + 버튼: 스토어/공지 링크, 썸네일 자동 첨부

세일 감지: 할인율/가격 변동 시 임베드 공지

Premium Apps Entitlement: Discord 구독/comp(무상 제공) 보유 길드만 Pro 기능 허용

Pro 슬래시 커맨드

/discount_set — 최소 할인율(%)

/keywords_set — 패치 제목 키워드 필터

/role_set — 멘션 역할(sale/patch)

/channel_set — 알림 채널(sale/patch)

/cooldown_set — 동일 앱 알림 최소 간격(분)

공통: /watch_add|remove|list, /license_status|license_refresh, /diag

요구 사항

Python 3.8+ (venv 권장), discord.py, aiohttp, requests

디스코드 봇 토큰, (Pro) Application ID

(권장) Ubuntu + systemd 서비스

빠른 설치 (Ubuntu, Pro)
# 0) 폴더/가상환경
sudo install -d -o $USER -g $USER -m 0755 /opt/game-notify
python3 -m venv /opt/game-notify/venv
/opt/game-notify/venv/bin/pip install -q --disable-pip-version-check aiohttp requests discord.py

# 1) 앱 코드 배치 (이미 깔려있다면 생략)
#   👉 최신 app.py는 레포지토리의 /opt/game-notify/app.py 로 배치하세요.

# 2) 상태 디렉토리
sudo install -d -o gamebot -g gamebot -m 0755 /var/lib/game-notify || sudo useradd -r -s /usr/sbin/nologin gamebot && \
sudo install -d -o gamebot -g gamebot -m 0755 /var/lib/game-notify

# 3) systemd 유닛
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

# 4) 환경파일(Pro)
sudo install -d -m 0755 /etc/game-notify
sudo tee /etc/game-notify/pro.env >/dev/null <<'ENV'
# 필수
DISCORD_TOKEN=여기에_프로_봇_토큰
DISCORD_APP_ID=여기에_프로_앱_Application_ID
TIER=pro

# 권장
STRICT_SPLIT=true
STEAM_CC=kr
POLL_SEC=180

# 옵션(초기 기본값)
STEAM_DISCOUNT_PCT=20
NEWS_FETCH_COUNT=3
NEWS_LIMIT_PER_RUN=1
POST_COOLDOWN_SEC=1.0
NAME_TTL_DAYS=7
COOLDOWN_MIN=5
NO_BACKFILL=true
# DEV_PRO_BYPASS=0  # 개발 테스트 외엔 끄세요
ENV

# 5) 기동
sudo systemctl daemon-reload
sudo systemctl enable --now game-notify@pro
journalctl -u game-notify@pro -f -o cat

Discord 설정 (Pro, Premium Apps)

Developer Portal > Applications > (Pro 앱)

Monetization > Premium Apps 활성화

Entitlements(구독/comp):

실제 과금 없이 자신의 길드에서 테스트하려면 comp(무상 제공) Entitlement를 해당 길드에 부여

그러면 /license_status에서 Entitlement: OK 로 표시되고 Pro 기능이 동작

필요한 권한/Intents 설정 (기본 텍스트 채널 전송이면 기본으로 충분)

환경 변수
변수	설명	기본
DISCORD_TOKEN	디스코드 봇 토큰	(필수)
DISCORD_APP_ID	Pro 앱 Application ID	(필수, Pro)
TIER	pro 고정	pro
STRICT_SPLIT	채널 분리 강제	true
CHANNEL_ID, CHANNEL_ID_PATCH, CHANNEL_ID_SALE	기본/패치/세일 채널 ID(숫자)	0
STEAM_CC	상점 국가코드	kr
STEAM_DISCOUNT_PCT	초기 최소 할인율	20
NEWS_KEYWORDS	패치 제목 키워드(콤마 구분)	빈값
STEAM_APPIDS	초기 watch appid(공백 구분)	빈값
POLL_SEC	폴링 간격(초)	180
ROLE_ID_SALE, ROLE_ID_PATCH	기본 멘션 역할 ID	0
NO_BACKFILL	과거글 백필 방지	true
NEWS_FETCH_COUNT	앱당 뉴스 조회 개수	3
NEWS_LIMIT_PER_RUN	루프당 앱당 포스트 한도	1
POST_COOLDOWN_SEC	메시지 간 지연(초)	1.0
NAME_TTL_DAYS	앱 이름 캐시(일)	7
COOLDOWN_MIN	동일 앱 알림 최소 간격(분)	5
ENTITLE_TTL_SEC	Entitlement 캐시(초)	600
DEV_PRO_BYPASS	개발용 Pro 강제 허용	false

길드별 설정은 /var/lib/game-notify/config.json에 저장되며 슬래시 커맨드로 갱신됩니다.

슬래시 커맨드

공통:

/watch_add appid:12345, /watch_remove appid:12345, /watch_list

/license_status, /license_refresh, /diag

Pro 전용:

/discount_set 10

/keywords_set hotfix,패치,update

/role_set kind:(sale|patch) role:@역할

/channel_set kind:(sale|patch) channel:#채널

/cooldown_set minutes:5

동작 규칙

NO_BACKFILL 시딩: watch 추가/부팅 직후 해당 앱의 최신 n개(기본 3개)를 이미 본 것으로 기록 → 과거 패치가 한꺼번에 올라오는 문제 방지

Pro 길드 판단: TIER=pro + 해당 길드가 Premium Apps Entitlement(구독/comp) 를 보유해야 Pro 기능 허용

Free 대비 차이: Free에서는 최소 할인율이 강제 30% 이상으로 고정되지만, Pro는 /discount_set으로 완화 가능

트러블슈팅

Entitlement: NO

comp를 부여했는지 확인 → /license_refresh 이후 /license_status 재확인

DISCORD_APP_ID 오입력/누락 확인

과거 패치가 우르르 올라옴

NO_BACKFILL=true 확인, 이미 올라온 경우 /watch_remove 후 /watch_add 로 재등록

권한 문제로 재시작/파일 배치 실패

sudo 사용, gamebot 사용자/그룹 존재 확인

라이선스

저장소의 LICENSE를 따릅니다.
