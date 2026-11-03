# ai-worker

Worker على Cloudflare اسمه `ai-api`، يخلي صفحات المساعد الذكي في موقعي ترد لأي زائر مجانًا، عن طريق نموذج مفتوح على Cloudflare Workers AI.

الرابط: https://ai-api.danaalrafadan.workers.dev

## المسارات
| المسار | الصفحة | الحقول | حد الرد |
|---|---|---|---|
| `/assistant` | [مساعد الدوام](https://danaalrafadan-web.github.io/مساعد%20الحضور/) | `question` | 500 توكن |
| `/trip` | [مخطط الرحلات](https://danaalrafadan-web.github.io/مخطط%20الرحلات/trip-planner.html) | `dest` `days` `budget` `likes` `notes` | 1500 توكن |
| `/attendance-bot` | [الحضور الذكي](https://danaalrafadan-web.github.io/smart-attendance/) | `question` `context` | 500 توكن |

## الحماية
- يقبل الطلبات من `https://danaalrafadan-web.github.io` بس، وأي موقع ثاني ياخذ 403
- 10 طلبات في الدقيقة لكل IP (429)، و500 حرف لكل حقل نصي و2000 لسجل الموظف (413)
- يتجاهل أي حقل غير الحقول المكتوبة فوق، وتعليمات الذكاء تبقى هنا مو في الصفحة
- ما يحفظ نص المستخدم وما يكتبه في السجلات
- الذكاء عن طريق الربط `AI` بدون أي مفتاح، وSupabase بالمفتاح العام `sb_publishable_` بس

## وين ألقى AI_INSTRUCTIONS؟
في `src/index.js`، أول شي في الملف، بين هذين السطرين:

```
// ===== تعليمات الذكاء الاصطناعي: عدلها من هنا =====
...
// ===== نهاية التعليمات =====
```

فيها ثلاث تعليمات: `assistant` و`trip` و`attendanceBot`، وتحتها `BOT_KNOWLEDGE` (اللوائح وشرح النظام).

## كيف أعدلها وأنشرها؟
1. عدل النص اللي تبيه، مثلاً `اللهجة: سعودية` إلى `اللهجة: مصرية`
2. احفظ الملف
3. من داخل هذا المجلد: `npm install` أول مرة، وبعدها `npx wrangler deploy`

## النموذج
- الأساسي: `@cf/meta/llama-3.3-70b-instruct-fp8-fast`، لأنه الأفضل بالعربي واللهجة في تجربة 3 أسئلة حقيقية من صفحاتي
- البديل: `@cf/openai/gpt-oss-20b`، لما يتعطل الأول. أرخص بكثير وأسرع
- الرصيد المجاني 10,000 Neurons باليوم، ويتجدد 3:00 الفجر بتوقيت الرياض. لما يخلص، الصفحة تعرض «مثال جاهز»

## المهمة المجدولة (batch job)
كل 6 ساعات (`0 */6 * * *`) يشغل Cloudflare الدالة `scheduled` بنفسه، وهي تقرأ صف واحد من جدول `prompt_templates` في Supabase ثلاث مرات. كذا قاعدة البيانات تشوف نشاط حقيقي، وما يوقف المشروع المجاني بسبب قلة الاستخدام.

تجربتها محليًا: `npx wrangler dev --test-scheduled` وبعدها افتح `http://localhost:8787/__scheduled`
