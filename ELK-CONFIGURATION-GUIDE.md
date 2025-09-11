# دليل إعدادات ELK Stack الشامل

## نظرة عامة
هذا الدليل يوضح جميع إعدادات مجموعة ELK Stack (Elasticsearch, Logstash, Kibana) مع شرح مفصل لكل إعداد ووظيفته والتوصيات المناسبة لخادم VPS الخاص بك.

### مواصفات VPS الحالية
- **نوع الخطة**: KVM 8
- **المعالج**: 8 أنوية (CPU cores)
- **الذاكرة**: 32 GB RAM
- **مساحة التخزين**: 400 GB SSD
- **تاريخ انتهاء الصلاحية**: 2026-08-29
- **التجديد التلقائي**: مفعل (Auto-renewal: On)

### تقييم الموارد المتاحة:
- **قوة معالجة ممتازة**: 8 أنوية كافية لتشغيل ELK Stack بكفاءة عالية
- **ذاكرة وفيرة**: 32GB تسمح بتخصيص ذاكرة كبيرة لـ Elasticsearch و Logstash
- **تخزين كافي**: 400GB مناسب لتخزين البيانات والفهارس لفترة طويلة
- **استقرار طويل المدى**: صالح حتى 2026 مع تجديد تلقائي

---

## 1. ملف Docker Compose الرئيسي

### الملف: `docker-compose.yml`

| الخدمة | الوصف | المنافذ المستخدمة | الإعداد الحالي | التوصية المحسنة | السبب |
|---------|-------|------------------|----------------|------------------|--------|
| **setup** | خدمة الإعداد الأولي للمستخدمين والأدوار | - | تشغيل عند الحاجة | تشغيل مرة واحدة فقط | إعداد أولي للأمان |
| **kibana-genkeys** | توليد مفاتيح التشفير لـ Kibana | - | غير مفعل | تشغيل وتفعيل المفاتيح | حماية البيانات الحساسة |
| **elasticsearch** | محرك البحث والفهرسة الرئيسي | 9200, 9300 | 512MB Java Heap | **12-16GB Java Heap** | استغلال 40-50% من الذاكرة المتاحة |
| **logstash** | معالج ومحول البيانات | 5044, 50000, 9600 | 256MB Java Heap | **4-6GB Java Heap** | معالجة أسرع للبيانات الكبيرة |
| **kibana** | واجهة المستخدم والتصور | 5601 | إعداد افتراضي | **2-4GB Memory Limit** | أداء أفضل للواجهة |

### التوصيات المحسنة لـ VPS (32GB RAM, 8 CPU):

```yaml
# إعدادات محسنة للذاكرة - استغلال أمثل للموارد
services:
  elasticsearch:
    environment:
      ES_JAVA_OPTS: -Xms12g -Xmx12g    # Elasticsearch: 12GB (37.5% من الذاكرة)
    deploy:
      resources:
        limits:
          memory: 16g
          cpus: '4.0'                   # 4 أنوية من أصل 8
        reservations:
          memory: 12g
          cpus: '2.0'

  logstash:
    environment:
      LS_JAVA_OPTS: -Xms4g -Xmx4g      # Logstash: 4GB (12.5% من الذاكرة)
    deploy:
      resources:
        limits:
          memory: 6g
          cpus: '2.0'                   # 2 أنوية
        reservations:
          memory: 4g
          cpus: '1.0'

  kibana:
    deploy:
      resources:
        limits:
          memory: 4g                    # Kibana: 4GB
          cpus: '1.0'                   # 1 نواة
        reservations:
          memory: 2g
          cpus: '0.5'
```

### توزيع الذاكرة المقترح (32GB إجمالي):
- **Elasticsearch**: 12GB (37.5%) - الأولوية العليا
- **Logstash**: 4GB (12.5%) - معالجة البيانات
- **Kibana**: 4GB (12.5%) - واجهة المستخدم
- **النظام**: 6GB (18.75%) - نظام التشغيل والخدمات
- **احتياطي**: 6GB (18.75%) - للتطبيقات الأخرى والنمو المستقبلي

