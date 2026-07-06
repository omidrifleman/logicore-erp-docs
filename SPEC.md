# SPEC.md — FreightCore Master Build Prompt v2 (تکمیل‌شده با Gap Analysis سبا سیستم)

```markdown
# FreightCore — Master Build Prompt (SPEC.md)
## نسخه ۲.۰ — یکپارچه‌شده با Gap Analysis کاتالوگ سبا سیستم
```

<aside>
📋

این سند یک «پرامپت فنی نهایی» است. کل متن زیر را مستقیماً به AI Coding Agent (Cursor / Claude Code / Windsurf) بدهید. Agent موظف است بدون پرسیدن سوال اضافی، برنامه را از صفر تا صد طبق همین سند بسازد. این پلتفرم کاملاً مبتنی بر وب است — هیچ اپلیکیشن نیتیو Android/iOS وجود ندارد. تمام پنل‌ها PWA هستند.

</aside>

---

# بخش ۱: معرفی و اهداف پروژه

## شرح محصول

یک پلتفرم SaaS چندمستاجری (Multi-tenant) به نام **FreightCore** بساز: سامانه جامع مدیریت حمل‌ونقل بین‌المللی و فورواردری که کل چرخه حیات یک «پرونده حمل» (Shipment File) را از استعلام نرخ و ثبت سفارش تا عملیات چندوجهی (دریایی/هوایی/زمینی/ریلی/ترکیبی)، صدور اسناد (HBL/MBL، HAWB/MAWB، Manifest، Shipping Order)، مالی و حسابداری عملیاتی (AR/AP)، رهگیری زنده، کوریری Last-mile، پرتال نمایندگان و پرتال مشتری، نمایندگی کشتیرانی (Agency Line)، عملیات فیدری، مدیریت NVOCC و کانتینر، درگاه پرداخت چندبانکی، فروشگاه آنلاین و پرتال توزیع‌کنندگان در یک بستر یکپارچه **مبتنی بر وب** مدیریت می‌کند.

**مسئله‌ای که حل می‌کند:** پراکندگی اطلاعات پرونده‌ها بین اکسل/پیام‌رسان/سیستم‌های جزیره‌ای، قطع ارتباط عملیات و مالی، خطا و تاخیر در اسناد، نبود گزارش مدیریتی قابل اتکا، و نبود یکپارچگی با سامانه‌های بیرونی (گمرک، راهداری، بیمه، پست).

## مخاطب هدف

- شرکت‌های فورواردری و کریری بین‌المللی (۵ تا ۵۰۰ کاربر داخلی)
- نمایندگی‌های کشتیرانی، اپراتورهای NVOCC و مدیریت کانتینر
- شرکت‌های کوریری و پست سریع با تحویل درون‌شهری
- شرکت‌های فیدری و عملیات خطوط فرعی کشتیرانی
- توزیع‌کنندگان و نمایندگان فروش چندسطحی
- بازار جغرافیایی اصلی: ایران و منطقه خاورمیانه (فارسی/RTL اجباری)، با پشتیبانی کامل انگلیسی برای اسناد و طرف‌های خارجی

## معیارهای موفقیت قابل‌سنجش

1. ثبت یک پرونده حمل کامل (سفارش + قیمت + سند + هزینه) در **کمتر از ۴ دقیقه** توسط اپراتور آموزش‌دیده
2. صدور HBL از روی داده پرونده بدون ورود مجدد داده: **۰ فیلد تکراری تایپ‌شده**
3. مغایرت مالی پرونده (اختلاف سود محاسبه‌شده و واقعی): **کمتر از ۰.۵٪**
4. زمان پاسخ P95 برای APIهای فهرست/جستجو: **زیر ۳۰۰ms** و برای گزارش‌های داشبورد: **زیر ۲ ثانیه**
5. آپتایم ماهانه: **۹۹.۹٪**؛ RPO ≤ ۱۵ دقیقه، RTO ≤ ۱ ساعت
6. پوشش تست backend ≥ **۸۰٪** خطوط و ۱۰۰٪ برای منطق مالی
7. PWA: نمره Lighthouse PWA ≥ ۹۰؛ First Contentful Paint < ۱.۵s روی شبکه 4G
8. فرآیند Onboarding مشتری جدید از ثبت‌نام تا اولین پرونده: **کمتر از ۳۰ دقیقه** بدون نیاز به پشتیبانی انسانی

---

# بخش ۲: پشته فناوری (Tech Stack — نسخه‌های ۲۰۲۶)

