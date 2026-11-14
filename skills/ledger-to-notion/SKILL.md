---
name: ledger-to-notion
description: Read payment screenshots, receipt photos/PDFs, KakaoTalk/SMS payment notices, or card/bank export files and add the transactions as rows to the user's Notion household-ledger (가계부) database, after showing a preview and getting approval. Also clones a ledger template when the user has none, adapts to an existing ledger, and sets up an optional daily Slack-DM reminder routine. Use when the user wants to log spending, says "put this in my 가계부", or sends Naver Pay / Kakao Pay / card / Toss payment images.
license: MIT
metadata:
  category: misc
  locale: ko-KR
---

# 가계부 → Notion

## What this skill does
결제 스크린샷·영수증·결제 알림·내역 파일에서 거래(날짜, 이름, 금액, 수입/지출)를 읽어 사용자의 Notion 가계부 DB에 행으로 추가한다. 미리보기 표를 보여주고 승인받은 거래만 넣는다. 가계부가 없으면 공개 템플릿을 복제하게 하고, 있으면 그 구조에 맞춰 넣는다. 원하면 매일 저녁 Slack DM 알림 루틴(클라우드 루틴)도 만든다.

```
[매일 저녁 Slack DM 알림] → 사용자가 결제 스크린샷/영수증을 보냄 → 읽기 → 중복 확인 → 분류 → 미리보기 → 승인 → Notion 입력
```

## When to use
- "이 영수증 가계부에 넣어줘" + 이미지/PDF
- "네이버페이 결제내역 정리해줘" (스크린샷 첨부)
- "가계부 알림 설정해줘", "매일 저녁 8시에 가계부 입력하라고 알려줘"
- 카드사·간편결제·은행 내역 파일을 Notion 가계부로 옮기고 싶을 때

## When not to use
- 가계부 앱(뱅크샐러드 등)이나 스프레드시트에 직접 입력하는 일 (이 스킬은 Notion 전용)
- 투자·세무 조언, 예산 설계
- 사용자 확인 없이 대량으로 자동 입력해야 하는 경우 (이 스킬은 항상 미리보기·승인을 거친다)

## Prerequisites
- Notion 커넥터(MCP)가 연결되어 있어야 한다. deferred 상태면 ToolSearch로 `notion-search`, `notion-fetch`, `notion-query-data-sources`, `notion-create-pages`를 먼저 불러온다.
- 알림 루틴을 쓰려면 Slack·Notion 커넥터와 `/schedule`(클라우드 루틴) 사용이 가능해야 한다.
- **개인 설정 폴더** `~/.claude/ledger-to-notion-personal/`: 사용자 개인의 Notion ID 표, 소비 습관·입력 선호, 루틴 정보를 보관한다. 이 스킬이 처음 실행될 때 만들고(스킬 폴더는 읽기 전용 캐시일 수 있다), 스킬을 공유할 때 개인 정보가 딸려 가지 않게 스킬 폴더와 분리한다.

## Workflow

### 0. 시작: 개인 설정 읽기, 가계부 찾기
1. `~/.claude/ledger-to-notion-personal/`이 있으면 `profile.md`, `notion-ids.md`, `routine.md`를 읽는다. 거기 적힌 선호가 이 문서의 기본값보다 우선한다. `notion-ids.md`에 가계부 입력 DB(data source) ID가 없으면 가계부 페이지를 `notion-fetch`해서 찾고 그 파일에 추가한다.
2. 없으면 처음 쓰는 사용자다. `notion-search`로 "가계부"를 찾아 속성에 `이름(title) / 입력 날짜(date) / 지출 금액 / 수입 금액`이 있는 DB를 후보로 잡는다.
   - **있으면** → `${CLAUDE_SKILL_DIR}/references/template-and-adapt.md`의 "기존 가계부에 맞추기"
   - **없으면** → 같은 문서의 "템플릿 복제"
3. 온보딩 결과(data source ID, 카테고리·월별 통계 URL)와 사용자가 말해 준 선호(이름 스타일, 분류 습관, 한 주문을 어떻게 나눌지)는 개인 설정 폴더에 새로 만들어 저장한다. 선호를 새로 말하면 이 스킬 파일이 아니라 개인 `profile.md`에 기록한다.

### 1. 읽기
이미지(Read), PDF, 텍스트, 엑셀/CSV에서 `날짜, 이름, 금액, 수입/지출`을 뽑는다. 아래 "읽을 때 주의"를 지킨다. 금액이 흐리거나 잘렸으면 추측하지 말고 "확인 필요"로 표시한다. 금액 오류가 가계부에서 가장 치명적이다.

### 2. 중복 확인
해당 날짜 이후의 기존 행을 조회해 같은 날짜(±1일)·같은 금액이 있는지 본다. 간편결제를 카드로 결제하면 같은 결제가 두 번 올 수 있다. 의심되면 넣지 말고 묻는다.