---

## 2. إعدادات Elasticsearch

### الملف: `elasticsearch/config/elasticsearch.yml`

| الإعداد | القيمة الحالية | الوصف | التوصية المحسنة | السبب | الأهمية |
|---------|-----------------|-------|------------------|--------|----------|
| `cluster.name` | docker-cluster | اسم الكلاستر | `bms-production-cluster` | تمييز أفضل في البيئة الإنتاجية | متوسطة |
| `network.host` | 0.0.0.0 | عنوان الشبكة | **الاحتفاظ بـ 0.0.0.0** | يسمح بالوصول من جميع الواجهات في Docker | عالية |
| `xpack.license.self_generated.type` | trial | نوع الترخيص | **الاحتفاظ بـ trial** | يوفر جميع الميزات لمدة 30 يوم | متوسطة |
| `xpack.security.enabled` | true | تفعيل الأمان | **يجب الاحتفاظ بـ true** | حماية أساسية ضد الوصول غير المصرح | عالية جداً |
| `ES_JAVA_OPTS` | -Xms512m -Xmx512m | ذاكرة Java | **-Xms12g -Xmx12g** | استغلال أمثل لموارد VPS (32GB) | عالية جداً |

### إعدادات إضافية مقترحة:

```yaml
# إعدادات الأداء
indices.memory.index_buffer_size: 20%
indices.memory.min_index_buffer_size: 96mb

# إعدادات التخزين
path.data: /usr/share/elasticsearch/data
path.logs: /usr/share/elasticsearch/logs

# إعدادات الشبكة
http.port: 9200
transport.port: 9300

# إعدادات الأمان المتقدمة
xpack.security.transport.ssl.enabled: true
xpack.security.http.ssl.enabled: false  # للتطوير فقط
```

---

## 3. إعدادات Kibana

### الملف: `kibana/config/kibana.yml`

| الإعداد | القيمة الحالية | الوصف | التوصية المحسنة | السبب | الأهمية |
|---------|-----------------|-------|------------------|--------|----------|
| `server.name` | kibana | اسم الخادم | `bms-kibana-server` | تمييز أفضل في السجلات والمراقبة | منخفضة |
| `server.host` | 0.0.0.0 | عنوان الخادم | **الاحتفاظ بـ 0.0.0.0** | يسمح بالوصول من جميع الواجهات | عالية |
| `elasticsearch.hosts` | http://elasticsearch:9200 | عناوين Elasticsearch | **الاحتفاظ بالقيمة الحالية** | يعمل بشكل صحيح في Docker Compose | عالية |
| `elasticsearch.username` | kibana_system | اسم المستخدم | **الاحتفاظ بـ kibana_system** | مستخدم مدمج مخصص لـ Kibana | عالية |
| `elasticsearch.password` | ${KIBANA_SYSTEM_PASSWORD} | كلمة المرور | **استخدام كلمة مرور قوية** | حماية الوصول لـ Elasticsearch | عالية جداً |
| `monitoring.ui.container.elasticsearch.enabled` | true | مراقبة Elasticsearch | **الاحتفاظ بـ true** | مراقبة أداء وحالة Elasticsearch | متوسطة |
| `monitoring.ui.container.logstash.enabled` | true | مراقبة Logstash | **الاحتفاظ بـ true** | مراقبة معالجة البيانات | متوسطة |
| `xpack.encryptedSavedObjects.encryptionKey` | غير مفعل | مفتاح تشفير الكائنات | **تفعيل مفتاح 32 حرف** | حماية البيانات المحفوظة الحساسة | عالية |

### إعدادات Fleet المتقدمة:

| المكون | الوصف | التوصية |
|---------|-------|----------|
| `xpack.fleet.agents.fleet_server.hosts` | خوادم Fleet Server | تفعيل عند الحاجة لإدارة Agents |
| `xpack.fleet.packages` | الحزم المثبتة | الاحتفاظ بالحزم الأساسية فقط |

