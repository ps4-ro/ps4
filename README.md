# CSSFontFace GoldHEN Host — إصدار الاستقرار (Stable One-Click Build) r4

هوست ويبكيت PS4 لتفعيل **GoldHEN** عبر المتصفح — يعمل على تحديثات النظام من **6.00 إلى 11.02**.

A PS4 WebKit exploit host that activates **GoldHEN** through the browser, supporting firmware **6.00 – 11.02**.

---

## التغييرات في هذا الإصدار / What's new in this build

| الميزة / Feature | قبل / Before | بعد / After |
|---|---|---|
| جلب البايلود (r4) | fetch بعد الجيلبرك (قد يفشل فيتوقف قبل النهاية) | **Warm cache**: تُجلب التصحيحات + البايلود أثناء العدّاد قبل أي سباق — صفر شبكة بعد الجيلبرك + fallback بثلاث محاولات |
| التعليق / التجمد (r4) | انتظار forever إذا علق RPC | **Stall watchdog**: لا تقدم خلال 45 ثانية → إعادة تحميل تلقائية |
| تحميل السكربتات (r4) | فشل تحميل واحد يوقف الجولة | **إعادة محاولة** تلقائية قبل الفشل |
| الثيم (r4) | داكن أزرق/أخضر قديم | **ثيم PS4** (كحلي داكن + ذهبي GoldHEN) بنفس معرّفات الواجهة |
| تفعيل GoldHEN | زر + عدّاد 5 ثوانٍ + اختيار يدوي للسلسلة | **ضغطة واحدة** — زر واحد كبير + عدّاد 3 ثوانٍ تلقائي قابل للإلغاء |
| اختيار سلسلة النواة | يدوي دائمًا (localStorage) قد يختار خطأً | **تلقائي** حسب الإصدار: Lapse دائمًا (شامل 6.00–11.02)، NetCtrl يدوي فقط على 9.00+ (يتطلب ثوابت KL_LOCK غير موجودة تحت 9.00) |
| الفشل أثناء التشغيل | رسالة خطأ فقط، الزر معطّل، يجب فتح الصفحة يدويًا | **إعادة محاولة تلقائية** عبر إعادة تحميل الصفحة حتى 3 مرات (sessionStorage)، ثم زر Retry |
| السجل التفصيلي (debug) | مفعّل دائمًا — يحمّل DOM ويشوّش توقيت السباقات | **معطّل افتراضيًا** — يُفعّل فقط عبر `?debug=1` |
| مراحل التنفيذ | سطر واحد في الكونسول | شارات مراحل (WebKit → Kernel → GoldHEN) بحالات ملونة + رسالة حالة |
| التحقق | لا يوجد | تحقق بعد كل مرحلة: ARW primitives، kernel_base، setuid(0) بعد الجيلبرك، صحة payload بعد mmap |
| التنظيف (cleanup) | خطأ تنظيف واحد يوقف الجيلبرك | cleanup محمي — أخطاؤه لا تمنع إكمال العملية |
| العمال (workers) | موت العامل = تعليق forever | معالجة onerror ترفض الوعود المعلقة + فحص ping قبل السباق |
| ذاكرة التخزين المؤقت | AppCache قديم يبقى عالقًا بعد التحديث | bootstrap يقارن HOST_VERSION ويحدّث/يعيد التحميل مرة واحدة |
| رسائل الخطأ | عامة | مصنّفة حسب المرحلة (webkit/kernel/payload) مع رقم المحاولة |

---

## النشر / Deployment

المجلد هو ملفات ثابتة — انسخه إلى أي خادم HTTP:
The folder is static files — drop it on any HTTP server:

```bash
# الخيار الأسهل — بايثون
cd ps4_host
python3 -m http.server 8080
```

ثم على PS4 افتح المتصفح: `http://IP-الجهاز:8080/` — ثم اضغط الزر (أو انتظر العدّاد).

Then on the PS4 browser open `http://<host-ip>:8080/` and press the button (or wait for the countdown).

> ملاحظة: في أول زيارة بعد التحديث، قد يظهر الإصدار القديم بسبب AppCache — أعد فتح الصفحة مرة ثانية، أو امسح كاش المتصفح مرة واحدة.
> Note: on the first visit after an update the old cached page may appear due to AppCache — reopen once, or clear the browser cache once.

---

## خريطة الإصدارات / Firmware map

| النطاق / Range | السلسلة التلقائية / Auto chain | ملاحظات / Notes |
|---|---|---|
| 6.00 – 8.52 | **Lapse** (AIO double-free) | NetCtrl غير متاح: ثابت `KL_LOCK` غير معرّف تحت 9.00 — الراديو معطّل |
| 9.00 – 11.02 | **Lapse** (افتراضي — الأكثر اختبارًا) | يمكن اختيار **NetCtrl** يدويًا (ucred triple-free) بإلغاء "Auto chain" |

