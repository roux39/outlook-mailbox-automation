[한국어](rule-design.md) | **English**

# Rule Design Document

[← Back to README](../README.en.md)

## 1. Requirements

| No. | Requirement | Decision |
|---|---|---|
| R1 | See similar requests in one place | Use **categories (tags)** instead of moving mail. Emails stay in their folders and are grouped by category |
| R2 | New emails must be categorized automatically | Incoming-mail rules check subject keywords and apply categories |
| R3 | Categorize both domestic supplier and internal emails | Domestic supplier folders + inbox (internal mail) are in scope |
| R4 | Mark overseas mail as read, except mail addressed to me | Use the condition "I'm not on the To line" (CC-only mail is marked read) |
| R5 | Mark mail older than two months as read | Bulk mark-as-read for unread mail before the cutoff date |
| R6 | Delete nothing | No delete or move actions. Existing categories kept as they were |

## 2. Category design

The design started with 6 categories and was refined to 8 after reviewing real mail.

| Change | Reason |
|---|---|
| Removed "Logistics & customs" | Shipment and customs are overseas work, not needed for domestic purchasing |
| Added "Demand / Forecast" | Demand and forecast emails from sales were mixed in with ETA questions |
| Merged "Goods receipt & delivery notes" | Delivery notes follow the same flow as goods receipt processing |
| Dropped keywords for receipt confirmations, shipping requests and DC arrivals | They mostly matched notification emails that are never looked up later |
| Kept tax invoices as a separate "Settlement" category | Delivery notes (receipt check) and tax invoices (payment) are checked at different times |

## 3. Keyword principles

- **Avoid short English words that cause false matches.**
  - `PO` also matches inside Report, Import, Position → replaced with `PO No`, `PO#`, `Purchase Order`
  - `ETA` also matches inside Details, Metal → replaced with Korean phrases (`입고 일정`, `납기`)
- **Check the subject only, not the body.** Bodies mix several topics as replies accumulate.
- **Allow several categories on one email.** Forcing a single category would cause missed requests.
- Known limitation: `COO` may also match words like "Cooperation", so it's monitored in use.

## 4. Rule order and behavior

Outlook runs rules top to bottom and stops at any rule with "Stop processing more rules" turned on.

| Order | Rule | Stop processing |
|---|---|---|
| 1 | Overseas import/customs mail → mark read | Off |
| 2–9 | 8 category rules | Off |
| 10 | Overseas senders → mark read + clear categories | Off |
| 11+ | Existing per-supplier folder rules | Overseas: on / Domestic: off |

- New rules are added at the top of the list, so the category rules run before the folder moves; emails are moved to their usual folder with the category already applied.
- Overseas emails also pass through the category rules first, so rule 10 clears their categories to keep them out of the category views.

## 5. Identifying overseas senders

The first idea was to identify overseas mail by email domain, but overseas affiliate staff use the **same company domain**, so domain alone wasn't enough.

- Senders were collected from the existing overseas folder rules (about 240 rules).
- External suppliers and forwarders were added by **domain**; overseas affiliate staff by **individual address**.
- A few senders with no recent mail couldn't be matched to an address and were left out. Their emails still go to the right folder but stay unread; they're added to the list when found.

## 6. Problems and fixes

| Problem | Cause | Fix |
|---|---|---|
| CC emails stayed unread when only "sent to me" should | Outlook's "To me" filter includes both To and CC | Switched to a `to:` search that checks the To line only |
| Some emails weren't restored to unread after a bulk mark-as-read | The restore ran before the folder-wide mark-as-read finished on the server | Waited for completion, then restored and re-queried to verify |
| Only 75 emails can be selected at once | Outlook on the web's selection limit | Repeated search → select → process |
| Couldn't process only old emails in conversation view | Selection works per conversation, catching recent replies too | Switched to "Don't group messages" during the work, then restored the original view |
| All existing rules had "Stop processing more rules" on | All 315 folder rules were set to stop further rules | Changed the setting on 51 domestic supplier rules + 5 tax-invoice rules |

## 7. Day-to-day operation

- New emails are categorized automatically. Only emails whose subject has no keyword need a manual tag.
- If emails of one type keep being missed, add the word from their subject to that rule.
- If emails get the wrong category, replace the triggering keyword with a more specific phrase.