### 3. 분류
개인 `notion-ids.md`의 카테고리에서 고른다. 개인 `profile.md`의 습관을 우선하고, 애매하면 "확인 필요"로 둔다. 목록에 없는 카테고리를 만들지 않는다.

### 4. 미리보기와 승인
표(날짜 / 이름 / 금액 / 카테고리)로 먼저 보여주고 승인을 받는다. Notion에 들어간 행은 되돌리기 번거로워서 건너뛰지 않는다. 사용자가 "바로 넣어"라고 하면 그 세션에서만 건너뛴다.

### 5. 입력
승인된 거래만 `notion-create-pages`(parent는 data_source_id)로 넣고, 넣은 건수·합계·건너뛴 건을 짧게 보고한다. 지출 한 건의 속성:
```json
{"이름": "...", "date:입력 날짜:start": "2026-11-12", "date:입력 날짜:is_datetime": 0,
 "지출 금액": 12000,
 "분류": ["<지출 분류 URL>"], "지출 카테고리": ["<카테고리 URL>"],
 "월별 통계": ["<해당 월 URL>"]}
```
수입은 `수입 금액`, `수입 카테고리`, 수입 분류 URL을 쓴다. `분류`와 `월별 통계` relation을 빼면 월별 통계가 틀어지므로 항상 채운다. 연도가 다르면 그 연도 DB가 따로 있다(`template-and-adapt.md`의 "새해").

### 읽을 때 주의
- **포인트 사용액은 지출에서 뺀다.** 네이버페이·카카오페이는 실제 지출 = 결제정보의 머니/카드 결제액이다. 포인트 "사용"은 빼고, 포인트 "적립"은 건드리지 않는다.
- **정가와 할인가를 구분한다.** 상품 옆 취소선 금액은 정가다. 가계부에는 쿠폰·배송비 할인이 반영된 실제 결제액을 쓴다.
- **합계를 검증한다.** `상품금액 − 쿠폰할인 + 배송비 − 배송비할인 = 결제액`이 맞아야 한다. 안 맞으면 스크린샷에 가려진 품목이 있다는 뜻이니 가려진 부분을 한 장 더 보내 달라고 한다.
- 한 주문에 판매처·카테고리가 섞여 있으면 "한 행 / 카테고리별 / 판매처별" 중 어떻게 넣을지 묻는다(개인 `profile.md`에 선택이 있으면 그대로 따른다). 배송비와 할인은 해당 판매처 행에 포함해 합계를 결제액에 맞춘다.
- 날짜가 화면에 없으면 주문번호 앞자리(YYYYMMDD)에서 읽는다.
- 이름이 `...`로 잘려 있으면 **보이는 그대로** 넣는다. 안 보이는 뒷부분을 추정해서 채우지 않는다(틀린 이름이 가계부에 남는 것보다 잘린 이름이 낫다). 같은 화면이나 다른 스크린샷에 전체 이름이 실제로 보이면 그 글자만 쓰고, 잘렸다는 사실은 입력 후 알린다.

### 알림 루틴 (선택)
매일 저녁 Slack DM으로 "가계부 입력 시간이에요"를 보내는 클라우드 루틴. 만드는 법은 `${CLAUDE_SKILL_DIR}/references/routine-setup.md`. 보낼 대상·문구·시간을 사용자에게 보여주고 확인받은 뒤에 만든다(자동 발송이라 외부에 보이는 행동이다). 루틴은 알림과 요약만 하고 입력은 이 스킬이 한다.

## Done when
- 승인된 거래가 모두 Notion 가계부에 들어갔고, 건수·합계·건너뛴 건이 보고되었다.
- 새 사용자라면 개인 설정 폴더에 ID 표와 선호가 저장되었다.
- 알림을 요청받았다면 루틴을 만들고 시험 실행해 사용자가 DM 수신을 확인했다.

## Notes
- 기존 행을 수정하거나 지우지 않는다. 추가만 한다. 수정이 필요하면 사용자에게 말한다.
- 카드번호·계좌번호·승인번호, 배송지 이름·전화번호·주소는 가계부에 옮기지 않는다.
- 사용자가 준 파일 외의 곳(메일, 카톡)은 시키기 전에 읽지 않는다.
- Notion 페이지의 공유 설정을 바꾸거나 루틴을 삭제하는 일은 사용자 확인 없이 하지 않는다.
- 월 합계를 맞춰볼 자료(카드사 엑셀, 통장 내역)를 사용자가 주면 합계를 비교해 빠진 거래를 알려준다.
- 다른 구조의 기존 가계부에 맞추는 모드는 아직 한 가계부(제작자 것)에서만 검증되었다. 처음에는 항상 대응표를 사용자에게 확인받는다.
