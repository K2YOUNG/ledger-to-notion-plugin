# ledger-to-notion

A Claude Code plugin with one skill, **ledger-to-notion**. It reads payment screenshots, receipt photos and PDFs, KakaoTalk or SMS payment notices, and card or bank export files, then adds the transactions as rows to your Notion household ledger (가계부). Every batch is shown as a preview table first, and only the rows you approve are added. It can also send you a daily Slack DM reminder.

The skill is written for Korean users: it speaks Korean and knows Korean payment services (Naver Pay, Kakao Pay, Toss).

결제 스크린샷·영수증·결제 알림·카드/은행 내역을 Notion 가계부에 넣어 주는 Claude Code 플러그인입니다. 한국어 안내는 [GUIDE.ko.md](GUIDE.ko.md)에 있습니다.

New here? Read the step-by-step guide: [English](GUIDE.md) · [한국어](GUIDE.ko.md)

Or let your coding agent do it. Paste this into Claude Code:

```
Follow https://github.com/K2YOUNG/ledger-to-notion-plugin/blob/main/GUIDE.md to install and set up the ledger-to-notion plugin for me. Run the commands yourself, and stop and tell me whenever I need to do something.
```

## Install

```
claude plugin marketplace add K2YOUNG/ledger-to-notion-plugin
claude plugin install ledger-to-notion@ledger-to-notion
```

Restart Claude Code, then send a receipt with `/ledger-to-notion:ledger-to-notion` (or just "이 영수증 가계부에 넣어줘").

## Requirements

- **Claude Code** only. The skill uses Claude's Notion connector, and for the reminder, the Slack connector and cloud routines (`/schedule`).
- **Notion** connected to Claude (claude.ai settings → Connectors). Check it with `/mcp` in Claude Code.
- **Slack** and cloud routines only if you want the daily reminder.

## Your ledger

- No ledger yet? The skill points you to a public Notion template to duplicate into your own workspace. It has the ledger database plus monthly totals and income/expense categories that update by formula.
- Already have one? The skill maps your properties to what it needs and asks you to confirm the mapping first. This mode has only been tested on the author's ledger so far.

## Privacy

- Your Notion IDs, habits and reminder details are saved in `~/.claude/ledger-to-notion-personal/`, outside the plugin folder, so they never travel with the plugin.
- Card numbers, account numbers, approval numbers and delivery names, phone numbers and addresses are never copied into the ledger.
- It only adds rows. It never edits or deletes existing rows, and it doesn't read your mail or chats unless you ask.

## Layout

```
.claude-plugin/   marketplace.json, plugin.json
skills/ledger-to-notion/
  SKILL.md        workflow and reading rules (Korean)
  references/     template-and-adapt.md, routine-setup.md
```

## License

MIT, see `LICENSE`.
