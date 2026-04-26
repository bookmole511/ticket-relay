# TicketReceiverAPI

TicketReceiverAPI는 Relay Processor로부터 전달받은 JSON 티켓을 수신하여, `type` 값에 따라 12개의 전용 핸들러 중 하나로 라우팅하여 처리한 후 성공 응답을 반환하는 경량 FastAPI 서비스입니다. Health Check 엔드포인트도 제공하여 업스트림 시스템에서 상태를 확인할 수 있습니다.

## 프로젝트 구조

```text
ticket_receiver_api/
├── main.py
├── config.py
├── models.py
├── handlers.py
├── dispatcher.py
├── utils.py
├── requirements.txt
├── README.md
└── .env.example
```
## 설치 방법

```bash
cd ticket_receiver_api
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
```

## 설정 (Configuration)
환경 변수로 설정을 관리하며, pydantic-settings를 사용합니다. 
모든 변수는 TICKET_RECEIVER_ 접두사를 가집니다.

.env.example 파일을 복사하여 사용하세요:

```bash
copy .env.example .env
```

## 주요 설정 항목

- `TICKET_RECEIVER_HOST`: 서버 바인딩 호스트 (기본값: `0.0.0.0`)
- `TICKET_RECEIVER_PORT`: 서버 포트 (기본값: `8000`)
- `TICKET_RECEIVER_LOG_LEVEL`: 로그 레벨 (기본값: `INFO`)
- `TICKET_RECEIVER_LOG_FILE`: 로그 파일 경로 (기본값: `ticket_receiver_api.log`)
- `TICKET_RECEIVER_CORS_ORIGINS`: CORS 허용 Origin (기본값: `*`)
- `TICKET_RECEIVER_RELOAD`: 개발용 reload 활성화 (기본값: `false`)

## 실행 방법
```bash
python main.py
```

또는 uvicorn 직접 실행:

```bash
uvicorn main:app --host 0.0.0.0 --port 8000
```
API 문서는 다음 주소에서 확인할 수 있습니다:

- `http://localhost:8000/docs`
- `http://localhost:8000/redoc`

## 주요 엔드포인트
### GET /health

서버 상태와 UTC 타임스탬프 반환합니다.

```bash
curl http://localhost:8000/health
```

반환 예시:

```json
{
  "status": "healthy",
  "timestamp": "2026-04-25T14:00:00.000000Z"
}
```

### POST /tickets
JSON 티켓을 받습니다. 필수 필드는 `type` 하나뿐이며, 값은 `1`부터 `12`까지의 정수 또는 정수 형태의 문자열일 수 있습니다.

```bash
curl -X POST http://localhost:8000/tickets ^
  -H "Content-Type: application/json" ^
  -d "{\"type\": 3, \"title\": \"Sample Ticket\", \"data\": {\"source\": \"relay\"}}"
```

반환 예시:

```json
{
  "status": "success",
  "message": "Ticket processed successfully",
  "type": 3,
  "ticket_id": "5ec78f63-d775-4d50-ad32-af44f84b40fa"
}
```

## 티켓 Type 속성 데이터별 반환 예시

Type `1`:

```bash
curl -X POST http://localhost:8000/tickets -H "Content-Type: application/json" -d "{\"type\": 1, \"title\": \"Login issue\"}"
```

Type `6`:

```bash
curl -X POST http://localhost:8000/tickets -H "Content-Type: application/json" -d "{\"type\": 6, \"title\": \"Billing update\", \"data\": {\"account_id\": \"A-100\"}}"
```

Type 문자열 `12`:

```bash
curl -X POST http://localhost:8000/tickets -H "Content-Type: application/json" -d "{\"type\": \"12\", \"title\": \"Escalation\"}"
```

Unknown type:

```bash
curl -X POST http://localhost:8000/tickets -H "Content-Type: application/json" -d "{\"type\": 99, \"title\": \"Unknown\"}"
```

HTTP `400 Bad Request`을 반환합니다.

## Logging

로그는 stdout과 설정된 로그 파일에 모두 기록됩니다.
기본 형식:

```text
2026-04-25 14:00:00 | INFO     | main | TicketReceiverAPI starting
```

운영 상황에서는 로그파일에 대한 별도 관리가 필요합니다.