| لایه | انتخاب | نسخه | دلیل |
| --- | --- | --- | --- |
| Frontend | Next.js (App Router) + React | 16.2.x / React 19.2 | SSR/ISR برای پرتال‌ها، Turbopack پایدار، View Transitions، اکوسیستم RTL |
| UI Kit | Tailwind CSS + shadcn/ui + Radix | Tailwind 4.x | توکن‌محور، پشتیبانی کامل RTL با logical properties |
| PWA | Workbox 7 + next-pwa | آخرین پایدار | Service Worker، Offline-first، Web Push API، Add to Home Screen |
| State/Data | TanStack Query 5 + Zustand 5 | آخرین پایدار | جداسازی server-state و client-state |
| Backend | NestJS | 11.1.x (Node.js 22 LTS) | معماری ماژولار منطبق بر ماژول‌های دامنه، DI، Express v5 |
| ORM | Prisma | 6.x | Migration ایمن، typed client، پشتیبانی RLS |
| دیتابیس | PostgreSQL | 18.x | uuidv7() بومی، Async I/O، temporal constraints؛ Row-Level Security |
| Cache/Queue | Redis 7.4 + BullMQ 5 | آخرین پایدار | صف اعلان‌ها، PDF، import/export، وبهوک‌ها |
| Realtime | WebSocket ([Socket.IO](http://Socket.IO) 4) | — | رهگیری زنده و اعلان درون‌برنامه‌ای |
| Object Storage | S3-compatible (MinIO در dev) | — | فایل اسناد، PDFها |
| PDF/اسناد | Playwright HTML→PDF + Handlebars | — | اسناد دوزبانه فارسی/انگلیسی با فونت embed شده |
| Geolocation (راننده) | Browser Geolocation API (وب) | — | driver tracking از طریق مرورگر موبایل — بدون اپ نیتیو |
| امضای دیجیتال | Canvas Signature Pad (وب) | — | در مرورگر موبایل؛ فاز ۳: PKI واقعی |
| Observability | OpenTelemetry + Grafana/Loki/Tempo + Sentry | — | ردیابی توزیع‌شده و خطاها |
| CI/CD | GitHub Actions + Docker Compose (dev) + Kubernetes-ready Dockerfile | — | — |

## سرویس‌های شخص‌ثالث (همه پشت interface انتزاعی + آداپتور Mock در dev)

- **نقشه/ژئوکدینگ:** Neshan یا [Map.ir](http://Map.ir) (ایران) + OpenStreetMap/Mapbox GL JS؛ interface واحد `MapProvider`
- **رهگیری کانتینر/پرواز:** interface `CarrierTrackingProvider` با آداپتورهای stub (MVP: ثبت دستی milestone + وبهوک)
- **پیامک/اعلان:** Kavenegar یا [SMS.ir](http://SMS.ir) (interface `SmsProvider`)، ایمیل SMTP (Nodemailer)، Push از طریق **Web Push API** (VAPID)
- **درگاه پرداخت:** interface عمومی `PaymentGateway` با آداپتورهای چندگانه: Zarinpal، Mellat، Parsian، Saderat، Saman؛ **PCI DSS compliant**
- **امضای دیجیتال اسناد:** interface `ESignProvider` — MVP: Canvas signature pad وب + hash SHA-256؛ فاز ۳: PKI معتبر
- **نرخ ارز:** interface `FxRateProvider` با ثبت دستی + API (نرخ روزانه در `fx_rates`)
- **IoT/سنسور:** interface `IotSensorProvider` پروتکل MQTT 5.0 برای سنسورهای دما/رطوبت کانتینر reefer
- **سامانه گمرک:** interface `CustomsProvider` — اتصال به GMPCS/EPL ایران (REST/SOAP)؛ MVP: mock + دستی
- **سامانه راهداری:** interface `RoadTransportProvider` — API راهداری + ناجا برای پروانه حمل
- **بیمه باربری:** interface `CargoInsuranceProvider` — آداپتور شرکت‌های بیمه
- **پست ایران:** interface `PostalProvider` — API ردیابی و ارسال پست ایران

---

# بخش ۳: معماری سیستم

## ۳.۱ Multi-tenancy

- مدل: **دیتابیس مشترک + ستون `tenant_id` روی تمام جداول دامنه + PostgreSQL Row-Level Security (RLS)**
- هر request پس از احراز هویت، `tenant_id` را در `SET LOCAL app.tenant_id` قرار می‌دهد
- هیچ query دامنه‌ای بدون tenant context اجرا نمی‌شود (interceptor سراسری NestJS)
- Super-admin در schema جدا (`platform`) کار می‌کند
- فایل‌ها در bucket با prefix `tenant/{tenant_id}/...`
- **White-label:** هر tenant می‌تواند لوگو، رنگ‌های برند، دامنه اختصاصی (custom domain) و نام نمایشی پلتفرم را در تنظیمات سازمان تعریف کند؛ PWA با ServiceWorker دامنه اختصاصی را پشتیبانی می‌کند

## ۳.۲ نمودار توصیفی ارتباط ماژول‌ها

```
[CRM/فروش] → استعلام نرخ → [Pricing/Quotation] → تبدیل به سفارش →
[Shipment Core: پرونده حمل] ←→ [Documents] (صدور اسناد)
        │                    ←→ [Finance AR/AP] (هزینه/درآمد)
        │                    ←→ [Tracking] (milestone + GPS وب)
        │                    ←→ [Courier/Last-mile] (پرونده‌های Parcel)
        │                    ←→ [Agency Line] (نمایندگی کشتیرانی)
        │                    ←→ [Feeder] (عملیات فیدری)
        │                    ←→ [NVOCC] (مدیریت کانتینر)
        ├──→ [Agent Portal PWA] (تخصیص پرونده + تسویه)
        ├──→ [Customer Portal PWA] (وضعیت/اسناد/صورتحساب)
        ├──→ [Distributor Portal PWA] (نمایندگان فروش/اقساط)
        ├──→ [Driver PWA] (ماموریت راننده با GPS مرورگر)
        └──→ [BI/Reports] (از read replica)
[IPG Multi-Bank] ←→ [Finance] ←→ [E-Commerce]
[Customs/EPL] ←→ [Shipment Core] ←→ [Documents]
[Insurance] ←→ [Shipment Core]
[Road Transport] ←→ [Carrier/Courier]
[Org Settings/Plans] بر همه ماژول‌ها حاکم است
[Notification Service] از همه ماژول‌ها event می‌گیرد
```

## ۳.۳ مدل دیتابیس

```sql
-- === PLATFORM ===
platform.tenants(id, name, slug, plan_id FK, status, trial_ends_at, locale_default, timezone,
  brand_logo_file_id, brand_primary_color, brand_secondary_color, custom_domain, pwa_name, pwa_short_name)
platform.plans(id, name, code, price_monthly, features JSONB, limits JSONB)
platform.tenant_modules(id, tenant_id FK, module_code, enabled BOOL, enabled_until)

-- === USERS & AUTH ===
users(id, tenant_id, email, phone, password_hash, full_name, avatar_url, status, last_login_at,
  mfa_secret, locale, role_type[internal|agent|customer|distributor|contractor|shipping_line_agent])
roles(id, tenant_id NULL, code, name)
user_roles(user_id FK, role_id FK, branch_id FK NULL)
permissions(id, code)
role_permissions(role_id, permission_id, actions BIT)
branches(id, tenant_id, name, code, address, phone, is_hq BOOL)
audit_logs(id, tenant_id, user_id, action, entity_type, entity_id, before JSONB, after JSONB, ip, ua, created_at)

-- === PARTIES ===
parties(id, tenant_id, type[customer|carrier|agent|supplier|consignee|shipper|customs_broker|
  shipping_line|distributor|contractor|insurance_co|post_provider], legal_name, trade_name,
  tax_id, country, city, address, postal_code, phones JSONB, emails JSONB, credit_limit,
  payment_terms_days, status)
contacts(id, party_id FK, name, position, phone, email)

-- === QUOTATION & PRICING ===
quotations(id, tenant_id, quote_no, party_id FK, mode, incoterm, origin_location_id,
  dest_location_id, cargo JSONB, valid_until DATE, status, total_amount, currency, assigned_to FK)
quotation_lines(id, quotation_id FK, charge_code FK, description, qty, unit, unit_price,
  currency, cost_price, margin_pct)
rate_cards(id, tenant_id, carrier_party_id FK, mode, origin, destination, container_type,
  valid_from, valid_to, buy_rate, sell_rate, currency, transit_days, notes)
charges(id, tenant_id, code, name_fa, name_en, category, default_currency, taxable BOOL)

-- === SHIPMENTS ===
shipments(id, tenant_id, file_no, quotation_id FK NULL, direction, mode, service_type,
  incoterm, shipper_party_id, consignee_party_id, customer_party_id, agent_party_id,
  carrier_party_id, shipping_line_party_id NULL, origin_location_id, dest_location_id,
  etd, eta, atd, ata, status, branch_id, operator_user_id, salesperson_user_id,
  customs_declaration_no NULL, insurance_policy_no NULL, road_permit_no NULL)
shipment_legs(id, shipment_id FK, seq INT, mode, from_location_id, to_location_id,
  carrier_party_id, vehicle_ref, etd, eta, atd, ata, status)
shipment_containers(id, shipment_id FK, container_no, type, seal_no, tare_kg, gross_kg,
  packages INT, volume_cbm, nvocc_managed BOOL, rental_cost NUMERIC, rental_currency)
shipment_cargo(id, shipment_id FK, description, hs_code, packages INT, package_type,
  gross_weight_kg, chargeable_weight_kg, volume_cbm, is_dg BOOL, dg_class, un_no, temperature_c)
milestones(id, shipment_id FK, leg_id NULL, code, planned_at, actual_at, location_id, note, source)
locations(id, code, name_fa, name_en, type, country, lat, lng)

-- === DOCUMENTS ===
documents(id, tenant_id, shipment_id FK, doc_type, doc_no, direction, language, status,
  data JSONB, file_id FK, version INT, signed_hash, issued_at, issued_by, qr_code_data NULL)
files(id, tenant_id, bucket_key, filename, mime, size_bytes, sha256, uploaded_by)

-- === FINANCE ===
invoices(id, tenant_id, invoice_no, type, party_id FK, shipment_id FK NULL, branch_id,
  currency, fx_rate, subtotal, tax_amount, total, status, due_date, tax_ref_no NULL)
invoice_lines(id, invoice_id FK, shipment_id NULL, charge_id FK, description, qty, unit_price,
  tax_pct, total)
payments(id, tenant_id, party_id, direction, method, amount, currency, fx_rate, paid_at,
  ref_no, gateway_txn_id NULL, gateway_provider NULL, pci_token NULL, status)
payment_allocations(id, payment_id FK, invoice_id FK, amount)
shipment_costs(id, shipment_id FK, charge_id, supplier_party_id, amount, currency, fx_rate,
  status, purchase_invoice_id NULL)
fx_rates(id, tenant_id NULL, base_ccy, quote_ccy, rate, rate_date)
ledger_entries(id, tenant_id, entry_no, entry_date, description, shipment_id NULL,
  lines JSONB, source, posted BOOL)
accounts(id, tenant_id, code, name, type, parent_id)

-- === TRACKING ===
tracking_devices(id, tenant_id, vehicle_id FK NULL, imei, provider, status)
vehicles(id, tenant_id, type, plate_no, driver_user_id NULL, capacity_kg, status)
gps_positions(id, device_id FK, lat, lng, speed, heading, recorded_at, source[device|browser])
geofences(id, tenant_id, name, polygon GEOJSON, alert_on)

-- === COURIER ===
courier_orders(id, tenant_id, order_no, customer_party_id, pickup_address JSONB,
  delivery_address JSONB, parcel JSONB, cod_amount NULL, service, status,
  courier_user_id NULL, route_id NULL,
  delivery_proof JSONB{type[signature_canvas|photo|otp], data, otp_verified_at},
  promised_at, delivered_at)
courier_routes(id, tenant_id, courier_user_id, date, stops JSONB, status)

-- === AGENT ===
agent_agreements(id, tenant_id, agent_party_id, commission_pct, settlement_cycle, currency, status)
agent_settlements(id, agreement_id FK, period_from, period_to, total_commission, status, statement_file_id)

-- === CRM ===
crm_leads(id, tenant_id, source, company, contact_name, phone, email, status, owner_user_id, converted_party_id NULL)
crm_activities(id, tenant_id, party_id NULL, lead_id NULL, type, subject, due_at, done_at, owner_user_id)
loyalty_accounts(id, tenant_id, party_id, points INT, tier)
loyalty_transactions(id, account_id, delta, reason, shipment_id NULL)

-- === AGENCY LINE ===
shipping_line_contracts(id, tenant_id, shipping_line_party_id FK, mode, valid_from, valid_to,
  freight_rate, surcharges JSONB, special_conditions TEXT, status[active|expired|draft])
agency_bills_of_lading(id, tenant_id, shipment_id FK, doc_no, direction[import|export],
  bl_type[original|telex|surrender|seaway], copies_original INT, digital_signed BOOL,
  qr_code_url, status, issued_at)
agency_performance(id, tenant_id, shipping_line_party_id FK, period_month DATE,
  total_teu INT, revenue NUMERIC, currency)

-- === FEEDER ===
feeder_voyages(id, tenant_id, vessel_name, voyage_no, route_code, departure_port_id FK,
  arrival_port_id FK, etd DATE, eta DATE, atd DATE, ata DATE,
  capacity_teu INT, booked_teu INT, status[planned|open|closed|departed|arrived])
feeder_bookings(id, tenant_id, voyage_id FK, shipment_id FK NULL, customer_party_id FK,
  container_count INT, container_type, cargo_weight_kg, revenue NUMERIC, currency, status)
feeder_costs(id, voyage_id FK, type[fuel|port|crew|other], amount NUMERIC, currency, description)

-- === NVOCC ===
nvocc_containers(id, tenant_id, container_no, type, status[available|booked|in_transit|at_port|returned],
  current_location_id FK NULL, shipping_line_party_id FK, lease_start DATE, lease_end DATE,
  lease_cost_per_day NUMERIC, currency, last_event_at)
nvocc_allocations(id, tenant_id, container_id FK, shipment_id FK, allocated_at, released_at)
nvocc_demurrage(id, tenant_id, container_id FK, shipment_id FK, free_days INT,
  start_date DATE, end_date DATE, daily_rate NUMERIC, currency, total_amount, status)

-- === IPG MULTI-BANK ===
payment_gateways(id, tenant_id, provider[zarinpal|mellat|parsian|saderat|saman|stripe],
  display_name, config_encrypted JSONB, is_active BOOL, priority INT, is_default BOOL)
payment_transactions(id, tenant_id, gateway_id FK, amount, currency, status,
  gateway_txn_id, gateway_ref, callback_payload JSONB, pci_token, created_at, verified_at)
digital_wallets(id, tenant_id, party_id FK, balance NUMERIC, currency, status)
wallet_transactions(id, wallet_id FK, delta NUMERIC, type[credit|debit|refund], ref_payment_id FK NULL)
qr_payment_requests(id, tenant_id, amount, currency, party_id FK, expires_at, status, scanned_at)

-- === E-COMMERCE ===
product_categories(id, tenant_id, parent_id FK NULL, name_fa, name_en, slug, sort_order)
products(id, tenant_id, category_id FK, sku, name_fa, name_en, description JSONB,
  base_price NUMERIC, currency, stock_qty INT, is_active BOOL)
price_lists(id, tenant_id, name, customer_group[all|distributor|wholesale|retail], valid_from, valid_to)
price_list_items(id, price_list_id FK, product_id FK, price NUMERIC, currency)
discount_campaigns(id, tenant_id, code, type[percent|fixed], value, min_order, valid_from, valid_to, usage_limit)
orders(id, tenant_id, order_no, customer_party_id FK, order_type[cash|installment|wholesale],
  status[pending|confirmed|invoiced|shipped|delivered|cancelled], subtotal, discount, tax, total,
  currency, notes, created_by)
order_items(id, order_id FK, product_id FK, qty INT, unit_price, total)

-- === DISTRIBUTOR ===
distributors(id, tenant_id, party_id FK, level[national|provincial|city], parent_distributor_id FK NULL,
  sales_quota NUMERIC, currency, commission_pct, status)
installment_plans(id, tenant_id, order_id FK, customer_party_id FK,
  total_amount NUMERIC, currency, interest_rate_pct, installments_count INT, start_date DATE)
installment_payments(id, plan_id FK, seq INT, due_date DATE, amount NUMERIC,
  paid_at TIMESTAMP NULL, payment_id FK NULL, status[pending|paid|overdue])

-- === CUSTOMS / COMPETITIVE EDGE ===
customs_declarations(id, tenant_id, shipment_id FK, declaration_type[import|export|transit],
  declaration_no, customs_office, declarant_party_id FK,
  epl_submission_id NULL, epl_status[draft|submitted|approved|rejected],
  epl_submitted_at, epl_approved_at, goods_value NUMERIC, currency, duty_amount NUMERIC,
  tax_amount NUMERIC, total_payable NUMERIC, status, notes)
customs_declaration_items(id, declaration_id FK, line_no INT, hs_code, description,
  qty NUMERIC, unit, unit_value NUMERIC, currency, country_of_origin, net_weight_kg)
customs_documents(id, declaration_id FK, doc_type, file_id FK, uploaded_at)

-- === ROAD TRANSPORT PERMIT / COMPETITIVE EDGE ===
road_permits(id, tenant_id, shipment_id FK, vehicle_id FK, permit_no,
  permit_type[domestic|international|oversize|hazmat], issued_by_authority,
  valid_from DATE, valid_to DATE, status[pending|issued|expired|revoked],
  rahdarico_ref NULL, naaja_ref NULL, file_id FK NULL)

-- === CARGO INSURANCE / COMPETITIVE EDGE ===
insurance_policies(id, tenant_id, shipment_id FK, policy_no, insurer_party_id FK,
  coverage_type[all_risk|named_perils|total_loss], insured_value NUMERIC, currency,
  premium NUMERIC, premium_currency, valid_from DATE, valid_to DATE,
  status[quoted|active|claimed|expired], notes)
insurance_claims(id, policy_id FK, claim_no, incident_date DATE, incident_description TEXT,
  claimed_amount NUMERIC, approved_amount NUMERIC NULL, status[open|submitted|approved|rejected|paid])

-- === POSTAL / COMPETITIVE EDGE ===
postal_shipments(id, tenant_id, shipment_id FK NULL, courier_order_id FK NULL,
  tracking_code, provider[post_iran|chapar|tipax], service_type,
  sender_info JSONB, recipient_info JSONB, declared_weight_kg NUMERIC,
  declared_value NUMERIC, status, last_tracking_event JSONB, last_checked_at)

-- === NOTIFICATIONS ===
notifications(id, tenant_id, user_id NULL, party_id NULL,
  channel[sms|email|inapp|webpush], template_code, payload JSONB, status, sent_at)
notification_templates(id, tenant_id, code, channel, lang, subject, body)
web_push_subscriptions(id, tenant_id, user_id FK, endpoint, p256dh, auth, created_at)
webhooks(id, tenant_id, url, secret, events TEXT[], active)
api_keys(id, tenant_id, name, key_hash, scopes TEXT[], last_used_at)

-- === ONBOARDING ===
onboarding_checklists(id, tenant_id, step_code, status[pending|done|skipped], completed_at, data JSONB)
```

## ۳.۴ طراحی API

```
Auth:         POST /auth/login | POST /auth/refresh | POST /auth/mfa/verify | POST /auth/logout
Tenant:       GET/PATCH /tenant | GET/PATCH /tenant/branding | GET /tenant/modules | CRUD /branches | CRUD /users | CRUD /roles
Parties:      CRUD /parties | CRUD /parties/:id/contacts | GET /parties/:id/statement
Pricing:      CRUD /rate-cards | POST /rate-cards/import | CRUD /quotations | POST /quotations/:id/send | POST /quotations/:id/accept | POST /quotations/:id/convert
Shipments:    CRUD /shipments | POST /shipments/:id/legs | POST /shipments/:id/containers | POST /shipments/:id/milestones | POST /shipments/:id/status | GET /shipments/:id/profitability | POST /shipments/:id/close
Docs:         GET /shipments/:id/documents | POST /documents | POST /documents/:id/finalize | POST /documents/:id/amend | GET /documents/:id/pdf | GET /documents/:id/qr
Finance:      CRUD /invoices | POST /invoices/:id/approve | CRUD /payments | POST /payments/:id/allocate | CRUD /shipment-costs | GET /reports/ar-aging | GET /reports/ap-aging | CRUD /fx-rates | GET /ledger/entries
Tracking:     GET /tracking/shipments/:id | POST /tracking/positions | GET /vehicles/live | CRUD /geofences | GET /public/track/:file_no
Courier:      CRUD /courier/orders | POST /courier/orders/:id/assign | POST /courier/orders/:id/status | POST /courier/routes/optimize | POST /courier/orders/:id/pod
Agency Line:  CRUD /agency/contracts | CRUD /agency/bls | GET /agency/performance | POST /agency/bls/:id/finalize
Feeder:       CRUD /feeder/voyages | POST /feeder/voyages/:id/bookings | GET /feeder/voyages/:id/manifest | GET /feeder/dashboard
NVOCC:        CRUD /nvocc/containers | POST /nvocc/containers/:id/allocate | POST /nvocc/containers/:id/release | GET /nvocc/demurrage | GET /nvocc/dashboard
IPG:          CRUD /ipg/gateways | POST /payments/initiate | POST /payments/verify | GET /payments/:id/qr | CRUD /wallets | POST /wallets/:id/topup
Ecommerce:    CRUD /store/categories | CRUD /store/products | CRUD /store/price-lists | CRUD /store/campaigns | CRUD /orders | GET /orders/:id/invoice
Distributor:  CRUD /distributors | GET /distributors/:id/performance | CRUD /installment-plans | GET /installment-plans/:id/schedule | POST /installment-payments/:id/pay
Customs:      CRUD /customs/declarations | POST /customs/declarations/:id/submit-epl | GET /customs/declarations/:id/epl-status | CRUD /customs/declarations/:id/documents
RoadPermit:   CRUD /road-permits | POST /road-permits/:id/submit | GET /road-permits/:id/status
Insurance:    CRUD /insurance/policies | POST /insurance/policies/:id/activate | CRUD /insurance/claims
Postal:       POST /postal/shipments | GET /postal/shipments/:id/track | POST /postal/shipments/:id/refresh
Agents:       GET /agent/shipments | POST /agent/shipments/:id/milestones | GET /agent/settlements
CRM:          CRUD /crm/leads | POST /crm/leads/:id/convert | CRUD /crm/activities | GET /customer-portal/*
BI:           GET /bi/dashboards/:code | GET /bi/kpis | POST /bi/reports/run | GET /bi/scenarios/simulate
Onboarding:   GET /onboarding/checklist | POST /onboarding/steps/:code/complete
Platform:     CRUD /platform/tenants | CRUD /platform/plans | POST /platform/tenants/:id/impersonate
Notif:        GET /notifications | CRUD /notification-templates | POST /webpush/subscribe | CRUD /webhooks | CRUD /api-keys
```

## ۳.۵ یکپارچگی‌های بیرونی (External Integrations)

| سیستم بیرونی | نوع اتصال | پروتکل/استاندارد | ماژول(ها) | محیط MVP |
| --- | --- | --- | --- | --- |
| درگاه پرداخت چندبانکی (Zarinpal/Mellat/Parsian/...) | REST API | HTTPS + TLS 1.3 | IPG | Mock adapter + Zarinpal واقعی |
| Core Banking (انتقال وجه خودکار) | REST/SOAP | HTTPS + mTLS | Finance | Mock |
| خطوط کشتیرانی (نرخ/وضعیت) | REST Webhook | HTTPS | Agency Line, Feeder | Stub + webhook ingest |
| بنادر/ترمینال (وضعیت تخلیه/بارگیری) | REST API | HTTPS | NVOCC | Mock + دستی |
| GPS/GNSS دستگاه‌های ردیابی | HTTP batch ingest | HTTPS POST (batch ۵۰۰) | Tracking | واقعی |
| GPS مرورگر راننده | Browser Geolocation API | HTTPS + WebSocket | Courier PWA | واقعی |
| IoT سنسور (دما/رطوبت) | MQTT 5.0 | TLS + MQTT broker | Tracking/NVOCC | Mock broker |
| پیامک | REST API | HTTPS | همه | Kavenegar واقعی |
| ایمیل | SMTP/REST | TLS SMTP یا API | همه | Mailhog در dev |
| Web Push Notification | Web Push Protocol (VAPID) | HTTPS | همه | واقعی |
| امضای دیجیتال MVP | Canvas + SHA-256 hash | HTTPS | Documents, Courier | واقعی |
| امضای دیجیتال فاز ۳ | PKI REST API | HTTPS + Certificate | Documents | Mock |
| QR Code پرداخت/بارنامه | QR Code generation (qrcode lib) | — | IPG, Documents | واقعی |
| سامانه مودیان (مالیاتی) | REST API | HTTPS | Finance | Mock + صف retry |
| سامانه جامع گمرکی GMPCS/EPL | REST/SOAP | HTTPS + auth token | Customs | Mock → واقعی فاز ۲ |
| سامانه راهداری + ناجا | REST/SOAP | HTTPS | Road Permit | Mock → واقعی فاز ۲ |
| بیمه باربری (شرکت‌های بیمه) | REST API | HTTPS | Insurance | Mock adapter |
| پست ایران (tracking API) | REST API | HTTPS | Postal | Mock → واقعی فاز ۲ |
| نقشه/ژئوکدینگ | REST API | HTTPS | Tracking, Courier | Neshan/[Map.ir](http://Map.ir) واقعی |
| نرخ ارز | REST API | HTTPS | Finance | دستی + API روزانه |

---

# بخش ۴: سیستم نقش و دسترسی (RBAC + Feature Gating)

## نقش‌های سیستمی (۱۳ نقش)

1. **Super Admin (پلتفرم)** — مدیریت tenantها، پلن‌ها، مانیتورینگ
2. **Org Admin (ادمین سازمان)** — کاربران، نقش‌ها، شعب، تنظیمات، White-label برندینگ
3. **Operations Manager (مدیر عملیات)** — همه پرونده‌ها + تایید وضعیت‌های حساس
4. **Operator (کارشناس عملیات)** — پرونده‌های شعبه/تخصیص‌یافته خود
5. **Accountant (حسابدار)** — فاکتور، پرداخت، تسویه، گزارش مالی
6. **Sales (بازرگانی/فروش)** — CRM، استعلام، مشاهده پرونده مشتریان خود
7. **Agent (نماینده/ایجنت داخلی)** — پرونده‌های تخصیصی؛ milestone و سند
8. **Driver/Courier (راننده/پیک — PWA مرورگر)** — ماموریت‌های خود؛ وضعیت + POD از مرورگر
9. **Customer (مشتری نهایی — Customer Portal PWA)** — پرونده‌ها، اسناد، صورتحساب، رهگیری
10. **🆕 Shipping Line Agent (نماینده کشتیرانی — Agency Line Portal PWA)** — قراردادها، بارنامه‌ها، عملکرد نمایندگی
11. **🆕 Distributor (نماینده فروش — Distributor Portal PWA)** — سفارش‌گذاری، اقساط، گزارش فروش، کمیسیون
12. **🆕 Contractor (پیمانکار — Contractor Portal PWA)** — پرونده‌های تخصیصی، milestone، اسناد، صورتحساب
13. **🆕 External Carrier (خط کشتیرانی — Carrier Portal PWA)** — مشاهده پرونده‌های مرتبط، تأیید milestone، بارنامه

## ماتریس دسترسی (V=مشاهده, C=ایجاد, E=ویرایش, D=حذف, A=تایید)

| ماژول | OrgAdmin | OpsMgr | Operator | Accountant | Sales | Agent | Driver | Customer | ShippingLineAgent | Distributor | Contractor | ExtCarrier |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| پرونده حمل | VCEDA | VCEDA | VCE | V | V(خود) | V/E-ms | — | V(خود) | V(خود) | — | V(خود) | V(خود) |
| قیمت/استعلام | VCED | VA | V | V | VCE | — | — | V(خود) | — | VC | — | — |
| اسناد | VCEDA | VCEA | VCE | V | V | VC | — | V(نهایی) | V(خود) | V | V | V |
| مالی AR/AP | V | V | V(هزینه) | VCEA | V(فاکتور) | V(صورتحساب) | — | V(خود) | — | V(خود) | V(خود) | — |
| رهگیری | VCED | VCE | VCE | — | V | V | C(موقعیت) | V(خود) | V(خود) | — | V(خود) | V(خود) |
| کوریری | VCED | VCEA | VCE | V | V | — | V/E(خود) | VC | — | — | — | — |
| Agency Line | VCEDA | VA | V | V | — | — | — | — | VCEA | — | — | V |
| Feeder | VCEDA | VCEA | VCE | V | — | — | — | — | V | — | — | V |
| NVOCC | VCEDA | VCEA | VCE | V | — | — | — | — | V | — | — | V |
| IPG/پرداخت | VCEDA | — | — | VCEA | — | — | — | VC(خود) | — | VC | — | — |
| E-Commerce | VCEDA | V | — | V | VCED | — | — | VC | — | VC | — | — |
| Distributor | VCEDA | V | — | V(تسویه) | VCE | — | — | — | — | V(خود) | — | — |
| CRM | VCED | V | — | — | VCED | — | — | — | — | V | — | — |
| BI/گزارش | V(همه) | V(عملیات) | V(شعبه) | V(مالی) | V(فروش) | V(خود) | — | — | V(خود) | V(خود) | — | — |
| Customs | VCEDA | VA | VCE | V | — | — | — | — | — | — | V | — |
| Road Permit | VCEDA | VA | VCE | — | — | — | V | — | — | — | V | — |
| Insurance | VCEDA | VA | VCE | V | V | — | — | V(خود) | — | — | — | — |
| Postal | VCEDA | VCE | VCE | — | — | — | — | V(خود) | — | — | — | — |
| Onboarding | VCEDA | — | — | — | — | — | — | — | — | — | — | — |
| تنظیمات سازمان | VCEDA | — | — | — | — | — | — | — | — | — | — | — |

## Feature Gating بر اساس پلن

- **Starter:** Core+Docs+Finance-lite+Onboarding
- **Pro:** +Tracking+CRM+CustomerPortal+IPG+Courier+AgencyLine
- **Enterprise:** همه + NVOCC + Feeder + Distributor + Customs + Insurance + RoadPermit + Postal + API + BI-Advanced + White-label
- limitها در interceptor بررسی؛ در ۸۰٪ سقف اعلان ارسال می‌شود

---

# بخش ۵: ماژول‌ها و فیچرهای ریز

<aside>
⚠️

قاعده کلی همه فرم‌ها: فیلدهای الزامی با اعتبارسنجی Zod (client) و class-validator (server)؛ اعداد پولی NUMERIC(18,2)؛ تاریخ‌ها با تقویم دوگانه شمسی/میلادی (dayjs + jalali)؛ همه فهرست‌ها دارای جستجو، فیلتر، مرتب‌سازی، خروجی Excel. همه پنل‌های خارجی (مشتری/نماینده/راننده/توزیع‌کننده) به‌صورت PWA با route-based یا subdomain جداگانه هستند.

</aside>

## ۵.۱ هسته عملیات حمل (Shipment Core)

**صفحات:** فهرست پرونده‌ها (جدول + کانبان)، جزئیات (تب‌ها: کلی، لگ‌ها، کانتینر/کالا، milestoneها، اسناد، مالی، گمرک، بیمه، تاریخچه)، فرم ایجاد/ویرایش، تقویم ETD/ETA، صفحه بستن پرونده.

**فرم ایجاد پرونده:** direction (الزامی)، mode (الزامی)، service_type (الزامی، وابسته به mode)، customer (autocomplete، الزامی)، shipper/consignee (الزامی)، shipping_line (اختیاری، برای دریایی)، incoterm (الزامی برای بین‌المللی)، origin/destination (الزامی)، HS code پیش‌فرض، cargo_ready_date، ETD/ETA (ETA ≥ ETD)، operator، salesperson، branch (الزامی)، remarks (≤۲۰۰۰ کاراکتر)، customs_required BOOL، insurance_required BOOL.

**Workflow:** draft → booked (≥۱ leg + carrier) → in_transit (milestone DEPARTED) → arrived → delivered (نیاز POD) → closed (همه هزینه actual + فاکتورها approved + تایید OpsMgr). Cancel قبل از in_transit با دلیل.

**Edge Caseها:** ۱) تغییر mode بعد از صدور سند → مسدود؛ ۲) کانتینر تکراری در پرونده باز → هشدار؛ ۳) ETA < ETD → خطا؛ ۴) حذف پرونده دارای فاکتور → ممنوع؛ ۵) دو اپراتور همزمان → optimistic locking؛ ۶) multimodal بدون leg → block Booking؛ ۷) سقف پلن → block + CTA ارتقا؛ ۸) پرونده نیاز گمرک بدون شماره اظهارنامه → هشدار مانع Close.

