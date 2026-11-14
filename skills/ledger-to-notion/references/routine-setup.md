# 알림 루틴 (Cloud Routine + Slack DM)

이미 만든 루틴의 ID·Slack ID·환경 ID 같은 개인 정보는 `~/.claude/ledger-to-notion-personal/routine.md`에 있다(스킬 공유 대상 아님).

## 새로 만들 때

`RemoteTrigger`(deferred, ToolSearch로 불러온다)의 `create`를 쓴다. 사용자에게 먼저 보여주고 확인받는다: 이름, 시간(KST→UTC 변환 확인), 모델, 받는 곳, 문구.

1. Slack 사용자 ID: `slack_read_user_profile`(user_id 생략 시 현재 사용자)로 구한다. 또는 `slack_search_users` 결과 상단의 "current logged in user" ID.
2. 환경 ID는 `/schedule` 스킬 안내의 environment 목록에서 가져온다. 커넥터 UUID도 같은 목록에서 Notion과 Slack을 가져온다.
3. body (프롬프트의 `<...>`를 채운다):
```json
{"name": "가계부 입력 알림", "cron_expression": "0 11 * * *", "enabled": true,
 "mcp_connections": [<Notion>, <Slack>],
 "job_config": {"ccr": {"environment_id": "<env>",
   "session_context": {"model": "claude-haiku-5-5", "allowed_tools": ["Bash", "Read"]},
   "events": [{"data": {"uuid": "<새 v4 uuid 소문자>", "session_id": "", "type": "user",
     "parent_tool_use_id": null, "message": {"role": "user", "content": "<프롬프트>"}}}]}}}
```
4. 프롬프트 요지(영어로 쓰고 메시지는 한국어): ① KST 오늘 날짜·이번 달 계산 ② Notion 가계부 data source를 **읽기만** 해서 오늘 행 수·지출 합계와 이번 달 지출 합계 조회(어떤 행도 만들거나 고치거나 지우지 않음) ③ 사용자 본인 Slack ID에게만 DM 한 통, 형식:
```
🧾 가계부 입력 시간이에요
오늘 입력: N건 (지출 X원)
이번 달 지출: Y원
결제 스크린샷·영수증을 Claude에게 보내 주세요.
```
④ Notion 조회가 실패하면 숫자 없이 알림만 보내고, Slack 실패 시 오류를 최종 답에 남김. 다른 사용자·채널에는 절대 보내지 않는다.
5. 만든 뒤 `run`으로 한 번 실행하고 사용자에게 DM이 왔는지 확인받는다. 안 오면 `list_runs` → `get_run_log`로 원인을 본다(대개 커넥터 권한).

## 알아둘 것
- 항상 보낸다(오늘 입력했어도). 알림이 많으면 "오늘 입력이 있으면 건너뛰기"를 프롬프트에 추가할 수 있다.
- 루틴은 알림과 요약만 한다. 스크린샷을 받거나 가계부에 쓰지 않는다. 입력은 이 스킬이 한다.
- 루틴은 API로 삭제할 수 없다. 일시중지·삭제는 사용자가 claude.ai/code/routines에서 한다.
- 새 연도 가계부로 바뀌면 프롬프트의 data source ID를 `update`로 바꾸고 개인 `routine.md`도 고친다.
- 만든 뒤 루틴 ID와 Slack ID는 개인 `routine.md`에 기록한다.
