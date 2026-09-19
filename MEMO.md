# RC Workshop for Odoo: Pet Project Checklist
Links:
- https://www.odoo.com/documentation/19.0/developer/tutorials/setup_guide.html
- https://www.odoo.com/documentation/19.0/developer/tutorials/server_framework_101.html
- https://www.odoo-community.org/

An Odoo module for an RC (cars/planes) hobby shop and workshop, with Telegram customer notifications. Items are in rough build order, so each phase leaves you with something working.

* * *

## 0\. Scope first

- [ ] Write a one-paragraph pitch, e.g. "Odoo module to manage RC models, repairs and batteries, with Telegram status updates for customers"
- [ ] Define an MVP: repair orders + Telegram notifications. Everything else is a stretch goal
- [ ] Sketch the data model on paper (models and relations)
- [ ] Write 8-10 user stories ("As a technician I can...", "As a customer I get a message when my model is ready")
- [ ] Pick a technical module name (`rc_workshop`) and the Odoo version (18 or 19)

## 1\. Environment and tooling

- [ ] docker-compose with Odoo + Postgres, custom addons folder mounted
- [ ] Developer mode on, `--dev=all` for auto-reload
- [ ] Git repo from day one, `.gitignore`, `.env` for secrets
- [ ] `pre-commit` with black / flake8 / `pylint-odoo`
- [ ] Debugger (VS Code or PyCharm attach) and a `logging` setup you actually use
- [ ] Know the update command: `-u rc_workshop` after changing Python or XML

## 2\. Module skeleton

- [ ] `__manifest__.py` (name, version, depends, data list, license, category, `application: True`)
- [ ] `__init__.py` files importing every models, wizard and controllers file
- [ ] Folders: `models/`, `views/`, `security/`, `data/`, `demo/`, `report/`, `wizard/`, `controllers/`, `tests/`, `static/description/icon.png`
- [ ] Manifest data order: security -> data -> views -> menus -> reports

## 3\. Data models (domain)

- [ ] **RC model catalog** (extend `product.template`): type (car/plane/heli/drone/boat), scale, power (electric/nitro), motor kV, battery cell count (2S/3S/4S), wingspan or wheelbase, radio protocol/frequency, skill level, brand
- [ ] **Customer's models** ("garage/hangar"): partner, product, serial number, purchase date, notes, upgrade log
- [ ] **Repair order**: customer, model, problem description, technician, priority, dates, state (`received -> diagnosing -> waiting parts -> repairing -> testing -> ready -> picked up / cancelled`)
- [ ] **Repair lines**: parts used (link to products and stock), labor hours, price
- [ ] **Battery tracking** (nice niche feature): LiPo cell count, capacity (mAh), cycle count, last health check, status
- [ ] **Compatibility**: which parts fit which models (Many2many)
- [ ] Optional: events and races (registration), demo model rental

## 4\. Business logic

- [ ] Sequence for repair numbers (`ir.sequence`, e.g. `RC/2026/0001`)
- [ ] Computed fields with correct `@api.depends` (totals, days in workshop, is-overdue)
- [ ] `@api.constrains` validations (e.g. cell count > 0, end date after start date)
- [ ] State transition methods (buttons), with `self.ensure_one()` where needed
- [ ] Wizard, e.g. "Change status and notify customer" or "Batch-assign technician"
- [ ] Cron: overdue repairs, stale "ready for pickup" reminders
- [ ] Mail templates and chatter (`mail.thread`, `mail.activity.mixin`)
- [ ] Multi-company field (`company_id`) and default values

## 5\. Customizing standard Odoo (important for interviews)

- [ ] Extend `res.partner` (Telegram chat ID, notification opt-in)
- [ ] Extend `product.template` (RC attributes above) using `_inherit`
- [ ] Extend `sale.order` or `account.move` (e.g. link an invoice to a repair, or a button "Create repair from sale")
- [ ] View inheritance with xpath, never editing core files
- [ ] At least one method override using `super()`
- [ ] Optional: hook into `stock` for reserving parts

## 6\. Views and UI