**Acceptance:** Given پرونده draft با leg دریایی، When اپراتور Booking می‌زند، Then وضعیت booked، milestone BOOKED ثبت، اعلان SMS/Email به مشتری ارسال می‌شود.

## ۵.۲ قیمت‌گذاری و استعلام نرخ (Pricing & Quotation)

[محتوای کامل اصلی حفظ شده — بدون تغییر نسبت به نسخه قبل]

**صفحات:** فهرست نرخ‌نامه‌ها، import اکسل، فهرست استعلام‌ها، فرم استعلام، مقایسه نرخ کریرها، پیش‌نمایش PDF.

**فرم استعلام:** مشتری (الزامی)، mode/service (الزامی)، مبدا/مقصد (الزامی)، مشخصات بار (الزامی)، اینکوترمز، اعتبار تا (پیش‌فرض ۷ روز)، ارز (الزامی). خطوط قیمت: charge (الزامی)، تعداد×قیمت (>۰)، قیمت خرید (اختیاری). اگر مارجین < حداقل سازمان (پیش‌فرض ۵٪) → نیاز به تایید OpsMgr.

**Workflow:** draft → send (PDF دوزبانه + ایمیل/SMS) → accepted/rejected/expired → convert-to-shipment.

**Edge Caseها:** ۱) نرخ‌نامه منقضی → هشدار + پیشنهاد جایگزین؛ ۲) تبدیل مجدد → ممنوع؛ ۳) ارز متفاوت → تبدیل با fx_rate؛ ۴) accept بعد از valid_until → خطا؛ ۵) import اکسل خراب → گزارش ردیف‌به‌ردیف.