---

## 4. إعدادات Logstash

### الملف: `logstash/config/logstash.yml`

| الإعداد | القيمة الحالية | الوصف | التوصية المحسنة | السبب | الأهمية |
|---------|-----------------|-------|------------------|--------|----------|
| `api.http.host` | 0.0.0.0 | عنوان API | **الاحتفاظ بـ 0.0.0.0** | يسمح بمراقبة Logstash من Kibana | متوسطة |
| `node.name` | logstash | اسم العقدة | `bms-logstash-node` | تمييز أفضل في المراقبة والسجلات | منخفضة |
| `LS_JAVA_OPTS` | -Xms256m -Xmx256m | ذاكرة Java | **-Xms4g -Xmx4g** | معالجة أسرع للبيانات الكبيرة | عالية جداً |
| `pipeline.workers` | افتراضي (1) | عدد العمال | **4 workers** | استغلال 50% من أنوية المعالج | عالية |
| `pipeline.batch.size` | 125 | حجم الدفعة | **1000** | معالجة أكثر كفاءة للبيانات | متوسطة |
| `queue.type` | memory | نوع الطابور | **persisted** | حماية من فقدان البيانات عند إعادة التشغيل | عالية |

### إعدادات إضافية مقترحة:

```yaml
# إعدادات الأداء
pipeline.workers: 4                    # عدد العمال (25% من الأنوية)
pipeline.batch.size: 1000              # حجم الدفعة
pipeline.batch.delay: 50               # تأخير الدفعة بالميلي ثانية

# إعدادات الذاكرة
queue.type: persisted                  # نوع الطابور المستمر
queue.max_bytes: 2gb                   # الحد الأقصى لحجم الطابور

# إعدادات المراقبة
monitoring.enabled: true
monitoring.elasticsearch.hosts: ["http://elasticsearch:9200"]
```

### الملف: `logstash/config/pipelines.yml`

| خط المعالجة | الملف | الوصف | التوصية |
|--------------|-------|-------|----------|
| `main` | logstash.conf | خط المعالجة الرئيسي | للبيانات العامة |
| `mysql-books` | mysql-books.conf | معالجة بيانات الكتب | مخصص لقاعدة بيانات BMS |

---

## 5. إعدادات خطوط المعالجة

### الملف: `logstash/pipeline/logstash.conf`

| القسم | المنفذ/الإعداد | الوصف | التوصية |
|-------|---------------|-------|----------|
| **Input - Beats** | 5044 | استقبال بيانات من Filebeat/Metricbeat | الاحتفاظ بالإعداد |
| **Input - TCP** | 50000 | استقبال بيانات TCP عامة | يمكن تعطيله إذا لم يُستخدم |
| **Output - Elasticsearch** | elasticsearch:9200 | إرسال البيانات لـ Elasticsearch | الاحتفاظ بالإعداد |

### الملف: `logstash/pipeline/mysql-books.conf`

| المكون | الإعداد | القيمة الحالية | التوصية المحسنة | السبب | الأهمية |
|---------|---------|-----------------|------------------|--------|----------|
| **JDBC Connection** | jdbc_connection_string | MySQL على 145.223.98.97:3306 | **إضافة SSL وتشفير الاتصال** | حماية بيانات قاعدة البيانات أثناء النقل | عالية |
| **Schedule** | schedule | كل دقيقة (\* \* \* \* \*) | **كل 5 دقائق (\*/5 \* \* \* \*)** | تقليل الحمل على قاعدة البيانات | عالية |
| **Batch Size** | LIMIT | 10 سجلات | **500-1000 سجل** | معالجة أكثر كفاءة مع موارد VPS الكبيرة | متوسطة |
| **Index Name** | index | book-pages | **book-pages-{+YYYY.MM}** | تنظيم أفضل للبيانات حسب الشهر | متوسطة |
| **Character Encoding** | charset | UTF-8 | **الاحتفاظ بـ UTF-8** | دعم كامل للنصوص العربية | عالية |
| **Connection Pool** | غير محدد | اتصال واحد | **إضافة connection pooling** | أداء أفضل مع قاعدة البيانات | متوسطة |

