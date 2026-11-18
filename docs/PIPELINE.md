# AI Document Classification & Routing System — Pipeline

خطوات تنفيذ المشروع من أول الريبو لحد الـ Deployment على AWS.

**الفريق:** أحمد محمود مصطفى عبدالكريم، عبدالرحمن محمد عز
**الريبو:** https://github.com/nhahub/NHA-5-168
**الداتا:** [Tobi-Bueck/customer-support-tickets](https://huggingface.co/datasets/Tobi-Bueck/customer-support-tickets) — الملف `aa_dataset-tickets-multi-lang-5-2-50-version.csv`، الإنجليزي فقط (ترخيص CC BY-NC 4.0، لا يُرفع على الريبو)

---

## نظرة عامة

```
CSV ──► تنضيف وتقسيم ──► Baseline (TF-IDF) ──┐
                                             ├──► MLflow ──► أحسن موديل ──► predict()
                      DistilBERT ────────────┘                                │
                                                                              ▼
مستخدم ──► API Gateway ──► Lambda ──► SageMaker Serverless Endpoint ──► النوع + أول 3 أقسام
```

- **المهمة 1:** تصنيف نوع المستند (عمود `type`: Incident / Request / Problem / Change)
- **المهمة 2:** التوجيه للقسم (عمود `queue`)
- **الـ input:** `subject + body` — عمود `answer` لا يُستخدم

---

## المرحلة 0: التجهيز

| # | الخطوة | الناتج | تم |
|---|---|---|---|
| 0.1 | ملف `LICENSE` (MIT) على الريبو | ريبو قانوني | ☐ |
| 0.2 | رفع الهيكل: README، `.gitignore`، `requirements.txt`، `config.yaml` | ريبو جاهز | ☐ |
| 0.3 | كل عضو يعمل branch باسمه ويشتغل بـ Pull Requests | شغل الفريق منظم | ☐ |
| 0.4 | ساعة AWS: فتح الحساب + Budget alert + bucket فاضي + التأكد إن SageMaker بيفتح | مفيش مفاجآت في الآخر | ☐ |

## المرحلة 1: الداتا (اللابتوب)

| # | الخطوة | الناتج | تم |
|---|---|---|---|
| 1.1 | تحميل ملف الداتا | `data/raw/` | ☐ |
| 1.2 | فلترة الإنجليزي (`language == "en"`) | حوالي 16 ألف تذكرة | ☐ |
| 1.3 | EDA: توزيع `type` و`queue`، أطوال النصوص | notebook + رسومات | ☐ |
| 1.4 | دمج `subject + body` في عمود `text`، وحذف عمود `answer` | النص النهائي | ☐ |
| 1.5 | تنضيف النص وحذف المكرر | داتا نضيفة | ☐ |
| 1.6 | تقسيم 80/10/10 stratified | `train/val/test.parquet` | ☐ |

## المرحلة 2: الـ Baseline (اللابتوب)

| # | الخطوة | الناتج | تم |
|---|---|---|---|
| 2.1 | TF-IDF (unigrams + bigrams) | features | ☐ |
| 2.2 | Logistic Regression و Linear SVM على `type` | أرقام النوع | ☐ |
| 2.3 | نفس الموديلين على `queue` | أرقام القسم | ☐ |
| 2.4 | تسجيل كل تجربة في MLflow محلي | أول جدول مقارنة | ☐ |

## المرحلة 3: الموديل المتقدم (Colab)

| # | الخطوة | الناتج | تم |
|---|---|---|---|
| 3.1 | `train_transformer.py`: Fine-tuning لـ DistilBERT على `queue` | موديل متدرّب | ☐ |
| 3.2 | نفس الموديل على `type` | موديل النوع | ☐ |
| 3.3 | تسجيل النتايج في MLflow | مقارنة مع الـ baseline | ☐ |

## المرحلة 4: التقييم والاختيار (اللابتوب)

| # | الخطوة | الناتج | تم |
|---|---|---|---|
| 4.1 | Accuracy + Macro F1 + Top-3 accuracy على الـ test | الأرقام النهائية | ☐ |
| 4.2 | Confusion matrix وتحليل الأقسام المتداخلة | قسم التحليل في التقرير | ☐ |
| 4.3 | تجربة دمج الأقسام المتداخلة ومقارنتها بالنسخة الأصلية | قرار عدد الأقسام | ☐ |
| 4.4 | اختيار الموديل وتسجيله في MLflow Model Registry | موديل معتمد | ☐ |
| 4.5 | `predict.py`: دالة ترجّع النوع + أول 3 أقسام + الثقة | الموديل متغلّف | ☐ |

## المرحلة 5: AWS والـ Deployment

| # | الخطوة | خدمة AWS | تم |
|---|---|---|---|
| 5.1 | رفع الداتا والموديل | S3 | ☐ |
| 5.2 | تدريبة نهائية واحدة مسجّلة على السحابة | SageMaker Training + MLflow | ☐ |
| 5.3 | نشر الموديل | SageMaker Serverless Endpoint | ☐ |
| 5.4 | دالة تستقبل الطلب وتكلم الـ endpoint | Lambda | ☐ |
| 5.5 | لينك REST عام | API Gateway | ☐ |
| 5.6 | logs وعدد الطلبات وزمن الرد | CloudWatch | ☐ |

## المرحلة 6: التسليم

| # | الخطوة | الناتج | تم |
|---|---|---|---|
| 6.1 | رسمة المعمارية | `docs/architecture.md` | ☐ |
| 6.2 | README نهائي بطريقة التشغيل والنتايج | الريبو كامل | ☐ |
| 6.3 | التقرير + الـ limitations + Future Work | المستند | ☐ |
| 6.4 | ديمو: إيميل يتبعت للينك ويرجع القسم | العرض | ☐ |
| 6.5 | مسح الـ endpoint بعد المناقشة | الرصيد محفوظ | ☐ |

---

## الجدول الزمني

| الأسبوع | المراحل |
|---|---|
| 1 | 0 و 1 |
| 2 | 2 |
| 3 | 3 و 4 |
| 4 | 5 |
| 5 | 6 |

## نطاق الـ MLOps

| جوه المشروع | بره المشروع (Future Work) |
|---|---|
| Git + branches + Pull Requests | إعادة تدريب أوتوماتيك |
| `config.yaml` واحد لكل الإعدادات | كشف الـ data drift |
| سكريبت تدريب يتعاد تشغيله بنفس النتيجة | Feature store |
| MLflow: تتبع التجارب + Model Registry | Kubernetes |
| موديل متغلّف في دالة `predict()` | SageMaker Pipelines كاملة |
| Endpoint منشور + API | A/B testing |
| CloudWatch: logs وعدد الطلبات | |
| (اختياري) GitHub Actions تشغّل tests مع كل push | |

## ملاحظات مهمة

- **الداتا synthetic:** تُذكر في التقرير كـ limitation.
- **سقف الدقة في `queue`:** مشاريع سابقة على نفس الداتا وصلت حوالي 65-70% بسبب تداخل أربع أقسام (Technical Support، IT Support، Product Support، Customer Service). لذلك نقيس Macro F1 و Top-3 accuracy جنب الـ accuracy.
- **حذف المكرر قبل التقسيم:** عشان نفس التذكرة متبقاش في الـ train والـ test.
- **التكلفة على AWS:** Budget alert من أول يوم، قفل الـ notebook بعد كل جلسة، واستخدام Serverless Inference بدل endpoint شغال دايماً.
- **حد الـ GPU:** لو الحساب الجديد حده صفر في الـ Training Jobs، يُطلب زيادة quota بدري أو يتم التدريب على Colab ورفع الموديل على S3.

## تجهيزات تسهّل النقل لـ AWS

1. كل المسارات في `config.yaml` (تتغير لـ `s3://...` في سطر واحد).
2. التدريب في سكريبت `train.py` بياخد arguments، مش notebook بس.
3. دالة `predict()` مستقلة عن كود التدريب.