- ثغرة الويبكيت CSSFontFace نفسها: PS4 **6.00 – 11.02** (مؤكدة من توثيق مستودع CSSFontFace-Exploit).
- ملاحظة: إصدارات 11.50+ وغيرها خارج النطاق بسبب تغيّر بنية `m_featureSettings` في WebKit الأحدث.
- PS5 غير مدعوم (console=5 → TODO في الكود الأصلي).

---

## آلية إعادة المحاولة / Retry policy

1. كل فشل يصنّف حسب المرحلة: `webkit` / `kernel` / `payload`.
2. تلقائيًا: إعادة تحميل الصفحة حتى **3 محاولات** (تُحتسب عبر sessionStorage — لا تُمسح بإعادة التحميل).
3. **Stall watchdog (r4)**: إذا لم يكن أي تقدم خلال 45 ثانية (سطر كونسول أو تحديث واجهة) تعيد الصفحة تحميل نفسها تلقائيًا بدل التجمد للأبد.
4. بعد 3 محاولات: تظهر رسالة واضحة + زر **Retry**، وإذا استمر الفشل أغلق تطبيق المتصفح (Close Application) وأعد فتحه — هذا يصفّر حالة WebKit ويحسّن فرص النجاح.
5. البايلود والتصحيحات تُحمَّل مسبقًا أثناء العدّاد (Warm cache) — لا يوجد أي fetch بعد الجيلبرك.

---

## هيكل الملفات / File layout

```
index.html            — الواجهة + bootstrap (HOST_VERSION / فحص الكاش)
includes/script.js    — التحكم: عدّاد، سلسلة تلقائية، إعادة محاولة، حالات UI
includes/style.css    — التنسيقات
src/main.js           — منسّق المراحل: userland → kernel → payload
src/misc.js           — logger (verbose=false افتراضيًا)، BInt، كشف الإصدار
src/ps4/userland.js   — UAF/ARW/ROP/سكان syscalls
src/ps4/kernel.js     — KernelView ARW، jailbreak، kernel_patches (kexec)
src/ps4/constants.js  — إزاحات لكل إصدار + KPATCH
src/lapse.js          — سلسلة النواة Lapse (AIO double-free) — 6.00–11.02
src/netctrl.js        — سلسلة النواة NetCtrl (ucred triple-free) — 9.00+ فقط
src/worker.js, src/workers.js — عمال RPC للسباقات
src/loader.js         — تحميل payload.bin (مع تحقق من الكتابة)
src/payload.bin       — محمّل GoldHEN (ضع نسختك هنا إن لزم)
src/ps4/patches/*.bin — تصحيحات النواة لكل إصدار
cache.manifest        — AppCache (حدّث سطر الإصدار بعد أي تعديل)
```

---

## استكشاف الأخطاء / Troubleshooting

| المشكلة / Issue | الحل / Fix |
|---|---|---|
| يتجمد ولا يصل إلى `===END===` | (r4) warm cache + watchdog يعيدان التحميل تلقائيًا؛ إن تكرر أغلق المتصفح وأعد فتحه — واستخدم كابل شبكة إن أمكن |
| فشل متكرر في مرحلة WebKit | أغلق تطبيق المتصفح من PS4 وأعد فتحه (يصفّر حالة WebKit) ثم أعد المحاولة |
| "NetCtrl" معطّل | طبيعي على 6.00–8.52 — استخدم Lapse (التلقائي) |
| الإصدار القديم يظهر بعد التحديث | أعد فتح الصفحة مرة ثانية أو امسح كاش المتصفح |
| GoldHEN لا يُفعّل رغم نجاح الجيلبرك | تأكد من وجود `/data/GoldHEN.bin` (أو النسخة الصحيحة لجهازك) — payload.bin هو المحمّل فقط |
| تريد سجلات مفصلة | أضف `?debug=1` إلى الرابط |

---

## تحذير / Disclaimer

لأغراض البحث والتعليم فقط على أجهزتك الخاصة. استخدمه بمسؤوليتك الخاصة.
For research and educational purposes only, on your own devices. Use at your own risk.

## الشكر / Credits

- الباحثون: ufm42، Nathan Fargo (ntfargo)، nhk، Dr.Yenyen، AlAzif — ومجتمع PS4 homebrew.
- هذا البناء: تحسينات استقرار (واجهة ضغطة واحدة، إعادة محاولة، اختيار تلقائي، حماية تنظيف، فحص مراحل) فوق الهوست الأصلي CSSFontFace.