**Acceptance:** Given استعلام accepted، When Convert می‌زند، Then پرونده draft با خطوط مالی estimated ساخته و استعلام قفل می‌شود.

## ۵.۳ مدیریت اسناد حمل (Documentation)

**صفحات:** آرشیو اسناد، فرم صدور (per type)، پیش‌نمایش PDF زنده، مدیریت قالب (لوگو، سربرگ، مهر)، فهرست در انتظار.

**فرم HBL:** shipper/consignee/notify (prefill)، شماره سند (خودکار)، vessel/voyage (الزامی)، بنادر بارگیری/تخلیه (الزامی)، شرح کالا (الزامی)، freight terms، تعداد نسخه اصلی (۱-۳)، تاریخ صدور، زبان (fa/en/bilingual). **QR Code:** در صدور نهایی، QR Code حاوی URL رهگیری عمومی به PDF اضافه می‌شود.

**Workflow:** generate (prefill) → draft → finalize (PDF + sha256 + QR Code + شماره قطعی + قفل) → amend (version+1) → void با دلیل. Manifest: چند پرونده هم‌کشتی/هم‌پرواز → سند تجمیعی.

**Edge Caseها:** ۱) MBL بدون carrier → خطا؛ ۲) دو HBL همزمان → هشدار + تایید؛ ۳) فونت فارسی در PDF → embed اجباری؛ ۴) amend سند با سند وابسته → هشدار زنجیره؛ ۵) حذف فایل → گزارش سلامت + re-generate از snapshot.

**Acceptance:** Given پرونده booked، When HBL finalize می‌شود، Then PDF دوزبانه با QR Code و hash تولید، در پرتال مشتری PWA قابل دانلود می‌شود.

## ۵.۴ مالی و حسابداری (AR/AP)

[محتوای کامل اصلی حفظ شده]

**صفحات:** هزینه‌های پرونده، فاکتور فروش/خرید، دریافت/پرداخت، Statement، سود/زیان پرونده، AR/AP Aging، نرخ ارز، دفتر روزنامه، صف مالیاتی.

**قواعد:** فاکتور approved غیرقابل ویرایش؛ ledger دوطرفه هنگام approve و payment؛ چک اعتبار بدهی باز مشتری.

**Edge Caseها:** ۱) تخصیص بیش از مانده → خطا؛ ۲) پرداخت ارزی به فاکتور ریالی → سود/زیان تسعیر؛ ۳) void با پرداخت → ممنوع تا لغو؛ ۴) هزینه actual بدون فاکتور ۳۰ روز → آلارم؛ ۵) ارسال مالیاتی شکست → retry backoff؛ ۶) banker's rounding.

**Acceptance:** Given فاکتور ۱۰۰ + پرداخت ۶۰ + پرداخت ۴۰، When تخصیص کامل، Then paid + مانده صفر + ledger balanced.

## ۵.۵ رهگیری و مکان‌یابی زنده

**صفحات:** نقشه زنده ناوگان، timeline milestone، صفحه عمومی (بدون login)، مدیریت دستگاه‌ها/خودروها، geofenceها، گزارش تاخیر.

**ورودی داده:** ۱) دستی (فرم milestone)؛ ۲) API ingest دستگاه GPS (batch ۵۰۰)؛ ۳) **Browser Geolocation API** از PWA راننده (watchPosition با accuracy ≥ ۵۰m)؛ ۴) وبهوک کریر؛ ۵) MQTT از سنسور IoT (دما/رطوبت کانتینر reefer — آلارم اگر دما از بازه خارج شد).

**Edge Caseها:** ۱) clock skew → رد + لاگ؛ ۲) پرش GPS >300km/h → فیلتر؛ ۳) قطع شبکه → آخرین موقعیت stale (>۳۰ دقیقه) با برچسب؛ ۴) صفحه عمومی → rate-limit ۱۰ req/min/IP؛ ۵) milestone ترتیب اشتباه → هشدار؛ ۶) سنسور دما بیرون از بازه → push notification فوری به OpsMgr.

**Acceptance:** Given راننده با PWA در مرورگر موبایل، When watchPosition event آید، Then نقشه زنده ≤ ۵ ثانیه (WebSocket) به‌روز و در gps_positions با source='browser' ذخیره می‌شود.

## ۵.۶ کوریری و پست سریع (Last-mile)

**صفحات:** ثبت سفارش، دیسپچ‌بورد، مسیرهای روزانه، **Driver PWA** (صفحه وب واکنش‌گرا در مرورگر موبایل: فهرست ماموریت، جزئیات، دکمه‌های وضعیت، ثبت POD با Canvas signature pad وب، عکس با Camera API، OTP).

**فرم POD (وب):** Canvas signature pad (امضای انگشتی مرورگر)، یا عکس با `<input type=file accept=image/*>`، یا OTP پیامکی ۵ رقمی (انقضای ۱۰ دقیقه). موقعیت راننده: Browser Geolocation API (`navigator.geolocation.watchPosition`).

**Workflow:** registered → assign → picked_up → at_hub (اختیاری) → out_for_delivery → delivered/failed → returned.

**Edge Caseها:** ۱) راننده آفلاین → PWA صف offline با IndexedDB + sync بعدی (idempotent)؛ ۲) COD مغایرت → گزارش تسویه؛ ۳) لغو بعد از pickup → returned + هزینه برگشت؛ ۴) تحویل دوباره → idempotency؛ ۵) آدرس بدون lat/lng → geocode خودکار.

**Acceptance:** Given سفارش out_for_delivery با COD، When راننده از Browser Canvas امضا ثبت می‌کند، Then delivered، POD در S3 ذخیره، پیامک نظرسنجی ارسال، COD به تسویه اضافه می‌شود.

## ۵.۷ پرتال شرکا و نمایندگان (Agent Portal PWA)

[محتوای کامل اصلی حفظ شده — به PWA تبدیل شده]

**صفحات:** داشبورد نماینده، پرونده‌های تخصیصی، milestone، بارگذاری سند، تسویه کمیسیون، پروفایل قرارداد. همه صفحات PWA واکنش‌گرا در `/portal/agent/...`.

**Edge Caseها:** ۱) دید قیمت خرید/فروش → مسدود با field-level permission؛ ۲) milestone با تاریخ آینده → رد؛ ۳) تغییر کمیسیون وسط دوره → نسخه‌بندی + pro-rata؛ ۴) دو نماینده برای یک پرونده → ممنوع؛ ۵) غیرفعال‌سازی نماینده → قطع فوری دسترسی.

**Acceptance:** Given ۵ پرونده delivered، When settlement تولید، Then Σ(درآمد×درصد) با جزئیات per-shipment و پس از approve قفل.

## ۵.۸ CRM و پرتال مشتری (Customer Portal PWA)

**پرتال مشتری PWA** در `/portal/customer/...` با قابلیت Add to Home Screen. صفحات: داشبورد، پرونده‌های من + رهگیری، اسناد (دانلود PDF با QR)، صورتحساب + پرداخت IPG، استعلام جدید، تیکت پشتیبانی، باشگاه (امتیاز/سطح).

**Edge Caseها:** ۱) سرنخ تکراری → merge؛ ۲) IPG callback نامعتبر → verify دوطرفه؛ ۳) کاربر چند شرکت → انتخاب context؛ ۴) دانلود draft → ممنوع؛ ۵) opt-out پیامک → فلگ.

**Acceptance:** Given فاکتور sent، When پرداخت IPG موفق، Then payment ثبت، تخصیص خودکار، رسید PDF و پیامک تایید.

## ۵.۹ داشبورد مدیریتی و BI

**صفحات:** داشبورد CEO (KPI: پرونده باز/بسته، درآمد/هزینه/سود، تاخیر، Top مشتریان/مسیرها)، داشبورد COO (پرونده‌های معوق، milestone عقب‌افتاده)، داشبورد مالی (AR/AP aging، جریان نقد)، گزارش‌ساز با **Drill-down**، **شبیه‌سازی سناریو** (what-if: تغییر نرخ ارز/حجم پرونده → اثر بر سود)، خروجی Excel/PDF.

**Drill-down:** هر widget کلیک‌پذیر → نمایش داده‌های جزئی همان بازه/دسته.

**Scenario Simulation:** انتخاب متغیر (نرخ ارز، حجم پرونده ماهانه، نرخ کمیسیون)، تعریف مقدار فرضی، محاسبه آنی تأثیر بر P&L بدون تغییر داده واقعی.

**Edge Caseها:** ۱) بازه >۲ سال → گروه ماهانه اجباری؛ ۲) گیرنده غیرفعال → skip؛ ۳) ارز مخلوط → تبدیل به ارز پایه؛ ۴) فیلد ممنوع → فیلتر ستون؛ ۵) timeout → صف async.

**Acceptance:** Given 100k پرونده، When داشبورد CEO باز، Then KPIها ≤۲ ثانیه از materialized view.

## ۵.۱۰ تنظیمات سازمانی، پلن‌ها و White-label

**صفحات:** پروفایل سازمان (لوگو، سربرگ، ارز پایه، تقویم)، **تنظیمات White-label** (لوگو PWA، رنگ primary/secondary، نام نمایشی، custom domain، آیکون PWA برای Add to Home Screen)، شعب، کاربران/نقش‌ها، قالب شماره‌گذاری، قالب اعلان، مدیریت پلن، API keys، وبهوک، **Audit Log Viewer** (جستجو، فیلتر by entity/user/date، export)، تنظیمات مالی.

**White-label PWA:** هر tenant می‌تواند `manifest.json` را با لوگو، نام و رنگ اختصاصی دریافت کند؛ custom domain با SSL خودکار (Let's Encrypt)؛ تمام پنل‌های PWA (مشتری/نماینده/راننده) با برند tenant نمایش داده می‌شوند.

**Edge Caseها:** ۱) تغییر ارز پایه با داده → ممنوع؛ ۲) حذف شعبه دارای پرونده → فقط غیرفعال؛ ۳) downgrade پلن → read-only؛ ۴) تغییر الگوی شماره → فقط رکوردهای جدید؛ ۵) آخرین OrgAdmin → نمی‌تواند خود را حذف کند.

**Acceptance:** Given پلن Starter، When ماژول NVOCC باز می‌شود، Then 403 `MODULE_NOT_ENABLED` + badge ارتقا.

---

## ۵.۱۱ 🆕 نمایندگی کشتیرانی (Agency Line Module)

**صفحات:** داشبورد نمایندگی (KPI: تعداد BL صادره، TEU ماهانه، قراردادهای فعال)، فهرست قراردادهای کشتیرانی، فرم قرارداد، فهرست بارنامه‌ها (صادراتی/وارداتی)، فرم صدور BL، جستجوی پیشرفته BL، گزارش عملکرد per خط کشتیرانی، Carrier Portal PWA (دسترسی خط کشتیرانی به داده‌های خودش).