---

## 6. متغيرات البيئة المطلوبة

### الملف: `.env`

| المتغير | الوصف | التوصية | الأهمية |
|---------|-------|----------|----------|
| `ELASTIC_VERSION` | إصدار ELK Stack | 8.11.0 أو أحدث | عالية |
| `ELASTIC_PASSWORD` | كلمة مرور المستخدم الرئيسي | كلمة مرور قوية (16+ حرف) | عالية جداً |
| `LOGSTASH_INTERNAL_PASSWORD` | كلمة مرور Logstash | كلمة مرور قوية | عالية |
| `KIBANA_SYSTEM_PASSWORD` | كلمة مرور نظام Kibana | كلمة مرور قوية | عالية |
| `METRICBEAT_INTERNAL_PASSWORD` | كلمة مرور Metricbeat | كلمة مرور قوية | متوسطة |
| `FILEBEAT_INTERNAL_PASSWORD` | كلمة مرور Filebeat | كلمة مرور قوية | متوسطة |

---

## 7. توصيات الأمان

### إعدادات الأمان الأساسية

| الإعداد | التوصية | السبب |
|---------|----------|--------|
| **كلمات المرور** | استخدام كلمات مرور قوية (16+ حرف) | منع الاختراق |
| **مفاتيح التشفير** | تفعيل مفاتيح التشفير في Kibana | حماية البيانات الحساسة |
| **SSL/TLS** | تفعيل HTTPS في الإنتاج | تشفير البيانات المنقولة |
| **Firewall** | تقييد الوصول للمنافذ | السماح فقط للعناوين المصرح بها |

### أوامر تفعيل مفاتيح التشفير:

```bash
# توليد مفاتيح التشفير
docker compose up kibana-genkeys

# إضافة المفاتيح إلى kibana.yml
xpack.security.encryptionKey: "your-32-char-key-here"
xpack.encryptedSavedObjects.encryptionKey: "your-32-char-key-here"
xpack.reporting.encryptionKey: "your-32-char-key-here"
```

---

## 8. توصيات الأداء لـ VPS

### تحسين استخدام الذاكرة

| الخدمة | الإعداد الحالي | التوصية المحسنة (32GB) | النسبة | السبب | الأولوية |
|---------|----------------|------------------------|--------|--------|----------|
| **Elasticsearch** | 512MB Java Heap | **12GB Java Heap** | 37.5% | محرك البحث يحتاج ذاكرة كبيرة للفهرسة والبحث | عالية جداً |
| **Logstash** | 256MB Java Heap | **4GB Java Heap** | 12.5% | معالجة البيانات الكبيرة من قاعدة البيانات | عالية |
| **Kibana** | إعداد افتراضي | **4GB Memory Limit** | 12.5% | واجهة مستخدم سريعة ومتجاوبة | متوسطة |
| **النظام والخدمات** | غير محدد | **6GB مخصص** | 18.75% | نظام التشغيل وDocker وخدمات أخرى | عالية |
| **احتياطي ونمو** | - | **6GB متاح** | 18.75% | للتوسع المستقبلي وتطبيقات إضافية | متوسطة |

### مقارنة الأداء المتوقع:
| المؤشر | الإعداد الحالي | بعد التحسين | التحسن المتوقع |
|---------|----------------|-------------|------------------|
| **سرعة البحث** | بطيء مع البيانات الكبيرة | سريع جداً | **20-30x أسرع** |
| **معالجة البيانات** | 10 سجلات/دقيقة | 500-1000 سجل/5 دقائق | **10-20x أكثر كفاءة** |
| **استجابة Kibana** | بطيء أحياناً | سريع ومتجاوب | **5-10x تحسن** |
| **استقرار النظام** | قد يتوقف مع الحمل العالي | مستقر تحت الضغط | **استقرار كامل** |

### إعدادات Docker Compose المحسنة:

```yaml
services:
  elasticsearch:
    environment:
      ES_JAVA_OPTS: -Xms12g -Xmx12g
    deploy:
      resources:
        limits:
          memory: 16g
        reservations:
          memory: 12g

  logstash:
    environment:
      LS_JAVA_OPTS: -Xms4g -Xmx4g
    deploy:
      resources:
        limits:
          memory: 6g
        reservations:
          memory: 4g
```

---

## 9. مراقبة الأداء

### مؤشرات الأداء المهمة

| المؤشر | الوصف | القيمة المثلى | كيفية المراقبة |
|---------|-------|---------------|----------------|
| **CPU Usage** | استخدام المعالج | < 80% | htop, docker stats |
| **Memory Usage** | استخدام الذاكرة | < 85% | free -h, docker stats |
| **Disk I/O** | عمليات القراءة/الكتابة | < 80% | iostat, iotop |
| **Elasticsearch Heap** | ذاكرة Java لـ ES | < 75% | Kibana Monitoring |
| **Logstash Heap** | ذاكرة Java لـ LS | < 75% | Kibana Monitoring |

### أوامر المراقبة:

```bash
# مراقبة استخدام الموارد
docker stats

# مراقبة سجلات الخدمات
docker compose logs -f elasticsearch
docker compose logs -f logstash
docker compose logs -f kibana

# فحص حالة Elasticsearch
curl -X GET "localhost:9200/_cluster/health?pretty"
```

---

## 10. النسخ الاحتياطي والاستعادة

### استراتيجية النسخ الاحتياطي

| المكون | التكرار | الطريقة | الأهمية |
|---------|---------|---------|----------|
| **بيانات Elasticsearch** | يومياً | Snapshot API | عالية جداً |
| **إعدادات التكوين** | أسبوعياً | نسخ الملفات | عالية |
| **قاعدة بيانات MySQL** | يومياً | mysqldump | عالية جداً |

### أوامر النسخ الاحتياطي:

```bash
# إنشاء snapshot لـ Elasticsearch
curl -X PUT "localhost:9200/_snapshot/backup_repo/snapshot_$(date +%Y%m%d)" \
  -H 'Content-Type: application/json' \
  -d '{ "indices": "book-pages", "ignore_unavailable": true }'

# نسخ احتياطي لملفات الإعدادات
tar -czf elk-config-backup-$(date +%Y%m%d).tar.gz \
  elasticsearch/config/ kibana/config/ logstash/config/ docker-compose.yml .env
```

---

## 11. استكشاف الأخطاء وإصلاحها

### المشاكل الشائعة والحلول

| المشكلة | السبب المحتمل | الحل |
|---------|---------------|------|
| **Elasticsearch لا يبدأ** | نقص في الذاكرة | زيادة heap size أو تقليل استخدام الذاكرة |
| **Kibana لا يتصل بـ ES** | مشكلة في المصادقة | التحقق من كلمات المرور في .env |
| **Logstash لا يستقبل بيانات** | مشكلة في المنافذ | التحقق من إعدادات الشبكة والمنافذ |
| **بطء في الاستعلامات** | فهرسة غير محسنة | تحسين mapping وإعدادات الفهرس |
| **امتلاء القرص الصلب** | تراكم البيانات | إعداد سياسات حذف البيانات القديمة |

### أوامر التشخيص:

```bash
# فحص حالة الخدمات
docker compose ps

# فحص سجلات الأخطاء
docker compose logs elasticsearch | grep ERROR
docker compose logs logstash | grep ERROR
docker compose logs kibana | grep ERROR

# فحص استخدام الموارد
docker stats --no-stream

# فحص مساحة القرص
df -h
du -sh /var/lib/docker/volumes/
```

---

## 12. إعدادات Extensions (الإضافات)

### Filebeat - جمع ملفات السجلات

