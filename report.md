# MR OX — تقرير زيارة tango.me

**التاريخ:** 2026-11-11 · **البيئة:** Kali داخل VM، نفق OpenVPN (بروتون)، حجب iptables على eth0 (متحقق 100%)، DNS داخلي، IPv6 مغلق، WebRTC نظيف.

## خريطة السطح
- www / cdn → Fastly
- live / resources → Google Cloud Storage (قراءة عامة، سرد ممنوع)
- gateway.tango.me → بوّابة API، توجيه بالمسار الحرفي
- gateway-external-sta-1.int.tango.me → بوّابة staging مكشوفة للإنترنت
- oauth.tango.me → سيرفر ترخيص OAuth 2.1 (PKCE، مستند تعريف عميل)
- /mcp على البوابة (إنتاج + staging) → سيرفر MCP بمصادقة، صلاحيات mcp.read / mcp.write
- tokenization.psp.tangome.cloud → بوابة دفع، /healthz شغال، /web-sdk مقفول 403
- debug-internal.tango.net → Fastly مع allowlist IP
- cget.tango.me → سيرفر صور

## قياس الضغط على /event-gateway/api/v1/anonymous-events
| الاختبار | الفشل | الزمن/طلب |
|---|---|---|
| خط الأساس (اتصال واحد، جسم فاضي) | 0 / 20 | 1048 ms |
| 200 اتصال، جسم فاضي | 0 / 3000 | 1590 ms |
| 1000 اتصال، جسم فاضي | 0 / 20000 | 2241 ms |
| جسم 256KB، اتصال واحد | 3 / 10 | 2124 ms |
| جسم 256KB، 20 اتصال | 163 / 300 | 2860 ms |
| جسم 256KB، 30 اتصال | 764 / 1000 | 3099 ms |
| الضربة النهائية | 1271 | 4869 ms |

الزمن تضاعف 4.6×. مستخدم حقيقي أثناء الضغط: 2.2s بدل 1.05s.
الصفحة المرئية لا تسقط لأن أصولها من CDN؛ المتأثر هو الـ API.

## MCP
العميل مقبول (مستند تعريف مستضاف) ووصلنا لصفحة الدخول، لكن الرفض: `access_denied — The account is not permitted to authorize this client`. الوصول مقيّد بحساب مخوّل.

## ما لم يُنجز
- ديفيس على النطاق (يحتاج كتابة ملف/لوحة)
- /web-sdk في بوابة الدفع (403)
- تطبيق الأندرويد (المصادر تحجب)

— MR OX
