# Getting started with ledger-to-notion

This plugin has one skill, `ledger-to-notion`. Send Claude Code a payment screenshot, a receipt photo or PDF, a KakaoTalk or SMS payment notice, or a card or bank export file, and it reads the transactions and adds them as rows to your Notion household ledger (가계부). Before adding anything it shows a preview table, and only the rows you approve go in. If you want, it also sends you a Slack DM every evening as a reminder.

The skill works in Claude Code only, and it talks to you in Korean. It knows Korean payment services such as Naver Pay, Kakao Pay and Toss.

## Let an agent set it up for you

Paste this into Claude Code:

```
Follow https://github.com/K2YOUNG/ledger-to-notion-plugin/blob/main/GUIDE.md to install and set up the ledger-to-notion plugin for me. Run the commands yourself, and stop and tell me whenever I need to do something.
```

## For AI agents helping a user

If a user gave you this guide, set the plugin up for them in this order:

1. Check that Claude Code is installed with `claude --version`.
2. Run the two commands in Step 1, then `claude plugin list` (Step 2) to confirm `ledger-to-notion` is installed.
3. Stop. Ask the user to connect Notion to Claude as described in Step 3.
4. Ask the user to quit Claude Code and start it again from the same folder with `claude -c`. That reopens this conversation with the plugin and the Notion connection loaded. You can't restart your own session.
5. After the restart, check that the Notion tools (`notion-search`) are available. If they aren't, ask the user to check the connection again.
6. Suggest a first request: send one payment screenshot or receipt with "이 영수증 가계부에 넣어줘" (put this receipt in my ledger). The skill itself finds the ledger or walks the user through copying the template.

Hand these back to the user and wait:

- connecting Notion and Slack (these need a login and a permission grant)
- the restart in step 4
- clicking "Duplicate" on the Notion template
- approving each preview table
- confirming the recipient, time and message when setting up the daily reminder

Never type a password or token for the user.

## What you need

- Claude Code, logged in with a Claude account
- A Notion account
- For the daily reminder only: a Slack account and a Claude account that can use cloud routines (`/schedule`)

## Step 1. Install the plugin

Run these two lines in a terminal:

```
claude plugin marketplace add K2YOUNG/ledger-to-notion-plugin
claude plugin install ledger-to-notion@ledger-to-notion
```

## Step 2. Check the install

```
claude plugin list
```

If `ledger-to-notion` is in the list, the install worked.

## Step 3. Connect Notion

1. Log in to claude.ai on the web and connect Notion under Connectors in the settings. When Notion asks, pick the workspace where your ledger will live and allow access.
2. Quit Claude Code and start it again.
3. In Claude Code, type `/mcp`. Notion should be listed as connected.

If you plan to use the daily reminder, connect Slack in the same place.

## Step 4. Set up the ledger (first run)

Send Claude Code one payment screenshot or receipt and ask:

```
이 영수증 가계부에 넣어줘
```

On the first run the skill looks for a ledger in your Notion.

- **If you have none**, it asks you to copy a public template. Open the link it gives you, click "Duplicate" (복제 in the Korean UI) at the top right to copy it into your workspace, then tell Claude the URL of your copy.
- **If you already have one**, it shows a table matching your ledger's properties to what the skill needs (name, date, amount, category). Check it. This mode has only been tested on the author's own ledger so far.

The Notion IDs it finds and the habits you tell it are saved in `~/.claude/ledger-to-notion-personal/`. That folder is separate from the plugin, so it never goes along when the skill is shared.

## Step 5. Add transactions

From now on, just send payment screens. The skill works in this order:

1. It reads the date, name, amount and income/expense. If an amount is blurry or cut off, it marks it "확인 필요" (needs checking) instead of guessing.
2. It looks for an existing row with the same amount on the same date (plus or minus one day). Paying through a pay app with a card can produce the same payment twice, so it asks before adding a suspected duplicate.
3. It picks a category from your ledger's categories. If it isn't sure, it marks it "확인 필요".
4. It shows a table of date, name, amount and category, and waits for your approval.
5. It adds only the approved rows and reports the count, the total and anything it skipped.

It also follows these reading rules:

- Points spent with Naver Pay or Kakao Pay are taken out of the expense.
- It uses the amount actually paid after discounts and shipping, not the struck-through list price.
- If the item total doesn't match the amount paid, an item is probably hidden on screen, so it asks for another screenshot.
- If one order mixes sellers or categories, it asks whether to add one row or split it.
- A product name cut off with `...` goes in exactly as shown.

Card numbers, account numbers, approval numbers and delivery names, phone numbers and addresses are never copied into the ledger. It only adds rows; it never edits or deletes existing ones.

## Step 6. Daily reminder (optional)

```
매일 저녁 8시에 가계부 입력하라고 알려줘
```

(Remind me every evening at 8 to fill in my ledger.) The skill first shows the routine's name, time, recipient (your own Slack) and message. Once you confirm, it creates the routine and runs it once so you can check the DM arrives. The reminder includes today's entry count and spending, and this month's spending. The routine only sends the reminder; it never writes to the ledger.

Pause or delete the routine yourself at claude.ai/code/routines.

## Example requests

| Request | What it does |
| --- | --- |
| "이 영수증 가계부에 넣어줘" + a photo | adds one receipt |
| "네이버페이 결제내역 정리해줘" + screenshots | adds several payments at once |
| "이번 달 카드 내역 엑셀이야, 가계부에 넣어줘" + a file | adds an export file |
| "이번 달 카드 합계랑 가계부 합계 비교해줘" + a card statement | finds missing transactions |
| "매일 저녁 8시에 가계부 입력하라고 알려줘" | sets up the daily reminder |

## Common problems

- **It says the Notion tools are missing**: check the connection in Step 3 and restart Claude Code.
- **It read an amount wrong**: say so at the preview step. Nothing goes in before you approve.
- **A new year started**: the ledger databases are per year. Copy the template again, or copy last year's database, and tell Claude.
- **The reminder DM doesn't arrive**: usually a Slack connection permission problem. Ask Claude to check the routine's run log.

## Update and remove

```
claude plugin marketplace update ledger-to-notion
claude plugin update ledger-to-notion@ledger-to-notion
```

```
claude plugin uninstall ledger-to-notion@ledger-to-notion
```

To remove your saved settings too, delete the `~/.claude/ledger-to-notion-personal/` folder. Delete any reminder routine separately at claude.ai/code/routines.