- [ ] Form with statusbar, smart buttons, and chatter
- [ ] List, search (filters and group-by), and kanban grouped by state
- [ ] Calendar or pivot/graph view for reporting (repairs per month, per technician)
- [ ] Menus and actions organized under one app icon
- [ ] Check the view syntax for your version (`list` vs `tree`, no `attrs`)

## 7\. Security

- [ ] Groups: User (technician) and Manager
- [ ] `ir.model.access.csv` entry for every new model (the classic "forgot it" error)
- [ ] Record rules (e.g. technicians see only their repairs, multi-company rule)
- [ ] Avoid unnecessary `sudo()`

## 8\. Reports

- [ ] QWeb PDF: repair receipt / work order with QR code or barcode
- [ ] Optional: battery health report

## 9\. Telegram integration

- [ ] Create the bot via BotFather; store the token in Settings (`ir.config_parameter` / `res.config.settings`), never in code or git
- [ ] A small service class wrapping the Bot API (`sendMessage`, `setWebhook`) using `requests` with timeouts
- [ ] **Linking customers**: deep link `t.me/<bot>?start=<one-time-token>` -> webhook receives `/start <token>` -> saves `chat_id` on the partner
- [ ] **Outgoing notifications**: status changes, ready for pickup, new repair created
- [ ] **Message queue model** (`telegram.message`: recipient, text, state, error, attempts) processed by cron with retries, so a Telegram outage can't break a repair save
- [ ] Handle errors: user blocked the bot (403), rate limits (429), timeouts
- [ ] **Incoming webhook**: plain HTTP controller (`csrf=False`, since Telegram doesn't send Odoo's JSON-RPC envelope)
- [ ] Verify the webhook secret token header and dedupe on `update_id`
- [ ] Bot commands: `/start`, `/status`, `/repairs`, `/help`; optionally inline buttons
- [ ] Escape text properly (HTML parse mode is safer than MarkdownV2)
- [ ] Opt-in / opt-out (`/stop`)
- [ ] Staff channel: alerts for low stock of key parts, new urgent repairs
- [ ] Dev setup: HTTPS tunnel (ngrok or cloudflared) for the webhook, since Telegram requires HTTPS
- [ ] Never log the token

## 10\. Testing

- [ ] `TransactionCase` tests for models, state flow, and constraints
- [ ] Mock outgoing HTTP (`unittest.mock.patch`) so tests never call Telegram
- [ ] `HttpCase` or a controller test for the webhook, including the invalid-secret case
- [ ] Tag tests and know the run command: `odoo-bin -i rc_workshop --test-tags /rc_workshop`
- [ ] Demo data so a fresh install looks alive

## 11\. Code quality

- [ ] Follow the OCA guidelines (naming, file layout, XML IDs)
- [ ] Wrap user-facing strings in `_()` for translation
- [ ] Docstrings on non-obvious methods
- [ ] Efficient ORM use: no `search()` inside loops, use `read_group` / batch operations

## 12\. Optional extras

- [ ] Customer portal page for repair status
- [ ] Website/eCommerce filters by RC attributes
- [ ] Small external Python script or FastAPI service using Odoo's external API (Odoo 19 added a newer JSON-2 API; check the current docs)
- [ ] A second module (`rc_telegram`) separate from `rc_workshop`. A clean split shows good architecture

## 13\. Deploy and portfolio

- [ ] README: pitch, features, screenshots or a GIF, install steps, architecture diagram, and a screenshot of a Telegram conversation
- [ ] Git history with meaningful commits and tagged releases
- [ ] Live demo (cheap VPS, or a recorded video if hosting is a hassle)
- [ ] Port to a second Odoo version, since employers often run older ones (branches named `18.0`, `19.0`)
- [ ] Optional: a small OCA pull request
- [ ] CV bullet with concrete verbs, plus a "what I'd improve next" section in the README
- [ ] Naming: repo `odoo-rc-workshop`, display name "RC Workshop", title "RC Workshop for Odoo"

* * *

## Classic pitfalls

- [ ] Every new file is added to `__init__.py` and the manifest `data` list
- [ ] Every new model has access rights
- [ ] Module updated (`-u`) after XML changes
- [ ] No network calls inside a long database transaction
- [ ] No secrets committed to git