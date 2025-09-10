# دليل تشغيل نظام البحث في محتوى الكتب

## الإعداد السريع للاختبار المحلي

### 1. تأكد من وجود JDBC Driver

تأكد من وجود ملف `mysql-connector-j-8.0.33.jar` في مجلد `logstash/drivers/`

### 2. تشغيل النظام

```powershell
# إعداد المستخدمين (مرة واحدة فقط)
docker-compose --profile=setup up setup

# تشغيل النظام
docker-compose up -d
```

### 3. اختبار الاتصال

- Elasticsearch: <http://localhost:9200>
- Kibana: <http://localhost:5601>

### 4. مراقبة الفهرسة

```powershell
# مراقبة logs لـ Logstash
docker-compose logs -f logstash

# فحص الفهرس
curl -X GET "localhost:9200/book-pages/_search?pretty"
```

### 5. البحث في Kibana

1. افتح Kibana على <http://localhost:5601>
2. اذهب إلى Stack Management > Index Patterns
3. أنشئ Index Pattern: `book-pages*`
4. اذهب إلى Discover للبحث في المحتوى

### 6. البحث بـ API

```bash
# البحث الأساسي
curl -X GET "localhost:9200/book-pages/_search" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match": {
      "search_text": "النص المراد البحث عنه"
    }
  }
}'

# البحث المتقدم
curl -X GET "localhost:9200/book-pages/_search" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "should": [
        {"match": {"content": "النص المراد البحث عنه"}},
        {"match": {"book_title": "عنوان الكتاب"}},
        {"match": {"author_name": "اسم المؤلف"}}
      ]
    }
  },
  "highlight": {
    "fields": {
      "content": {}
    }
  }
}'
```

## الميزات المحسنة

- ✅ Arabic analyzer مدمج للبحث العربي الدقيق
- ✅ فهرسة محتوى الصفحات فقط للسرعة
- ✅ إعدادات محسنة للأداء (refresh_interval: 5s)
- ✅ عدد shards مناسب للاختبار المحلي
- ✅ استعلام SQL محسن للحصول على البيانات الأساسية فقط

## استكشاف الأخطاء

- إذا لم تظهر البيانات، تحقق من logs: `docker-compose logs logstash`
- تأكد من الاتصال بقاعدة البيانات: `docker-compose exec logstash cat /usr/share/logstash/data/.logstash_jdbc_last_run_pages`
