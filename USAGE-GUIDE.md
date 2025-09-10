# دليل استخدام نظام البحث في الكتب العربية

## ✅ الإعداد مكتمل بنجاح!

تم إعداد نظام ELK Stack للبحث في النصوص العربية وهو جاهز للاستخدام.

## 🏗️ حالة النظام الحالية

### خدمات مُفعلة:
- ✅ **Elasticsearch**: http://localhost:9200
- ✅ **Kibana**: http://localhost:5601  
- ✅ **Logstash**: يستورد البيانات تلقائياً كل دقيقة

### البيانات المستوردة:
- ✅ **فهرس**: `book-pages` تم إنشاؤه
- ✅ **الوثائق**: 50+ صفحة من الكتب العربية
- ✅ **المحلل**: Arabic Analyzer مُفعل

## 🚀 بدء الاستخدام

### 1. افتح Kibana
```
http://localhost:5601
```

### 2. إنشاء Index Pattern
1. انتقل إلى **Stack Management** > **Index Patterns**
2. اضغط **Create index pattern**
3. اكتب: `book-pages`
4. اختر **@timestamp** كـ Time field
5. اضغط **Create index pattern**

### 3. البحث في البيانات
1. انتقل إلى **Discover**
2. ستجد بياناتك جاهزة للبحث
3. استخدم شريط البحث للبحث باللغة العربية

## 📊 الحقول المتاحة للبحث

| الحقل | الوصف | مثال |
|-------|--------|-------|
| `content` | النص الكامل للصفحة | النص العربي |
| `book_title` | عنوان الكتاب | "أثر الصحابي علي" |
| `author_name` | اسم المؤلف | "Unknown Author" |
| `page_number` | رقم الصفحة | 1, 2, 3... |
| `book_id` | معرف الكتاب | "10", "11" |
| `document_title` | عنوان الوثيقة | "الكتاب - صفحة X" |

## 🔍 أمثلة على البحث

### في Kibana Discover:
```
content: "الصلاة"
book_title: "الفقه"
author_name: "علي"
content: "أبو حنيفة" AND book_id: "10"
```

### في Dev Tools:
```json
GET book-pages/_search
{
  "query": {
    "match": {
      "content": "الإمام"
    }
  },
  "size": 5
}
```

## 📈 إنشاء Dashboard

1. انتقل إلى **Analytics** > **Dashboard**
2. اضغط **Create dashboard**
3. أضف visualizations مثل:
   - **Pie chart**: توزيع الكتب
   - **Data table**: قائمة النتائج
   - **Metric**: عدد الصفحات الكلي

## 🔧 التحقق من النظام

### فحص البيانات:
```bash
# عدد الوثائق
curl -u elastic:changeme "http://localhost:9200/book-pages/_count"

# حالة الفهرس
curl -u elastic:changeme "http://localhost:9200/_cat/indices?v"
```

### مراقبة Logstash:
```bash
docker logs docker-elk-logstash-1 --tail 20
```

## 🎯 نصائح للبحث الفعال

1. **البحث البسيط**: استخدم كلمات مفردة
2. **البحث بالعبارة**: استخدم علامات الاقتباس `"أبو حنيفة"`
3. **البحث المتعدد**: استخدم `AND` و `OR`
4. **البحث الجزئي**: استخدم `*` مثل `الفقه*`

## 🚨 استكشاف الأخطاء

### إذا لم تظهر البيانات:
1. تأكد من تشغيل جميع الخدمات:
   ```bash
   docker-compose ps
   ```

2. فحص logs:
   ```bash
   docker logs docker-elk-logstash-1
   docker logs docker-elk-elasticsearch-1
   ```

3. إعادة تشغيل إذا لزم الأمر:
   ```bash
   docker-compose restart
   ```

## 📋 الخطوات التالية

1. ✅ **مكتمل**: إعداد ELK Stack
2. ✅ **مكتمل**: استيراد البيانات العربية  
3. ✅ **مكتمل**: تفعيل Arabic Analyzer
4. 🔄 **التالي**: إنشاء dashboards مخصصة
5. 🔄 **التالي**: إضافة المزيد من الكتب
6. 🔄 **التالي**: النشر على VPS للإنتاج

---

**🎉 النظام جاهز للاستخدام! افتح Kibana وابدأ البحث في النصوص العربية.**

للمساعدة: راجع الوثائق الرسمية لـ [Kibana](https://www.elastic.co/guide/en/kibana/current/index.html)