**فرم قرارداد کشتیرانی:**

- shipping_line (autocomplete از parties نوع shipping_line، الزامی)
- mode (sea/feeder، الزامی)
- valid_from / valid_to (تاریخ، الزامی، valid_to > valid_from)
- freight_rate (NUMERIC، الزامی)
- currency (الزامی)
- surcharges (JSONB: array از {code, name, amount, currency} — BAF، CAF، PSS و غیره)
- special_conditions (متن)
- status (draft/active)

**فرم صدور بارنامه نمایندگی (Agency BL):**

- shipment (ربط به پرونده، الزامی)
- doc_no (خودکار با الگوی تنظیم‌شده، قابل override)
- direction (import/export، الزامی)
- bl_type (original/telex/surrender/seaway، الزامی)
- copies_original (عدد ۱-۳، الزامی)
- تمام فیلدهای BL استاندارد (shipper/consignee/notify، vessel/voyage، بنادر، شرح کالا)
- digital_signed BOOL (امضای Canvas وب + hash SHA-256)
- QR Code (خودکار پس از finalize)
- ارسال خودکار به مشتری و دفتر مرکزی (ایمیل + In-app)

**Workflow:** draft → finalize (PDF + QR + hash + قفل) → ارسال خودکار → amend (نسخه جدید) → void. مدیریت گروهی BL: انتخاب چند BL → صدور دسته‌ای → ارسال bulk.

**Edge Caseها:**

1. قرارداد منقضی هنگام صدور BL → هشدار «قرارداد فعالی با این خط وجود ندارد» + پیشنهاد تمدید
2. BL با شماره تکراری در همان خط/دوره → خطا (unique constraint)
3. BL صادره برای پرونده‌ای که هنوز booked نیست → هشدار غیرمسدودکننده
4. تغییر نرخ قرارداد وسط دوره → نسخه‌بندی (version+1)، BLهای قبلی با نرخ قبلی محاسبه
5. Carrier Portal: خط کشتیرانی نباید اطلاعات قراردادهای خط رقیب را ببیند (field-level isolation)
6. BL void شده نباید در manifest منتشر شود

**Acceptance:**

- Given قرارداد فعال با Maersk، When BL finalize، Then PDF با QR Code تولید، hash در DB، BL به مشتری ایمیل و در Carrier Portal قابل مشاهده.
- Given خط کشتیرانی با نقش ExtCarrier login، When لیست BL را می‌بیند، Then فقط BLهای مرتبط با خود آن خط نمایش داده می‌شود.

---

## ۵.۱۲ 🆕 عملیات فیدری (Feeder Operations Module)

**صفحات:** داشبورد Feeder (voyageهای باز، ظرفیت، درآمد/هزینه)، فهرست voyageها، فرم voyage جدید، تخصیص بارگیری (booking)، Manifest فیدری، گزارش هزینه/سودآوری سفر، مقایسه ظرفیت ناوگان.

**فرم Feeder Voyage:**

- vessel_name (الزامی)
- voyage_no (الزامی، یکتا در tenant)
- route_code (کد مسیر از locations، الزامی)
- departure_port_id / arrival_port_id (الزامی)
- etd / eta (الزامی، eta > etd)
- capacity_teu (عدد صحیح مثبت، الزامی)
- status (planned/open/closed/departed/arrived)

**فرم بارگیری (Feeder Booking):**

- voyage_id (الزامی)
- shipment_id (اختیاری — لینک به پرونده)
- customer_party_id (الزامی)
- container_count (الزامی، صحیح مثبت)
- container_type (الزامی)
- cargo_weight_kg (الزامی)
- revenue (الزامی)، currency
- status

**فرم هزینه سفر:** type (fuel/port/crew/other)، amount، currency، description.

**Workflow:** voyage planned → open (پذیرش booking) → closed (ظرفیت تکمیل یا تاریخ قطع) → departed (ETD رسید) → arrived (ETA رسید + تخلیه).

**Edge Caseها:**

1. اضافه‌بارگیری (booked_teu > capacity_teu) → block با خطای «ظرفیت تکمیل»
2. voyage departed ولی ETA هنوز نرسیده → وضعیت in-transit + milestone tracking
3. لغو booking بعد از departed → ممنوع، فقط void با دلیل و گزارش مغایرت
4. سفر تکراری (همان vessel + voyage_no) → خطای unique
5. هزینه سوخت وارد‌شده بیش از ۵۰٪ کل درآمد → هشدار به مدیر
6. voyage بدون هیچ booking → هشدار «سفر خالی» در داشبورد

**Acceptance:**

- Given voyage با ظرفیت ۱۰۰ TEU و booked_teu=۹۸، When booking جدید با ۳ TEU، Then خطای 422 «ظرفیت کافی نیست».
- Given voyage arrived، When status به arrived تغییر می‌کند، Then milestone ARRIVED برای همه shipmentهای مرتبط ثبت و اعلان ارسال می‌شود.

---

## ۵.۱۳ 🆕 مدیریت NVOCC و کانتینر (NVOCC & Container Management)

**صفحات:** داشبورد کانتینر (نقشه موقعیت کانتینرها، KPI: موجودی/اجاره/demurrage)، فهرست کانتینرها با فیلتر (وضعیت/نوع/مکان)، فرم کانتینر، تخصیص به پرونده، release، محاسبه demurrage، گزارش سودآوری NVOCC.

**فرم کانتینر:**

- container_no (regex ISO6346 + check digit، الزامی)
- type (20DV/40DV/40HC/20RF/40RF/...، الزامی)
- shipping_line_party_id (خط کشتیرانی اجاره‌دهنده، الزامی)
- lease_start / lease_end (الزامی)
- lease_cost_per_day (NUMERIC، الزامی)، currency
- current_location_id (port/terminal/warehouse)
- status (available/booked/in_transit/at_port/returned)

**فرم تخصیص (Allocation):**

- container_id (الزامی)
- shipment_id (الزامی)
- free_days INT (روزهای فرانشیز — الزامی برای محاسبه demurrage)

**محاسبه Demurrage:** از آخرین روز free_days تا تاریخ release: `days × daily_rate`. هشدار خودکار وقتی free_days - ۳ روز باقیمانده.

**Workflow:** کانتینر available → allocated (تخصیص به shipment) → in_transit → at_port → returned (release).

**Edge Caseها:**

1. تخصیص کانتینر در حال استفاده (not available) → block
2. lease_end رسیده ولی کانتینر هنوز allocated → هشدار هر روز به OpsMgr + محاسبه جریمه
3. شماره کانتینر اشتباه (check digit) → خطای اعتبارسنجی
4. نرخ demurrage تغییر کند وسط اجاره → نسخه‌بندی نرخ (نرخ قدیم برای روزهای قبل)
5. کانتینر reefer بدون سنسور IoT → هشدار «رهگیری دما غیرفعال»
6. release قبل از تخلیه کامل → هشدار تأییدیه

**Acceptance:**

- Given کانتینر NVOCC با ۷ روز free و تخصیص به shipment، When روز ۸ ام release نشده، Then demurrage = ۱×daily_rate در حساب و اعلان به OpsMgr.
- Given کانتینر reefer با سنسور IoT، When دما از بازه (-18 تا 0) خارج شد، Then push notification فوری به OpsMgr و milestone TEMP_ALERT ثبت.

---

## ۵.۱۴ 🆕 پرتال‌های وب White-label و PWA (Web Portals & White-label PWA)

**توضیح:** این ماژول پنل‌های وب اختصاصی با قابلیت White-label برای ذی‌نفعان خارجی است. **هیچ اپلیکیشن نیتیو Android/iOS وجود ندارد.** همه پنل‌ها PWA (Progressive Web App) هستند.

**پنل‌های PWA:**

- `/portal/customer` — مشتری نهایی
- `/portal/agent` — نماینده/ایجنت داخلی
- `/portal/driver` — راننده/پیک (GPS مرورگر)
- `/portal/carrier` — خط کشتیرانی خارجی
- `/portal/distributor` — نماینده فروش
- `/portal/contractor` — پیمانکار

**قابلیت‌های PWA:**

- Service Worker با Workbox 7 (استراتژی: network-first برای API، cache-first برای static assets)
- Offline-first: فرم‌های تحویل/milestone در IndexedDB → sync هنگام اتصال
- Web Push Notification با VAPID
- Add to Home Screen با `manifest.json` اختصاصی per tenant (نام، آیکون، رنگ theme)
- Geolocation API برای راننده (`navigator.geolocation.watchPosition`)
- Camera API برای عکس POD (`<input type=file accept=image/*>`)
- Canvas Signature Pad برای امضای تحویل

**White-label تنظیمات (در پنل Org Admin):**