| الإعداد | الوصف | التوصية |
|---------|-------|----------|
| **المسارات المراقبة** | `/var/log/*.log` | إضافة مسارات سجلات التطبيقات |
| **معدل الإرسال** | كل 10 ثوان | تقليل إلى 30-60 ثانية للإنتاج |
| **المرشحات** | تصفية السجلات | إضافة مرشحات لتقليل الضوضاء |

### Metricbeat - مراقبة الأداء

| المؤشر | الوصف | الأهمية |
|---------|-------|----------|
| **System Module** | مراقبة النظام | عالية |
| **Docker Module** | مراقبة الحاويات | عالية |
| **MySQL Module** | مراقبة قاعدة البيانات | متوسطة |

### Heartbeat - مراقبة التوفر

| الخدمة | URL | التكرار |
|---------|-----|----------|
| **Elasticsearch** | http://localhost:9200 | كل دقيقة |
| **Kibana** | http://localhost:5601 | كل دقيقة |
| **تطبيق BMS** | http://your-app-url | كل 30 ثانية |

---

## 13. تحسين الأداء المتقدم

### إعدادات Elasticsearch المتقدمة

```yaml
# تحسين الفهرسة
index.refresh_interval: 30s
index.number_of_shards: 1
index.number_of_replicas: 0

# تحسين البحث
index.search.slowlog.threshold.query.warn: 10s
index.search.slowlog.threshold.fetch.warn: 1s

# إدارة الذاكرة
indices.fielddata.cache.size: 20%
indices.queries.cache.size: 10%
```

### إعدادات Logstash المتقدمة

```yaml
# تحسين المعالجة
pipeline.workers: 8
pipeline.batch.size: 2000
pipeline.batch.delay: 5

# إدارة الذاكرة
queue.type: persisted
queue.max_bytes: 4gb
queue.checkpoint.writes: 1024
```

---

## 14. سيناريوهات الاستخدام

### للتطوير والاختبار

```yaml
# إعدادات مخففة للتطوير
ES_JAVA_OPTS: -Xms1g -Xmx1g
LS_JAVA_OPTS: -Xms512m -Xmx512m

# تعطيل الأمان للاختبار السريع
xpack.security.enabled: false
```

### للإنتاج

```yaml
# إعدادات محسنة للإنتاج
ES_JAVA_OPTS: -Xms12g -Xmx12g
LS_JAVA_OPTS: -Xms4g -Xmx4g

# تفعيل جميع ميزات الأمان
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.http.ssl.enabled: true
```

---

## الخلاصة والتوصيات النهائية

### الأولويات العالية:
1. ✅ **تحديث إعدادات الذاكرة** لاستغلال موارد VPS بشكل أمثل
2. ✅ **تفعيل مفاتيح التشفير** في Kibana لحماية البيانات
3. ✅ **إعداد كلمات مرور قوية** لجميع المستخدمين
4. ✅ **تنفيذ استراتيجية النسخ الاحتياطي** اليومية

### الأولويات المتوسطة:
1. 🔄 **تحسين جدولة Logstash** لتقليل الحمل على قاعدة البيانات
2. 🔄 **إعداد مراقبة الأداء** المستمرة
3. 🔄 **تحسين إعدادات الشبكة** والأمان

### الأولويات المنخفضة:
1. 📋 **تخصيص أسماء الخدمات** لتكون أكثر وصفية
2. 📋 **إضافة إضافات Elasticsearch** حسب الحاجة
3. 📋 **تحسين واجهة Kibana** وإضافة لوحات مراقبة مخصصة

### نصائح للصيانة:
- **مراقبة يومية**: فحص استخدام الموارد والسجلات
- **تحديث أسبوعي**: تطبيق التحديثات الأمنية
- **مراجعة شهرية**: تحليل الأداء وتحسين الإعدادات
- **نسخ احتياطي**: تأكد من عمل النسخ الاحتياطية التلقائية

هذا الدليل يوفر أساساً قوياً لتشغيل ELK Stack بكفاءة على VPS الخاص بك مع الاستفادة الكاملة من الموارد المتاحة والحفاظ على أعلى مستويات الأمان والأداء. 🚀