- برند لوگو (فایل SVG/PNG، ذخیره در S3)
- رنگ primary/secondary (hex picker)
- نام PWA (نمایش روی Home Screen)
- آیکون PWA (512×512 PNG)
- Custom Domain (مثل `app.mycargo.ir`) با SSL خودکار (Let's Encrypt)
- صفحه ورود سفارشی (پس‌زمینه، پیام خوش‌آمد)

**پیاده‌سازی فنی:**

- Next.js App Router با route groups برای هر پنل
- Middleware بررسی `Host` header برای custom domain → بارگذاری تنظیمات tenant
- Dynamic `manifest.json` endpoint: `GET /api/manifest.json` (بر اساس tenant از Host)
- CSS variables برای رنگ برند inject‌شده از API settings
- Subdomain routing (اختیاری): `{tenant}.freightcore.ir` یا custom domain

**Edge Caseها:**

1. Custom domain مشتری با SSL منقضی → اعلان ۳۰ روز قبل از انقضا + auto-renew
2. PWA offline: فرم POD ذخیره IndexedDB → sync → idempotency با local UUID
3. Browser Geolocation denied → fallback به ورود دستی آدرس
4. Service Worker update → toast «نسخه جدید موجود است» + reload
5. دو tenant با custom domain یکسان → block + خطای منحصربه‌فرد
6. پنل راننده بدون HTTPS → Geolocation API کار نمی‌کند (HTTP block مرورگر) → باید HTTPS اجباری

**Acceptance:**

- Given tenant با custom domain `app.mycargo.ir` و لوگو سفارشی، When راننده از موبایل به `/portal/driver` می‌رود، Then پنل با نام و لوگو tenant نمایش داده، GPS مرورگر فعال، Add to Home Screen پیشنهاد می‌شود.
- Given Driver PWA آفلاین، When POD ثبت می‌کند، Then در IndexedDB ذخیره، پس از اتصال sync با idempotency-key یکتا.

---

## ۵.۱۵ 🆕 درگاه پرداخت چندبانکی (IPG Multi-Bank Payment Module)

**صفحات:** مدیریت درگاه‌ها (فهرست + فرم)، تاریخچه تراکنش‌ها (جستجو/فیلتر/export)، مدیریت کیف پول دیجیتال، گزارش مالی IPG، QR Code پرداخت، داشبورد پیگیری تراکنش‌ها.

**فرم مدیریت درگاه (Org Admin):**

- provider (select: zarinpal/mellat/parsian/saderat/saman/stripe، الزامی)
- display_name (الزامی)
- config (فیلدهای رمزنگاری‌شده per provider: merchant_id، terminal_id، private_key)
- is_active BOOL
- priority INT (برای routing خودکار — اول کمترین priority)
- is_default BOOL

**جریان پرداخت:**

1. درخواست پرداخت → انتخاب درگاه (دستی توسط کاربر یا خودکار بر اساس priority و availability)
2. POST /payments/initiate → redirect به درگاه بانکی
3. Callback → POST /payments/verify → تأیید دوطرفه با gateway_txn_id
4. ثبت payment + تخصیص به فاکتور

**روش‌های پرداخت:**

- پرداخت اینترنتی از درگاه بانکی
- پرداخت کارت‌به‌کارت (با ثبت ref دستی + تأیید حسابدار)
- کیف پول دیجیتال (شارژ از IPG → موجودی → پرداخت داخلی)
- QR Code پرداخت (generate → scan → پرداخت در اپ بانک)

**تقسیم پرداخت (Split Payment):** یک payment → تخصیص به چند فاکتور به‌صورت خودکار (اولویت: قدیمی‌ترین فاکتور).

**امنیت PCI DSS:**

- هیچ داده کارت در سرور FreightCore ذخیره نمی‌شود (tokenization توسط PSP)
- HTTPS TLS 1.3 اجباری برای تمام تراکنش‌ها
- Audit log کامل برای هر تراکنش
- Rate limiting: ۵ تلاش پرداخت/دقیقه/IP
- Idempotency key برای جلوگیری از پرداخت مضاعف

**Edge Caseها:**

1. درگاه اصلی down → auto-failover به درگاه بعدی بر اساس priority
2. Callback تکراری (duplicate) → بررسی gateway_txn_id یکتا → ignore duplicate
3. پرداخت موفق ولی تخصیص به فاکتور اشتباه → reconciliation report روزانه
4. QR Code منقضی (پس از ۱۵ دقیقه) → تولید QR Code جدید
5. کیف پول: موجودی ناکافی → پیشنهاد شارژ از IPG
6. Stripe (بین‌المللی): نرخ تبدیل لحظه‌ای → ذخیره fx_rate در لحظه پرداخت

**Acceptance:**

- Given فاکتور ۵۰۰,۰۰۰ تومان، When مشتری از Customer Portal پرداخت می‌کند، Then به درگاه پیش‌فرض redirect، پس از callback موفق payment ثبت، تخصیص خودکار، رسید PDF + Web Push ارسال.
- Given درگاه Mellat down، When پرداخت آغاز می‌شود، Then سیستم به Parsian (priority بعدی) failover و لاگ می‌کند.

---

## ۵.۱۶ 🆕 فروشگاه آنلاین (E-Commerce & Online Sales Portal)

**صفحات:** پنل مدیر فروش (محصولات، دسته‌بندی، لیست قیمت، کمپین)، صفحه فروشگاه (Distributor/Customer Portal)، سبد خرید، checkout، فهرست سفارشات، جزئیات سفارش، داشبورد فروش (KPI: فروش روزانه/هفتگی، محصول پرفروش، مناطق فعال).

**فرم محصول:**

- category_id (درختی، الزامی)
- sku (یکتا در tenant، الزامی)
- name_fa / name_en (الزامی)
- description (JSONB با rich text)
- base_price (NUMERIC، الزامی)، currency
- stock_qty (صحیح ≥ ۰)
- is_active BOOL

**فرم لیست قیمت:**

- name، customer_group (all/distributor/wholesale/retail)
- valid_from / valid_to
- خطوط: product_id + price اختصاصی برای آن گروه

**فرم کمپین تخفیف:**

- code (کوپن یکتا، اختیاری)
- type (percent/fixed)
- value (الزامی)
- min_order (حداقل مبلغ سفارش)
- valid_from/to، usage_limit

**Workflow سفارش:** pending (ثبت) → confirmed (تأیید دستی یا خودکار) → invoiced (فاکتور صادر) → shipped (لینک به shipment) → delivered / cancelled.

**Edge Caseها:**

1. موجودی صفر هنگام checkout → block + اعلان «ناموجود»
2. کمپین منقضی یا سقف استفاده رسیده → خطای کوپن
3. قیمت گروه مشتری vs قیمت پایه: همیشه کمترین نمایش داده شود
4. سفارش اقساطی بدون تأیید اعتبار → pending تا OpsMgr تأیید کند
5. فاکتور سفارش با چند ارز → convert به ارز پایه با fx_rate روز
6. لغو سفارش confirmed → موجودی برگردد + کوپن یک‌بار‌مصرف unlock

**Acceptance:**

- Given Distributor با price_list مخصوص، When checkout، Then قیمت از price_list Distributor اعمال، کوپن ۱۰٪ کسر، فاکتور صادر و موجودی کسر.
- Given سفارش delivered، When لینک به shipment، Then milestone DELIVERED در shipment به‌روز.

---

## ۵.۱۷ 🆕 پرتال توزیع‌کنندگان و فروش اقساطی (Distributor Portal & Installment Sales)

**صفحات:** Distributor Portal PWA در `/portal/distributor/...`، داشبورد (سهمیه فروش، کمیسیون، سفارشات فعال)، فهرست سفارشات، ثبت سفارش جدید، وضعیت اقساط، پرداخت قسط آنلاین، گزارش کمیسیون، مدیریت زیرنمایندگان (اگر level=national).

**فرم تعریف نماینده (Org Admin):**

- party_id (الزامی)
- level (national/provincial/city)
- parent_distributor_id (اختیاری برای سلسله‌مراتب)
- sales_quota (NUMERIC، ارز)
- commission_pct (۰-۱۰۰)
- status

**فرم طرح اقساط (Installment Plan):**

- order_id (الزامی)
- customer_party_id (الزامی)
- total_amount / currency
- interest_rate_pct (۰ برای بدون بهره)
- installments_count (عدد صحیح ۲-۶۰)
- start_date (الزامی)

→ سیستم جدول اقساط (schedule) را خودکار تولید می‌کند

**Workflow اقساط:** plan created → هر قسط: pending → paid (با payment IPG) / overdue (بعد از سررسید).

**هشدار اقساط:** ۳ روز قبل از سررسید → Web Push + SMS به مشتری. overdue → اعلان روزانه به Accountant.

**Edge Caseها:**

1. Distributor سطح city نمی‌تواند زیرنماینده تعریف کند
2. سهمیه فروش ماهانه رسیده → هشدار + block سفارش جدید (اختیاری، per policy)
3. قسط overdue بیش از ۳۰ روز → flag در اعتبار مشتری
4. لغو طرح اقساط بعد از پرداخت اول → محاسبه جریمه لغو + استرداد مابقی
5. تغییر commission_pct → فقط از دوره بعدی؛ گزارش‌های قدیمی تغییر نمی‌کنند
6. Distributor دیدن قیمت خرید سازمان → ممنوع (فقط قیمت فروش به آن گروه)

**Acceptance:**

- Given طرح اقساط ۶ قسطه، When قسط اول از Customer Portal IPG پرداخت، Then installment_payment.status=paid، remaining balance به‌روز، رسید PDF + SMS.
- Given Distributor با sales_quota ماهانه رسیده، When سفارش جدید ثبت، Then هشدار + درخواست تأیید OpsMgr.

---

## ۵.۱۸ 🆕 فرآیند Onboarding مشتری جدید (Customer Onboarding Flow)

**صفحات:** صفحه ثبت‌نام سازمان (public)، wizard چندمرحله‌ای Onboarding، checklist داشبورد Org Admin، صفحه‌های راهنمای interactive.

**مراحل Onboarding (Checklist گام‌به‌گام):**

| گام | عنوان | اجباری | زمان تخمینی |
| --- | --- | --- | --- |
| 1 | ثبت‌نام سازمان (نام، ارز پایه، timezone، locale) | ✅ | ۲ دقیقه |
| 2 | تأیید ایمیل مدیر اصلی | ✅ | ۱ دقیقه |
| 3 | انتخاب پلن و پرداخت اولیه (IPG یا کدتخفیف) | ✅ | ۳ دقیقه |
| 4 | تکمیل پروفایل سازمان (لوگو، آدرس، شناسه مالیاتی) | ✅ | ۵ دقیقه |
| 5 | ایجاد اولین شعبه | ✅ | ۲ دقیقه |
| 6 | دعوت اولین کاربران (ایمیل، نقش) | ⬜ | ۳ دقیقه |
| 7 | تنظیم نرخ ارز پایه | ⬜ | ۱ دقیقه |
| 8 | بارگذاری نرخ‌نامه یا وارد کردن نرخ اول | ⬜ | ۵ دقیقه |
| 9 | ایجاد اولین پرونده حمل (guided) | ⬜ | ۱۰ دقیقه |
| 10 | تنظیم White-label و custom domain (اختیاری) | ⬜ | ۵ دقیقه |

**ویژگی‌های Onboarding:**

- Progress bar در داشبورد Org Admin (نمایش ٪ تکمیل)
- هر گام: توضیح کوتاه + لینک مستقیم به صفحه مربوطه + tooltip
- گام ۹ (پرونده اول): wizard راهنما با highlight روی فرم‌ها
- ایمیل خوش‌آمد با لینک checklist + ویدیو معرفی (embed YouTube)
- پس از تکمیل ۵ گام اول: ایمیل تبریک + پیشنهاد دموی زنده با پشتیبان
- ایمیل یادآوری: اگر گام ناتمام > ۲۴ ساعت → ایمیل تشویقی
- جدول `onboarding_checklists` برای ذخیره وضعیت هر گام per tenant

**Edge Caseها:**

1. ثبت‌نام با ایمیل تکراری → «این ایمیل قبلاً ثبت شده — ورود یا بازیابی رمز»
2. ایمیل تأیید منقضی (۲۴ ساعت) → درخواست ارسال مجدد
3. پرداخت پلن ناموفق → tenant در حالت trial ۱۴ روزه + یادآوری هر ۳ روز
4. Onboarding بدون تکمیل گام ۵ (شعبه) → ایجاد پرونده block با راهنمای ایجاد شعبه
5. کاربر checklist را skip کند → در هر زمان از داشبورد قابل بازگشت
6. downgrade plan در دوره trial → اعلان و تأیید

**Acceptance:**

- Given tenant جدید بعد از ثبت‌نام، When گام ۱-۵ تکمیل، Then ایمیل تبریک + امکان ایجاد پرونده باز می‌شود.
- Given گام ۹ (ایجاد پرونده guided)، When اپراتور فرم را تکمیل، Then پرونده draft ایجاد، گام ۹ در checklist done، progress bar به ۹۰٪ رسد.

---

# بخش ۶: طراحی UI/UX

## سبک بصری

لوکس، مینیمال، مدرن — الهام از داشبوردهای بانکی/فینتک درجه‌یک. کارت‌های تخت + سایه ملایم، بدون گرادیان‌های تند.

## توکن‌های دیزاین

- **رنگ (روشن):** پس‌زمینه `#F7F8FA`؛ کارت `#FFFFFF`؛ متن `#0F172A`؛ ثانویه `#64748B`؛ Primary `#1E3A5F` hover `#16304F`؛ Accent طلایی `#C9A227` (≤۵٪)؛ Success `#0F9D58`؛ Warning `#D97706`؛ Danger `#DC2626`؛ Border `#E2E8F0`
- **رنگ (تیره):** پس‌زمینه `#0B1220`؛ سطح `#111A2C`؛ متن `#E6EAF2`؛ ثانویه `#8FA0B8`؛ Primary `#5B8DEF`
- CSS variables با `data-theme`؛ ذخیره localStorage + `prefers-color-scheme`
- **تایپوگرافی:** فارسی: Vazirmatn (400/500/700)؛ لاتین: Inter؛ مقیاس: 12/14/16/20/24/32px
- **White-label override:** رنگ‌های primary/secondary از تنظیمات tenant در CSS variables inject می‌شوند

## الزامات

- **RTL-first:** CSS logical properties؛ i18n با next-intl (fa پیش‌فرض + en)
- **PWA/موبایل‌فرست:** breakpoint 640/768/1024/1280؛ جدول‌ها در موبایل → کارت‌لیست؛ Driver PWA full-screen
- **Offline:** فرم‌های تحویل در Driver PWA باید با IndexedDB offline کار کنند
- **دسترس‌پذیری:** کنتراست AA، فوکوس مرئی، aria labels
- **انیمیشن:** transition 150-200ms؛ fade+slide 4px؛ skeleton loader؛ بدون parallax/bounce/confetti
- **Frontend/PWA Lead** مسئول معماری PWA، Service Worker، و تجربه کاربری موبایل‌فرست است

---

# بخش ۷: نقشه راه فاز به فاز

## فاز MVP

زیرساخت (auth+MFA، multi-tenancy+RLS، RBAC، audit log) → Parties → Shipment Core (دریایی FCL/LCL + جاده‌ای) → Documents (HBL/MBL، Shipping Order، QR Code) → Finance (هزینه، فاکتور، پرداخت) → Quotation ساده → **Customer Portal PWA** فقط‌خواندنی + رهگیری عمومی → اعلان SMS/Email/WebPush → **Onboarding wizard**.

**DoD:** همه Acceptance سبز؛ تست ≥۸۰٪؛ E2E «استعلام→پرونده→HBL→فاکتور→پرداخت→بستن»؛ نشت multi-tenancy صفر؛ deploy تک‌فرمانی؛ PWA Lighthouse ≥۹۰.

## فاز ۲

مدهای هوایی/ریلی/ترکیبی کامل → Tracking زنده (GPS دستگاه + **Browser Geolocation**) → **Courier Last-mile + Driver PWA** → Agent Portal PWA → **Agency Line Module** → **Feeder Module** → **NVOCC Module** → CRM + باشگاه → BI + Drill-down + Scenario Simulation → **IPG Multi-Bank + PCI DSS** → Feature Gating کامل → صف مالیاتی → **Customs MVP (mock)** → **Road Permit MVP** → **Insurance MVP**.

**DoD:** ماتریس دسترسی ۱۳ نقش با تست خودکار؛ P95 عملکردی در load test k6 (200 VU)؛ PCI DSS checklist.

## فاز ۳

AI (پیش‌بینی تاخیر ETA، OCR اسناد، پیشنهاد قیمت) → اتوماسیون (rule engine) → IoT سنسور کانتینر reefer (MQTT) → **E-Commerce Portal** → **Distributor Portal + فروش اقساطی** → **بهینه‌سازی PWA و Offline کامل** (Service Worker پیشرفته، background sync، پیشنهاد نصب هوشمند) → **White-label custom domain + SSL خودکار** → امضای دیجیتال PKI واقعی → EDI/API عمومی → **یکپارچگی گمرک واقعی (GMPCS/EPL)** → **پست ایران واقعی** → **راهداری واقعی**.

**DoD:** MAE پیش‌بینی تاخیر <۱۲ ساعت؛ OCR دقت ≥۹۰٪؛ OpenAPI کامل؛ PWA offline score ۱۰۰٪ در Lighthouse.

---

# بخش ۸: الزامات کیفیت و تست

## ۸.۱ روش‌شناسی و پوشش تست

- **Spec-Driven + TDD:** فایل spec در `/specs` → تست → پیاده‌سازی
- **Unit (Vitest/Jest) + Integration (Testcontainers با Postgres واقعی) + E2E (Playwright)**
- پوشش: backend ≥۸۰٪؛ ماژول مالی + RBAC/RLS **۱۰۰٪ branch**؛ frontend ≥۷۰٪
- **تست‌های PWA:** Lighthouse CI در هر PR؛ تست offline با `navigator.onLine` mock؛ تست Service Worker با Workbox Test Utils

## ۸.۲ 🆕 Backup & Disaster Recovery Runbook

### اهداف

- **RPO:** ≤ ۱۵ دقیقه (حداکثر از دست‌رفتن داده)
- **RTO:** ≤ ۱ ساعت (زمان بازیابی)

### استراتژی Backup

| نوع | فرکانس | روش | نگهداری | مقصد |
| --- | --- | --- | --- | --- |
| Full DB | روزانه (۰۲:۰۰) | `pg_dump`  • فشرده‌سازی gzip | ۳۰ روز | S3 bucket جداگانه (cross-region) |
| WAL Continuous | پیوسته | PostgreSQL WAL Archiving | ۷ روز | S3 |
| Incremental DB | هر ۱۵ دقیقه | WAL streaming (Barman یا pgBackRest) | ۲۴ ساعت | S3 |
| Redis | هر ۱ ساعت | RDB snapshot | ۲۴ ساعت | S3 |
| Object Storage (S3) | sync پیوسته | Replication cross-region | همیشه | Secondary region |
| Encryption Keys | روزانه | KMS backup | ۱ سال | Secure vault |

### Runbook بازیابی (DR)

```
مرحله ۱: شناسایی رویداد
  - هشدار Grafana/Sentry → PagerDuty → اعلان On-call
  - تأیید نوع رویداد: DB corruption / infra failure / data loss
  - برآورد زمان شروع مشکل → انتخاب نقطه بازیابی

مرحله ۲: اعلام وضعیت
  - Status page به‌روز: «سیستم در حال بازیابی»
  - اطلاع‌رسانی به tenantها از طریق ایمیل
  - تشکیل War Room (Slack channel اضطراری)

مرحله ۳: بازیابی DB
  - اگر RPO < 15min: بازیابی از WAL streaming (pgBackRest restore)
  - اگر RPO > 15min: بازیابی از آخرین Full Backup + اعمال WAL
  - دستورات:
    pgbackrest --stanza=main restore --target-time="YYYY-MM-DD HH:MM:SS"
    pg_ctl start
  - تأیید integrity: SELECT COUNT(*) از جداول کلیدی

مرحله ۴: بازیابی سرویس‌ها
  - Redis: بازیابی از RDB snapshot
  - API: restart Kubernetes pods
  - S3: اگر region اصلی down → switch به secondary region در ENV

مرحله ۵: تأیید بازیابی
  - اجرای smoke test suite (E2E حداقلی)
  - تأیید RLS: تست نشت cross-tenant
  - بررسی audit_logs آخرین رکورد vs نقطه بازیابی
  - تأیید IPG: تراکنش‌های pending → reconcile با PSP

مرحله ۶: بازگشت به حالت عادی
  - Status page: «سیستم عادی شد»
  - ایمیل گزارش RCA به tenantهای affected
  - ثبت incident در DECISIONS.md
```

### تست DR

- **هر ۳ ماه:** Drill بازیابی در محیط staging (بدون اطلاع تیم پشتیبانی)
- **خودکار (هفتگی):** تست بازیابی Full Backup در staging + تأیید data integrity
- نتایج در `PROGRESS.md` ثبت می‌شوند

## ۸.۳ امنیت

- Argon2id برای پسورد
- JWT access ۱۵ دقیقه + refresh rotation
- MFA TOTP اجباری Admin/Accountant؛ اختیاری برای سایر نقش‌ها
- رمزنگاری at-rest AES-256-GCM برای فیلدهای حساس (config درگاه پرداخت، مهر امضا)
- TLS 1.3 اجباری
- Rate limiting: login 5/min؛ IPG 5 تلاش/دقیقه/IP
- HTTP headers امنیتی (helmet)
- IDOR protection با tenant-scope تست‌شده
- Audit log append-only
- **PCI DSS:** tokenization توسط PSP؛ هیچ داده کارت در FreightCore ذخیره نشود
- OWASP ASVS L2
- HTTPS اجباری برای Geolocation API (PWA)

## ۸.۴ عملکرد

- P95 API خواندنی <300ms؛ نوشتنی <500ms؛ داشبورد <2s
- PDF <5s؛ GPS ingest 1000 msg/s
- KPIها از materialized view (refresh هر ۱۵ دقیقه)
- Index روی همه FKها و ستون‌های فیلتر
- Keyset pagination برای جداول بزرگ
- PWA: FCP <1.5s روی 4G

---

# بخش ۹: دستورالعمل اجرایی برای AI Coding Agent

## ترتیب اجرا

1. **Monorepo** (pnpm + Turborepo): `apps/web` (Next.js + PWA)، `apps/api` (NestJS)، `apps/courier-pwa` (Driver PWA — صفحه وب واکنش‌گرا جدا)، `packages/ui`، `packages/shared`، `packages/config`
2. **زیرساخت:** Docker Compose (postgres18, redis, minio, mailhog, mqtt-broker) → Prisma schema → auth/JWT/MFA → tenancy+RLS → RBAC (۱۳ نقش) → audit → i18n/RTL → PWA (Workbox + manifest.json dynamic)
3. **ماژول‌ها دقیقاً به ترتیب فاز MVP بخش ۷**؛ هر ماژول: spec → migration → service+tests → API → UI → E2E
4. پس از هر ماژول: `pnpm test && pnpm lint && pnpm build`؛ قرمز = توقف

## قالب هر Task

```markdown
### T-<شماره>: <عنوان>
شرح: <یک پاراگراف>
وابستگی‌ها: [T-x, T-y]
معیار پذیرش: <Given/When/Then>
تست‌های لازم: <unit/integration/e2e>
وضعیت: todo | in-progress | done (+ تاریخ)
```

## ساختار تیم فنی (نقش‌های Lead)

- **Backend Lead** — NestJS، Prisma، PostgreSQL، BullMQ
- **Frontend/PWA Lead** — Next.js، Workbox، PWA، RTL، White-label
- **DevOps Lead** — Docker، Kubernetes، CI/CD، Backup/DR
- **QA Lead** — Playwright، k6، Testcontainers
- **Data & AI Manager** — BI، ML، analytics (فاز ۳)

## داده Seed

- ۲ tenant (یکی با White-label custom domain)، ۳ شعبه، ۲۰ مشتری، ۲ Distributor، ۵۰ پرونده در وضعیت‌های مختلف، نرخ‌نامه، فاکتور/پرداخت، ۱ voyage فیدری، ۵ کانتینر NVOCC، ۱۰ سفارش کوریری
- کاربران دمو برای هر ۱۳ نقش با اعتبارنامه مشخص

## قانون ابهام

**سوال نپرس**؛ منطقی‌ترین پیش‌فرض سازگار با این سند را انتخاب و در `DECISIONS.md` ثبت کن.

---

# بخش ۱۰: 🆕 ماژول‌های Competitive Edge (فراتر از رقبا)

> این ماژول‌ها در کاتالوگ سبا سیستم (رقیب اصلی) وجود ندارند. پیاده‌سازی آن‌ها **تمایز رقابتی اصلی** FreightCore در بازار ایران است.
> 

## ۱۰.۱ سامانه جامع گمرکی و اظهارنامه EPL (Customs & EPL Integration)

**پس‌زمینه:** تمام واردات و صادرات ایران نیازمند اظهارنامه رسمی در سامانه‌های گمرک جمهوری اسلامی (GMPCS/EPL — سامانه جامع امور گمرکی / سامانه پنجره واحد بازرگانی) است. رقبا این یکپارچگی را ندارند.

**صفحات:** فهرست اظهارنامه‌ها (لینک به پرونده)، فرم اظهارنامه صادراتی/وارداتی، آپلود اسناد گمرکی، پیگیری وضعیت EPL، محاسبه حقوق گمرکی، گزارش ترخیص.

**فرم اظهارنامه گمرکی:**

- declaration_type (import/export/transit، الزامی)
- shipment_id (ربط به پرونده، الزامی)
- customs_office (گمرک مربوطه، select از لیست، الزامی)
- declarant_party_id (گمرکچی/ترخیصکار، الزامی)
- کالاها (آرایه): hs_code (الزامی، ۸-۱۰ رقم)، description (الزامی)، qty+unit (الزامی)، unit_value (الزامی)، currency، country_of_origin (الزامی)
- goods_value (NUMERIC، محاسبه خودکار از مجموع کالاها)
- duty_amount (محاسبه خودکار بر اساس نرخ‌های تعرفه HS code)
- tax_amount (VAT + سایر مالیات‌ها)
- total_payable (حقوق گمرکی کل)
- اسناد ضمیمه: invoice، packing list، BL/AWB، گواهی مبدا (آپلود چندفایل)

**Workflow:**

draft (تکمیل فرم) → validate (بررسی HS code و مقادیر) → submit_epl (ارسال به سامانه EPL از طریق interface `CustomsProvider`) → pending (در انتظار گمرک) → approved (دریافت شماره اظهارنامه) / rejected (دلیل رد + امکان اصلاح و ارسال مجدد) → paid (پرداخت حقوق گمرکی از IPG) → cleared (ترخیص نهایی).

**محاسبه خودکار حقوق:**

- جدول نرخ تعرفه HS code (بارگذاری سالانه از گمرک، ذخیره داخلی)
- فرمول: `duty = goods_value × duty_rate_pct + tax_amount`
- نمایش محاسبه گام‌به‌گام برای شفافیت

**Edge Caseها:**

1. HS code نامعتبر (نه در جدول تعرفه) → خطا با پیشنهاد HS مشابه
2. EPL timeout → retry خودکار با exponential backoff؛ پس از ۳ بار، وضعیت «ارسال ناموفق» + اعلان به اپراتور
3. اظهارنامه رد شده → نمایش دلیل دقیق EPL + دکمه «اصلاح و ارسال مجدد»
4. تغییر ارزش کالا بعد از submit → ممنوع؛ باید void و اظهارنامه جدید
5. پرونده Import/Export بدون اظهارنامه → هشدار هنگام تلاش برای Close
6. تعرفه HS code تغییر کند → اظهارنامه‌های draft با نرخ جدید محاسبه می‌شوند؛ submitted تغییر نمی‌کنند

**Acceptance:**

- Given پرونده صادراتی با ۳ قلم کالا، When اظهارنامه submit_epl، Then EPL submission_id دریافت، وضعیت pending، shipment.customs_declaration_no به‌روز، اعلان به اپراتور.
- Given EPL approved، When گمرک شماره اظهارنامه می‌دهد، Then declaration_no ذخیره، milestone CUSTOMS_IN ثبت در shipment، دکمه پرداخت حقوق فعال.
- Given حقوق گمرکی پرداخت (از IPG)، Then milestone CUSTOMS_OUT ثبت، پرونده آماده ترخیص.

---

## ۱۰.۲ سامانه راهداری و مجوز حمل (Road Transport Permit)

**پس‌زمینه:** شرکت‌های حمل‌ونقل جاده‌ای ایران برای هر سفر بین‌شهری/بین‌المللی نیاز به پروانه حمل از سازمان راهداری و ناجا دارند. این یکپارچگی رقبا ندارند.

**صفحات:** فهرست پروانه‌های حمل، فرم درخواست پروانه، پیگیری وضعیت، انقضای پروانه‌ها، گزارش انطباق (compliance).

**فرم درخواست پروانه:**

- shipment_id (الزامی)
- vehicle_id (الزامی — ماشین باید در سیستم ثبت باشد)
- permit_type (domestic/international/oversize/hazmat، الزامی)
- valid_from / valid_to (الزامی)
- issued_by_authority (راهداری/ناجا، الزامی)
- rahdarico_ref (شماره مرجع راهداری، اختیاری در دستی؛ خودکار در API)
- naaja_ref (شماره مرجع ناجا، اختیاری)
- file_id (اسکن پروانه صادره، الزامی پس از صدور)

**Workflow:**

pending (تقاضا) → submitted (ارسال به سامانه راهداری از طریق `RoadTransportProvider`) → issued (دریافت پروانه + آپلود اسکن) → expired / revoked.

**هشدارهای خودکار:**

- ۷ روز قبل از انقضا → اعلان به اپراتور
- پرونده جاده‌ای بدون پروانه معتبر → هشدار هنگام Booking
- وسیله نقلیه با پروانه منقضی → block در تخصیص سفر

**Edge Caseها:**

1. سفر بین‌المللی نیاز به پروانه TIR/CMR → validation خودکار بر اساس مقصد
2. پروانه oversize نیاز به مجوز پلیس راه → فیلد naaja_ref الزامی
3. پروانه hazmat نیاز به گواهی حمل مواد خطرناک → آپلود مدرک اجباری
4. API راهداری down → fallback به ثبت دستی + pending تأیید
5. انقضا در حین سفر → اعلان فوری + milestone PERMIT_EXPIRED

**Acceptance:**

- Given سفر جاده‌ای بین‌المللی، When Booking زده می‌شود و پروانه حمل معتبر وجود ندارد، Then هشدار «پروانه حمل الزامی است» + لینک ایجاد درخواست.
- Given پروانه ۵ روز تا انقضا، When scheduler شبانه اجرا، Then اعلان ایمیل + In-app به اپراتور با لینک تمدید.

---

## ۱۰.۳ بیمه باربری و محموله (Cargo Insurance Module)

**پس‌زمینه:** بیمه باربری در حمل‌ونقل بین‌المللی ایران الزامی بوده و ریسک‌های مالی عمده‌ای را پوشش می‌دهد. هیچ رقیبی این ماژول را ندارد.

**صفحات:** فهرست بیمه‌نامه‌ها (لینک به پرونده)، فرم درخواست بیمه، جزئیات بیمه‌نامه، ثبت خسارت (claim)، پیگیری claim، گزارش ریسک.

**فرم بیمه‌نامه:**

- shipment_id (الزامی)
- insurer_party_id (شرکت بیمه، autocomplete، الزامی)
- coverage_type (all_risk/named_perils/total_loss، الزامی)
- insured_value (NUMERIC، ارزش بیمه‌شده، الزامی — پیش‌فرض: goods_value + freight cost)
- currency (الزامی)
- premium (حق بیمه — محاسبه خودکار: insured_value × rate یا وارد دستی)
- premium_currency
- valid_from / valid_to (معمولاً از ETD تا delivery + ۳۰ روز)
- policy_no (شماره بیمه‌نامه — دستی پس از صدور یا از API)
- notes (شرایط خاص)

**فرم خسارت (Claim):**

- policy_id (الزامی)
- incident_date (الزامی)
- incident_description (الزامی، توضیح دقیق حادثه)
- claimed_amount (NUMERIC، الزامی)
- اسناد ضمیمه: گزارش خسارت، عکس، BL، invoice

**Workflow بیمه:** quoted (استعلام) → active (پس از پرداخت حق بیمه) → expired.

**Workflow Claim:** open (ثبت خسارت) → submitted (ارسال به بیمه‌گر از طریق `CargoInsuranceProvider`) → approved (مبلغ پرداختی مشخص) / rejected → paid.

**Edge Caseها:**

1. insured_value کمتر از goods_value → هشدار «ارزش بیمه ناکافی (Under-insurance)»
2. Claim بعد از انقضای بیمه‌نامه → ممنوع
3. Claim مبلغ بیش از insured_value → block (approved_amount ≤ insured_value)
4. پرونده hazmat: coverage_type باید all_risk باشد → validation اجباری
5. چند Claim در یک بیمه‌نامه → مجموع approved نباید از insured_value تجاوز کند
6. shipment بدون بیمه با insurance_required=true → هشدار مانع Booking

**Acceptance:**

- Given پرونده با insurance_required=true و بدون بیمه‌نامه فعال، When Booking، Then هشدار «بیمه باربری الزامی» + لینک ایجاد بیمه‌نامه.
- Given Claim ثبت‌شده، When بیمه‌گر approved_amount=۸۰٪، Then claim.status=approved، اعلان به مشتری، دکمه «ثبت دریافت خسارت» در Finance.

---

## ۱۰.۴ یکپارچگی پست ایران و شرکت‌های پستی (Postal Integration)

**پس‌زمینه:** بسیاری از کوریرها و فورواردرها از خدمات پست ایران، چاپار، تیپاکس و سایر ارائه‌دهندگان پستی استفاده می‌کنند. یکپارچگی API tracking و ارسال، FreightCore را از رقبا متمایز می‌کند.

**صفحات:** فهرست مرسولات پستی، فرم ارسال، پیگیری کد رهگیری، گزارش تحویل، import دسته‌ای.

**فرم مرسوله پستی:**

- shipment_id / courier_order_id (یکی الزامی — ربط به پرونده یا سفارش کوریری)
- provider (post_iran/chapar/tipax/...، الزامی)
- service_type (express/standard/registered/cod، الزامی)
- sender_info (JSONB: نام، موبایل، آدرس کامل، کد پستی)
- recipient_info (JSONB: نام، موبایل، آدرس کامل، کد پستی الزامی)
- declared_weight_kg (NUMERIC، الزامی)
- declared_value (NUMERIC، اختیاری برای COD)
- tracking_code (دستی یا خودکار از API)

**Workflow:**

registered (ثبت) → posted (تحویل به پست) → in_transit (در راه — tracking API) → delivered / returned / failed.

**Auto-Tracking:** job هر ۲ ساعت → `PostalProvider.getTrackingStatus(tracking_code)` → به‌روزرسانی `last_tracking_event` → اگر status تغییر کرد → milestone در shipment + اعلان.

**Import دسته‌ای:** آپلود Excel با ستون‌های tracking_code, provider, recipient_phone → ایجاد دسته‌ای postal_shipments + شروع auto-tracking.

**Edge Caseها:**

1. کد رهگیری نامعتبر در API پست → وضعیت «کد نامعتبر» + امکان اصلاح دستی
2. API پست ایران down → نمایش آخرین وضعیت ذخیره‌شده + برچسب زمان آخرین بررسی
3. مرسوله مفقوده (API: delivered ولی گیرنده دریافت نکرده) → دکمه «اعتراض» → ثبت claim پستی
4. COD: مبلغ باید به سیستم Finance تطبیق داده شود پس از تحویل
5. provider تغییر کند بعد از ثبت → فقط void و ثبت مجدد با provider جدید
6. Import دسته‌ای با tracking_code تکراری → skip + گزارش

**Acceptance:**

- Given مرسوله ثبت با tracking_code پست ایران، When job هر ۲ ساعت اجرا و status=delivered دریافت، Then postal_shipment.status=delivered، milestone DELIVERED در shipment/courier_order، اعلان SMS به فرستنده و گیرنده.
- Given import Excel 100 مرسوله، When آپلود، Then 100 رکورد ایجاد (جز تکراری‌ها با skip report)، auto-tracking برای همه فعال.

---

# پیوست: تغییرات نسبت به نسخه قبل (v1 → v2)

## تغییرات اساسی

### ۱. حذف کامل اپلیکیشن نیتیو موبایل

- ❌ حذف: React Native / Expo SDK از Tech Stack
- ❌ حذف: انتشار در Google Play / Apple App Store
- ❌ حذف: نقش «Mobile App Lead»
- ✅ جایگزین: PWA با Workbox 7 + next-pwa
- ✅ جایگزین: نقش «Frontend/PWA Lead»
- ✅ جایگزین: Browser Geolocation API برای راننده
- ✅ جایگزین: Canvas Signature Pad وب برای POD

### ۲. تکمیل Tech Stack

- ✅ اضافه: Workbox 7 + next-pwa (Service Worker، Offline-first)
- ✅ اضافه: Web Push API با VAPID
- ✅ اضافه: MQTT 5.0 broker برای IoT
- ✅ اضافه: ۱۰ interface برای سرویس‌های بیرونی جدید

## ماژول‌های جدید اضافه‌شده

| # | ماژول | بخش | وضعیت |
| --- | --- | --- | --- |
| 1 | Agency Line (نمایندگی کشتیرانی) | ۵.۱۱ | 🆕 کامل |
| 2 | Feeder Operations (فیدری) | ۵.۱۲ | 🆕 کامل |
| 3 | NVOCC & Container Management | ۵.۱۳ | 🆕 کامل |
| 4 | White-label PWA Portals | ۵.۱۴ | 🆕 کامل |
| 5 | IPG Multi-Bank + PCI DSS | ۵.۱۵ | 🆕 کامل |
| 6 | E-Commerce & Online Sales | ۵.۱۶ | 🆕 کامل |
| 7 | Distributor Portal + Installment | ۵.۱۷ | 🆕 کامل |
| 8 | Customer Onboarding Flow | ۵.۱۸ | 🆕 کامل |
| 9 | Backup & DR Runbook | ۸.۲ | 🆕 کامل |
| 10 | External Integrations (فهرست فنی) | ۳.۵ | 🆕 کامل |
| 11 | Customs & EPL Integration | ۱۰.۱ | 🆕 Competitive Edge |
| 12 | Road Transport Permit | ۱۰.۲ | 🆕 Competitive Edge |
| 13 | Cargo Insurance | ۱۰.۳ | 🆕 Competitive Edge |
| 14 | Postal Integration | ۱۰.۴ | 🆕 Competitive Edge |

## نقش‌های جدید اضافه‌شده به ماتریس دسترسی

| # | نقش | بخش |
| --- | --- | --- |
| 10 | Shipping Line Agent (نماینده کشتیرانی) | ۴ + ۵.۱۱ |
| 11 | Distributor (نماینده فروش) | ۴ + ۵.۱۷ |
| 12 | Contractor (پیمانکار) | ۴ |
| 13 | External Carrier (خط کشتیرانی خارجی) | ۴ + ۵.۱۱ |

## تکمیل‌های ماژول‌های موجود

- **۵.۱ Shipment Core:** اضافه customs_declaration_no، insurance_policy_no، road_permit_no، shipping_line_party_id
- **۵.۳ Documents:** اضافه QR Code در PDF نهایی
- **۵.۵ Tracking:** اضافه Browser Geolocation API، IoT MQTT، آلارم دمای reefer
- **۵.۶ Courier:** Driver PWA → وب‌پایه با Canvas POD و Browser GPS
- **۵.۹ BI:** اضافه Drill-down و Scenario Simulation
- **۵.۱۰ Org Settings:** اضافه White-label تنظیمات (لوگو، رنگ، custom domain، PWA manifest)

## جداول DB جدید

`shipping_line_contracts`، `agency_bills_of_lading`، `agency_performance`، `feeder_voyages`، `feeder_bookings`، `feeder_costs`، `nvocc_containers`، `nvocc_allocations`، `nvocc_demurrage`، `payment_gateways`، `payment_transactions`، `digital_wallets`، `wallet_transactions`، `qr_payment_requests`، `product_categories`، `products`، `price_lists`، `price_list_items`، `discount_campaigns`، `orders`، `order_items`، `distributors`، `installment_plans`، `installment_payments`، `customs_declarations`، `customs_declaration_items`، `customs_documents`، `road_permits`، `insurance_policies`، `insurance_claims`، `postal_shipments`، `onboarding_checklists`، `web_push_subscriptions`

---

*FreightCore [SPEC.md](http://SPEC.md) v2.0 — تولیدشده با Gap Analysis کاتالوگ سبا سیستم*

*آماده Import مستقیم در Cursor / Claude Code / Windsurf*