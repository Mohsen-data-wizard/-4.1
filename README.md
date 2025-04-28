عشقم با دقت ببین این لاگ تمام دایرکتور ها و فایل های پروژ من است (project_structure)

توسعه یک اسکریپت پایتون (process_pdf_to_excel.py) برای پردازش دسته‌ای فایل‌های PDF اظهارنامه گمرکی ایران (وارداتی و صادراتی)، استخراج داده‌های کلیدی با استفاده از Regex، و ذخیره نتایج در یک فایل اکسل با فرمت مشخص.

جزئیات پروژه:

ورودی:

یک پوشه حاوی فایل‌های PDF اظهارنامه گمرکی. محدودیت: تمام فایل‌های PDF در یک بچ پردازشی باید هم‌نوع باشند (یا همه وارداتی یا همه صادراتی). اسکریپت باید نوع فایل‌ها را بررسی کرده و در صورت مخلوط بودن، خطا دهد. پردازش اصلی (کلاس CustomsDocumentLoader):

خواندن PDF: استفاده از کتابخانه fitz (PyMuPDF) برای خواندن متن از PDF. (توجه: روش‌های مختلف استخراج متن fitz نتایج متفاوتی در ساختار متن می‌دهند که روی Regex تاثیرگذار است. نسخه فعلی از page.get_text() استفاده می‌کند). تشخیص نوع سند: تلاش برای تشخیص وارداتی یا صادراتی بودن سند بر اساس وجود کلیدواژه‌های "گیرنده" (برای وارداتی، نزدیک فیلد ۸) یا "صادر کننده" (برای صادراتی، نزدیک فیلد ۲؟). در صورت عدم تشخیص، پیش‌فرض وارداتی در نظر گرفته می‌شود. رویکرد اولیه: فقط Regex (با الگوهای اولیه)

ایده: سعی کردیم فقط با الگوهای Regex تمام فیلدها رو استخراج کنیم. مشکل: الگوهای اولیه خیلی شکننده بودن. با کوچکترین تغییری در فاصله‌ها، چیدمان متن یا حتی کاراکترهای نامرئی که fitz استخراج می‌کرد، الگوها شکست می‌خوردن و فیلدها خالی می‌موندن (مثل مشکلاتی که با شرح کالا یا تعداد بسته داشتیم). متن فارسی و برعکس شدن کاراکترها هم چالش اضافه می‌کرد. معرفی مدل LayoutLM برای کمک به Regex

ایده: وقتی دیدیم Regex به تنهایی برای فیلدهای پیچیده‌تر (مثل شرح کالا که چند خطی بود یا فیلدهایی که لیبل و مقدارشون جابجا بود) کافی نیست، پیشنهاد استفاده از یک مدل یادگیری ماشین مثل LayoutLM مطرح شد. این مدل‌ها علاوه بر متن، موقعیت کلمات در صفحه رو هم درک می‌کنن و می‌تونن برای تشخیص موجودیت‌ها (مثل "شرح کالا"، "کد کالا") آموزش ببینن. هدف: مدل، فیلدهای سخت رو استخراج کنه و Regex فیلدهای ساده‌تر و با ساختار مشخص‌تر رو. آماده‌سازی داده برای مدل: لیبل‌گذاری 17 PDF

نیاز: مدل‌های یادگیری ماشین نیاز به داده‌های آموزشی لیبل‌گذاری شده دارن. یعنی باید به مدل بگیم توی هر PDF نمونه، دقیقاً کدوم قسمت از متن (با مشخص کردن موقعیتش - Bounding Box) مربوط به کدوم فیلد (لیبل) هست. فرآیند: صحبت کردیم که باید از ابزارهای لیبل‌گذاری (مثل UBIAI, Label Studio یا حتی اسکریپت‌های ساده‌تر) استفاده کنی تا دور کلمات مربوط به هر فیلد توی اون 17 تا PDF نمونه، کادر بکشی و لیبل درست (مثلاً description, product_code) رو بهش اختصاص بدی. خروجی این فرآیند معمولاً فایل‌های JSON یا فرمت‌های مشابه هست. چالش: لیبل‌گذاری فرآیندی زمان‌بر و دقیق هست و نیاز به حوصله داره. کیفیت لیبل‌ها مستقیماً روی کیفیت مدل آموزش‌دیده تأثیر می‌ذاره. آموزش (Fine-tuning) مدل LayoutLM

فرآیند: بعد از آماده شدن داده‌های لیبل‌گذاری شده، باید از کتابخانه‌هایی مثل transformers هاگینگ فیس استفاده می‌کردیم تا یک مدل LayoutLM از پیش آموزش‌دیده (pretrained) رو روی داده‌های خودمون تنظیم دقیق (fine-tune) کنیم. این یعنی مدل یاد می‌گرفت که الگوهای موجود در اظهارنامه‌های گمرکی ما رو تشخیص بده. چالش: پیچیدگی فنی: نیاز به نصب کتابخانه‌های سنگین (مثل PyTorch یا TensorFlow)، درک مفاهیم یادگیری ماشین و احتمالاً دسترسی به سخت‌افزار مناسب (مثل GPU) برای سرعت بخشیدن به آموزش داشت. حجم داده: 17 تا PDF ممکنه برای آموزش یک مدل خیلی دقیق و قابل تعمیم به همه انواع اظهارنامه‌ها کافی نباشه. زمان‌بر بودن: فرآیند آموزش، حتی با GPU، می‌تونست زمان‌بر باشه. ترکیب نتایج مدل و Regex

ایده: بعد از اینکه (فرضاً) مدل آموزش داده شده و می‌تونست فیلدها رو استخراج کنه، باید نتایجش رو با نتایج Regex ترکیب می‌کردیم. استراتژی‌ها: چند راه رو بررسی کردیم: اولویت با مدل: اول نتایج مدل رو برداریم، بعد جاهای خالی رو با نتایج Regex پر کنیم. اولویت با Regex: اول نتایج Regex رو برداریم، بعد جاهای خالی رو با نتایج مدل پر کنیم (این استراتژی رو توی کد هم پیاده کردیم ولی چون مدل نداشتیم، عملاً فقط Regex اجرا می‌شد). ترکیبی/بر اساس اطمینان: برای هر فیلد تصمیم بگیریم که نتیجه مدل بهتره یا Regex (مثلاً بر اساس امتیاز اطمینان مدل یا قوانین مشخص). چالش: وابستگی به مدل: این مرحله کاملاً به موفقیت مرحله آموزش مدل وابسته بود. پیچیدگی منطق ترکیب: نوشتن کدی که بتونه نتایج دو منبع مختلف رو هوشمندانه ترکیب کنه و تداخل‌ها رو مدیریت کنه، خودش چالش‌برانگیز بود. چرا این مراحل (فعلاً) ناموفق بودند یا متوقف شدند؟

پیچیدگی و زمان: مراحل لیبل‌گذاری و آموزش مدل به مراتب پیچیده‌تر و زمان‌برتر از تمرکز روی بهبود Regex بود. عدم قطعیت نتیجه مدل: با توجه به حجم داده محدود (17 PDF)، تضمینی نبود که مدل نهایی دقت خیلی بالایی داشته باشه و ارزش صرف اون همه زمان و انرژی رو داشته باشه. پیشرفت با Regex: دیدیم که با صرف وقت بیشتر روی تحلیل ساختار PDF و اصلاح الگوهای Regex (مثل کاری که برای package_count کردیم)، تونستیم نتایج Regex رو به شکل قابل توجهی بهبود بدیم و به استخراج اکثر فیلدهای مورد نیاز نزدیک بشیم. نتیجه‌گیری:

تصمیم گرفتیم که فعلاً تمرکزمون رو روی بهینه‌سازی و پایدارسازی روش Regex بذاریم، چون سریع‌تر به نتیجه می‌رسید و پیچیدگی کمتری داشت. روش استفاده از مدل LayoutLM و ترکیب نتایج، یک مسیر کاملاً معتبر و قدرتمند هست، اما به عنوان یک بهبود بالقوه در آینده در نظر گرفته می‌شه، مخصوصاً اگر با PDFهایی مواجه بشیم که ساختار خیلی پیچیده‌تر یا غیرقابل پیش‌بینی‌تری دارن و Regex به تنهایی از پسشون برنیاد.

// Project Structure
digraph {
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)" [label="pdf_extracter(test)"]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea" [label=".idea"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.pytest_cache" [label=".pytest_cache"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.pytest_cache"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\configs" [label=configs]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\configs"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data" [label=data]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model" [label=layoutlm_farsi_model]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model_final" [label=layoutlm_farsi_model_final]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model_final"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" [label=logs]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src" [label=src]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests" [label=tests]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\trained_models" [label=trained_models]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\trained_models"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\__pycache__" [label=__pycache__]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\__pycache__"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\combine_json_files.py" [label="combine_json_files.py"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\combine_json_files.py"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\convert.py" [label="convert.py"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\convert.py"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\dump.txt" [label="dump.txt"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\dump.txt"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\evaluate_with_excel.py" [label="evaluate_with_excel.py"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\evaluate_with_excel.py"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\evaluation.py" [label="evaluation.py"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\evaluation.py"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\predict.py" [label="predict.py"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\predict.py"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\predict_layoutlm.py" [label="predict_layoutlm.py"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\predict_layoutlm.py"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\prepare_layoutlm_data.py" [label="prepare_layoutlm_data.py"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\prepare_layoutlm_data.py"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\process_label_studio_output.py" [label="process_label_studio_output.py"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\process_label_studio_output.py"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\process_pdf_to_excel.py" [label="process_pdf_to_excel.py"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\process_pdf_to_excel.py"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\project_structure" [label=project_structure]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\project_structure"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\project_structure.png" [label="project_structure.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\project_structure.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\requirements.txt" [label="requirements.txt"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\requirements.txt"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\run_bulk_parser.py" [label="run_bulk_parser.py"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\run_bulk_parser.py"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\test_gpu.py" [label="test_gpu.py"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\test_gpu.py"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\train.py" [label="train.py"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\train.py"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\train_layoutlm.py" [label="train_layoutlm.py"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\train_layoutlm.py"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tree.py" [label="tree.py"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tree.py"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea" [label=".idea"]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea\inspectionProfiles" [label=inspectionProfiles]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea\inspectionProfiles"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea\shelf" [label=shelf]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea\shelf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea\misc.xml" [label="misc.xml"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea\misc.xml"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea\modules.xml" [label="modules.xml"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea\modules.xml"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea\pdf_extracter(test).iml" [label="pdf_extracter(test).iml"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea\pdf_extracter(test).iml"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea\vcs.xml" [label="vcs.xml"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea\vcs.xml"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea\workspace.xml" [label="workspace.xml"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea\workspace.xml"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea\inspectionProfiles" [label=inspectionProfiles]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea\inspectionProfiles\profiles_settings.xml" [label="profiles_settings.xml"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea\inspectionProfiles" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea\inspectionProfiles\profiles_settings.xml"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea\inspectionProfiles\Project_Default.xml" [label="Project_Default.xml"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea\inspectionProfiles" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea\inspectionProfiles\Project_Default.xml"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea\shelf" [label=shelf]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea\shelf\Changes" [label=Changes]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea\shelf" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea\shelf\Changes"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea\shelf\Changes.xml" [label="Changes.xml"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea\shelf" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea\shelf\Changes.xml"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea\shelf\Changes" [label=Changes]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea\shelf\Changes\shelved.patch" [label="shelved.patch"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea\shelf\Changes" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.idea\shelf\Changes\shelved.patch"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.pytest_cache" [label=".pytest_cache"]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.pytest_cache\v" [label=v]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.pytest_cache" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.pytest_cache\v"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.pytest_cache\.gitignore" [label=".gitignore"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.pytest_cache" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.pytest_cache\.gitignore"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.pytest_cache\CACHEDIR.TAG" [label="CACHEDIR.TAG"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.pytest_cache" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.pytest_cache\CACHEDIR.TAG"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.pytest_cache\README.md" [label="README.md"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.pytest_cache" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.pytest_cache\README.md"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.pytest_cache\v" [label=v]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.pytest_cache\v\cache" [label=cache]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.pytest_cache\v" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.pytest_cache\v\cache"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.pytest_cache\v\cache" [label=cache]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.pytest_cache\v\cache\lastfailed" [label=lastfailed]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.pytest_cache\v\cache" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.pytest_cache\v\cache\lastfailed"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.pytest_cache\v\cache\nodeids" [label=nodeids]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.pytest_cache\v\cache" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.pytest_cache\v\cache\nodeids"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.pytest_cache\v\cache\stepwise" [label=stepwise]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.pytest_cache\v\cache" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\.pytest_cache\v\cache\stepwise"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\configs" [label=configs]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\configs\config.yaml" [label="config.yaml"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\configs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\configs\config.yaml"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data" [label=data]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\excel" [label=excel]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\excel"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" [label=images_for_labeling]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\jason_labled" [label=jason_labled]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\jason_labled"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" [label=regex_outputs]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" [label=training]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation" [label=validation]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\combined_tasks.json" [label="combined_tasks.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\combined_tasks.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\layoutlm_data.jsonl" [label="layoutlm_data.jsonl"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\layoutlm_data.jsonl"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\predicted_1_21.png" [label="predicted_1_21.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\predicted_1_21.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\processed_labeled_data.csv" [label="processed_labeled_data.csv"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\processed_labeled_data.csv"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\excel" [label=excel]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\excel\customs_data.xlsx" [label="customs_data.xlsx"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\excel" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\excel\customs_data.xlsx"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\excel\آبتین تجارت1403پاییز.xlsx" [label="آبتین تجارت1403پاییز.xlsx"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\excel" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\excel\آبتین تجارت1403پاییز.xlsx"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\excel\افشار تجارت منطقه آزاد ماکو 1402.xlsx" [label="افشار تجارت منطقه آزاد ماکو 1402.xlsx"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\excel" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\excel\افشار تجارت منطقه آزاد ماکو 1402.xlsx"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\excel\افشار تجارت منطقه آزاد ماکو زمستان 1401.xlsx" [label="افشار تجارت منطقه آزاد ماکو زمستان 1401.xlsx"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\excel" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\excel\افشار تجارت منطقه آزاد ماکو زمستان 1401.xlsx"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\excel\دریا تجارت مهر شایگان پاییز.xlsx" [label="دریا تجارت مهر شایگان پاییز.xlsx"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\excel" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\excel\دریا تجارت مهر شایگان پاییز.xlsx"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\excel\ستاره الماس آرارات واردات پاییز (1).xlsx" [label="ستاره الماس آرارات واردات پاییز (1).xlsx"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\excel" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\excel\ستاره الماس آرارات واردات پاییز (1).xlsx"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\excel\واردات ارغوان گستر.xlsx" [label="واردات ارغوان گستر.xlsx"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\excel" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\excel\واردات ارغوان گستر.xlsx"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\excel\واردات تابستان 1403بلوری.xlsx" [label="واردات تابستان 1403بلوری.xlsx"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\excel" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\excel\واردات تابستان 1403بلوری.xlsx"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\excel\واردات تابستان 1403علیسان سپهر.xlsx" [label="واردات تابستان 1403علیسان سپهر.xlsx"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\excel" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\excel\واردات تابستان 1403علیسان سپهر.xlsx"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\excel\واردات خلج تجارت2.xlsx" [label="واردات خلج تجارت2.xlsx"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\excel" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\excel\واردات خلج تجارت2.xlsx"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\excel\واردات پارلا تجارت1403.xlsx" [label="واردات پارلا تجارت1403.xlsx"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\excel" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\excel\واردات پارلا تجارت1403.xlsx"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" [label=images_for_labeling]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\07.14.1_page_0.png" [label="07.14.1_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\07.14.1_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\07.16.1_page_0.png" [label="07.16.1_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\07.16.1_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1 (21)_page_0.png" [label="1 (21)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1 (21)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1 (22)_page_0.png" [label="1 (22)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1 (22)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\10 (4)_page_0.png" [label="10 (4)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\10 (4)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\10 (4)_page_1.png" [label="10 (4)_page_1.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\10 (4)_page_1.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\10 (5)_page_0.png" [label="10 (5)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\10 (5)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\10 (5)_page_1.png" [label="10 (5)_page_1.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\10 (5)_page_1.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.02.21_page_0.png" [label="1403.02.21_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.02.21_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.02.26d_page_0.png" [label="1403.02.26d_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.02.26d_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.03.01b (1)_page_0.png" [label="1403.03.01b (1)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.03.01b (1)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.03.01b (1)_page_1.png" [label="1403.03.01b (1)_page_1.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.03.01b (1)_page_1.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.03.01b_page_0.png" [label="1403.03.01b_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.03.01b_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.03.01b_page_1.png" [label="1403.03.01b_page_1.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.03.01b_page_1.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.03.01c_page_0.png" [label="1403.03.01c_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.03.01c_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.03.06_page_0.png" [label="1403.03.06_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.03.06_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.03.09_page_0.png" [label="1403.03.09_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.03.09_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.05.11_page_0.png" [label="1403.05.11_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.05.11_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.05.24_page_0.png" [label="1403.05.24_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.05.24_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.10_page_0.png" [label="1403.10.10_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.10_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.10_page_1.png" [label="1403.10.10_page_1.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.10_page_1.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.10_page_2.png" [label="1403.10.10_page_2.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.10_page_2.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.10_page_3.png" [label="1403.10.10_page_3.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.10_page_3.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.10_page_4.png" [label="1403.10.10_page_4.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.10_page_4.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.10_page_5.png" [label="1403.10.10_page_5.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.10_page_5.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.10_page_6.png" [label="1403.10.10_page_6.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.10_page_6.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.10_page_7.png" [label="1403.10.10_page_7.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.10_page_7.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.15_page_0.png" [label="1403.10.15_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.15_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.15_page_1.png" [label="1403.10.15_page_1.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.15_page_1.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.24_page_0.png" [label="1403.10.24_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.24_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.24_page_1.png" [label="1403.10.24_page_1.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.24_page_1.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.24_page_2.png" [label="1403.10.24_page_2.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.24_page_2.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.24_page_3.png" [label="1403.10.24_page_3.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.24_page_3.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.24_page_4.png" [label="1403.10.24_page_4.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.24_page_4.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.24_page_5.png" [label="1403.10.24_page_5.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.24_page_5.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.24_page_6.png" [label="1403.10.24_page_6.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.24_page_6.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.24_page_7.png" [label="1403.10.24_page_7.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.24_page_7.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.2_page_0.png" [label="1403.10.2_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.2_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.2_page_1.png" [label="1403.10.2_page_1.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.2_page_1.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.9_page_0.png" [label="1403.10.9_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.10.9_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.14_page_0.png" [label="1403.7.14_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.14_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.14_page_1.png" [label="1403.7.14_page_1.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.14_page_1.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.14_page_2.png" [label="1403.7.14_page_2.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.14_page_2.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.14_page_3.png" [label="1403.7.14_page_3.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.14_page_3.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.14_page_4.png" [label="1403.7.14_page_4.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.14_page_4.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.14_page_5.png" [label="1403.7.14_page_5.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.14_page_5.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.18_page_0.png" [label="1403.7.18_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.18_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.18_page_1.png" [label="1403.7.18_page_1.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.18_page_1.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.18_page_2.png" [label="1403.7.18_page_2.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.18_page_2.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.18_page_3.png" [label="1403.7.18_page_3.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.18_page_3.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.18_page_4.png" [label="1403.7.18_page_4.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.18_page_4.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.18_page_5.png" [label="1403.7.18_page_5.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.18_page_5.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.18_page_6.png" [label="1403.7.18_page_6.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.18_page_6.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.18_page_7.png" [label="1403.7.18_page_7.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.18_page_7.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.24_page_0.png" [label="1403.7.24_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.24_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.2_page_0.png" [label="1403.7.2_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.2_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.2_page_1.png" [label="1403.7.2_page_1.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.2_page_1.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.2_page_2.png" [label="1403.7.2_page_2.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.2_page_2.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.2_page_3.png" [label="1403.7.2_page_3.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.2_page_3.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.2_page_4.png" [label="1403.7.2_page_4.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.2_page_4.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.2_page_5.png" [label="1403.7.2_page_5.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.2_page_5.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.2_page_6.png" [label="1403.7.2_page_6.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.2_page_6.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.2_page_7.png" [label="1403.7.2_page_7.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.7.2_page_7.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.9.17_page_0.png" [label="1403.9.17_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.9.17_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.9.17_page_1.png" [label="1403.9.17_page_1.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.9.17_page_1.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.9.17_page_2.png" [label="1403.9.17_page_2.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.9.17_page_2.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.9.17_page_3.png" [label="1403.9.17_page_3.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.9.17_page_3.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.9.21_page_0.png" [label="1403.9.21_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.9.21_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.9.21_page_1.png" [label="1403.9.21_page_1.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.9.21_page_1.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.9.21_page_2.png" [label="1403.9.21_page_2.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.9.21_page_2.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.9.21_page_3.png" [label="1403.9.21_page_3.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.9.21_page_3.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.9.6_page_0.png" [label="1403.9.6_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.9.6_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.9.6_page_1.png" [label="1403.9.6_page_1.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.9.6_page_1.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.9.6_page_2.png" [label="1403.9.6_page_2.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.9.6_page_2.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.9.6_page_3.png" [label="1403.9.6_page_3.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.9.6_page_3.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.9.6_page_4.png" [label="1403.9.6_page_4.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.9.6_page_4.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.9.6_page_5.png" [label="1403.9.6_page_5.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.9.6_page_5.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.9.6_page_6.png" [label="1403.9.6_page_6.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\1403.9.6_page_6.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\3 (10)_page_0.png" [label="3 (10)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\3 (10)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\31809198_page_0.png" [label="31809198_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\31809198_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\31832672_page_0.png" [label="31832672_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\31832672_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\33770854_page_0.png" [label="33770854_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\33770854_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\33816609_page_0.png" [label="33816609_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\33816609_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\33875182_page_0.png" [label="33875182_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\33875182_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\33875182_page_1.png" [label="33875182_page_1.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\33875182_page_1.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\34072088_page_0.png" [label="34072088_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\34072088_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\34072088_page_1.png" [label="34072088_page_1.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\34072088_page_1.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\34072088_page_2.png" [label="34072088_page_2.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\34072088_page_2.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\34072088_page_3.png" [label="34072088_page_3.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\34072088_page_3.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\34585347_page_0.png" [label="34585347_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\34585347_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\34594553_page_0.png" [label="34594553_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\34594553_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\34614659_page_0.png" [label="34614659_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\34614659_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\34645882_page_0.png" [label="34645882_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\34645882_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\34670663_page_0.png" [label="34670663_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\34670663_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\34683243_page_0.png" [label="34683243_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\34683243_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\34683243_page_1.png" [label="34683243_page_1.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\34683243_page_1.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\34771952_page_0.png" [label="34771952_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\34771952_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\34781478_page_0.png" [label="34781478_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\34781478_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\34781478_page_1.png" [label="34781478_page_1.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\34781478_page_1.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\34781478_page_2.png" [label="34781478_page_2.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\34781478_page_2.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\34781478_page_3.png" [label="34781478_page_3.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\34781478_page_3.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\4 (10)_page_0.png" [label="4 (10)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\4 (10)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\4 (11)_page_0.png" [label="4 (11)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\4 (11)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\4 (12)_page_0.png" [label="4 (12)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\4 (12)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\4 (13)_page_0.png" [label="4 (13)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\4 (13)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\4 (18)_page_0.png" [label="4 (18)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\4 (18)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\4 (19)_page_0.png" [label="4 (19)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\4 (19)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\4 (20)_page_0.png" [label="4 (20)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\4 (20)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\4 (21)_page_0.png" [label="4 (21)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\4 (21)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\4.1 (5)_page_0.png" [label="4.1 (5)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\4.1 (5)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\5 (10)_page_0.png" [label="5 (10)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\5 (10)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\5 (11)_page_0.png" [label="5 (11)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\5 (11)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\5 (9)_page_0.png" [label="5 (9)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\5 (9)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\5.1 (2)_page_0.png" [label="5.1 (2)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\5.1 (2)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\5.1 (3)_page_0.png" [label="5.1 (3)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\5.1 (3)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\5.1 (4)_page_0.png" [label="5.1 (4)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\5.1 (4)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\5.1 (5)_page_0.png" [label="5.1 (5)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\5.1 (5)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\6 (6)_page_0.png" [label="6 (6)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\6 (6)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\6 (7)_page_0.png" [label="6 (7)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\6 (7)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\6 (8)_page_0.png" [label="6 (8)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\6 (8)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\6 (9)_page_0.png" [label="6 (9)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\6 (9)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\6.1 (2)_page_0.png" [label="6.1 (2)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\6.1 (2)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\6.1 (3)_page_0.png" [label="6.1 (3)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\6.1 (3)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\6.1 (4)_page_0.png" [label="6.1 (4)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\6.1 (4)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\6.1 (5)_page_0.png" [label="6.1 (5)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\6.1 (5)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\612_page_0.png" [label="612_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\612_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\613_page_0.png" [label="613_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\613_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\7 (6)_page_0.png" [label="7 (6)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\7 (6)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\7 (7)_page_0.png" [label="7 (7)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\7 (7)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\7 (8)_page_0.png" [label="7 (8)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\7 (8)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\7 (9)_page_0.png" [label="7 (9)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\7 (9)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\8 (5)_page_0.png" [label="8 (5)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\8 (5)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\8 (6)_page_0.png" [label="8 (6)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\8 (6)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\8 (7)_page_0.png" [label="8 (7)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\8 (7)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\8 (8)_page_0.png" [label="8 (8)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\8 (8)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\8.1 (2)_page_0.png" [label="8.1 (2)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\8.1 (2)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\8.1 (3)_page_0.png" [label="8.1 (3)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\8.1 (3)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\9 (5)_page_0.png" [label="9 (5)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\9 (5)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\9 (6)_page_0.png" [label="9 (6)_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\9 (6)_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\q200_page_0.png" [label="q200_page_0.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\q200_page_0.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\q200_page_1.png" [label="q200_page_1.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\q200_page_1.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\q200_page_2.png" [label="q200_page_2.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\images_for_labeling\q200_page_2.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\jason_labled" [label=jason_labled]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\jason_labled\project-5-at-2025-04-18-09-36-514f7a4b.json" [label="project-5-at-2025-04-18-09-36-514f7a4b.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\jason_labled" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\jason_labled\project-5-at-2025-04-18-09-36-514f7a4b.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" [label=regex_outputs]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\07.14.1.json" [label="07.14.1.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\07.14.1.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\07.16.1.json" [label="07.16.1.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\07.16.1.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\1 (21)_page_0.json" [label="1 (21)_page_0.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\1 (21)_page_0.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\1 (22).json" [label="1 (22).json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\1 (22).json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\10 (4).json" [label="10 (4).json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\10 (4).json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\10 (5).json" [label="10 (5).json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\10 (5).json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\1403.05.11.json" [label="1403.05.11.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\1403.05.11.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\1403.05.24.json" [label="1403.05.24.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\1403.05.24.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\1403.10.10.json" [label="1403.10.10.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\1403.10.10.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\1403.10.15.json" [label="1403.10.15.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\1403.10.15.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\1403.10.2.json" [label="1403.10.2.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\1403.10.2.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\1403.10.24.json" [label="1403.10.24.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\1403.10.24.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\1403.10.9.json" [label="1403.10.9.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\1403.10.9.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\1403.7.14.json" [label="1403.7.14.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\1403.7.14.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\1403.7.18.json" [label="1403.7.18.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\1403.7.18.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\1403.7.2.json" [label="1403.7.2.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\1403.7.2.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\1403.7.24.json" [label="1403.7.24.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\1403.7.24.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\1403.9.17.json" [label="1403.9.17.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\1403.9.17.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\1403.9.21.json" [label="1403.9.21.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\1403.9.21.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\1403.9.6.json" [label="1403.9.6.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\1403.9.6.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\3 (10).json" [label="3 (10).json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\3 (10).json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\31809198.json" [label="31809198.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\31809198.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\31832672.json" [label="31832672.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\31832672.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\33770854.json" [label="33770854.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\33770854.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\33816609.json" [label="33816609.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\33816609.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\33875182.json" [label="33875182.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\33875182.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\34072088.json" [label="34072088.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\34072088.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\34585347.json" [label="34585347.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\34585347.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\34594553.json" [label="34594553.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\34594553.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\34614659.json" [label="34614659.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\34614659.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\34645882.json" [label="34645882.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\34645882.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\34670663.json" [label="34670663.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\34670663.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\34683243.json" [label="34683243.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\34683243.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\34771952.json" [label="34771952.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\34771952.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\34781478.json" [label="34781478.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\34781478.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\4 (11).json" [label="4 (11).json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\4 (11).json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\4 (12).json" [label="4 (12).json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\4 (12).json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\4 (13).json" [label="4 (13).json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\4 (13).json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\4.1 (5).json" [label="4.1 (5).json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\4.1 (5).json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\5 (10).json" [label="5 (10).json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\5 (10).json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\5 (11).json" [label="5 (11).json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\5 (11).json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\5.1 (2).json" [label="5.1 (2).json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\5.1 (2).json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\5.1 (3).json" [label="5.1 (3).json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\5.1 (3).json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\5.1 (4).json" [label="5.1 (4).json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\5.1 (4).json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\5.1 (5).json" [label="5.1 (5).json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\5.1 (5).json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\6 (8).json" [label="6 (8).json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\6 (8).json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\6 (9).json" [label="6 (9).json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\6 (9).json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\6.1 (2).json" [label="6.1 (2).json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\6.1 (2).json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\6.1 (3).json" [label="6.1 (3).json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\6.1 (3).json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\6.1 (4).json" [label="6.1 (4).json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\6.1 (4).json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\6.1 (5).json" [label="6.1 (5).json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\6.1 (5).json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\7 (8).json" [label="7 (8).json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\7 (8).json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\7 (9).json" [label="7 (9).json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\7 (9).json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\8 (7).json" [label="8 (7).json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\8 (7).json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\8 (8).json" [label="8 (8).json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\8 (8).json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\8.1 (2).json" [label="8.1 (2).json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\8.1 (2).json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\8.1 (3).json" [label="8.1 (3).json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\8.1 (3).json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\9 (5).json" [label="9 (5).json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\9 (5).json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\9 (6).json" [label="9 (6).json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\regex_outputs\9 (6).json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" [label=training]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\07.14.1.pdf" [label="07.14.1.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\07.14.1.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\07.16.1.pdf" [label="07.16.1.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\07.16.1.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1 (21).pdf" [label="1 (21).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1 (21).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1 (22).pdf" [label="1 (22).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1 (22).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\10 (4).pdf" [label="10 (4).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\10 (4).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\10 (5).pdf" [label="10 (5).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\10 (5).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.02.21.pdf" [label="1403.02.21.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.02.21.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.02.26d.pdf" [label="1403.02.26d.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.02.26d.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.03.01b (1).pdf" [label="1403.03.01b (1).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.03.01b (1).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.03.01b.pdf" [label="1403.03.01b.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.03.01b.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.03.01c.pdf" [label="1403.03.01c.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.03.01c.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.03.06.pdf" [label="1403.03.06.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.03.06.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.03.09.pdf" [label="1403.03.09.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.03.09.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.05.11.pdf" [label="1403.05.11.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.05.11.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.05.24.pdf" [label="1403.05.24.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.05.24.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.10.10.pdf" [label="1403.10.10.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.10.10.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.10.15.pdf" [label="1403.10.15.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.10.15.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.10.2.pdf" [label="1403.10.2.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.10.2.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.10.24.pdf" [label="1403.10.24.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.10.24.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.10.9.pdf" [label="1403.10.9.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.10.9.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.7.14.pdf" [label="1403.7.14.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.7.14.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.7.18.pdf" [label="1403.7.18.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.7.18.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.7.2.pdf" [label="1403.7.2.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.7.2.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.7.24.pdf" [label="1403.7.24.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.7.24.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.9.17.pdf" [label="1403.9.17.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.9.17.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.9.21.pdf" [label="1403.9.21.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.9.21.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.9.6.pdf" [label="1403.9.6.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\1403.9.6.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\3 (10).pdf" [label="3 (10).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\3 (10).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\31809198.pdf" [label="31809198.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\31809198.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\31832672.pdf" [label="31832672.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\31832672.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\33770854.pdf" [label="33770854.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\33770854.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\33816609.pdf" [label="33816609.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\33816609.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\33875182.pdf" [label="33875182.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\33875182.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\34072088.pdf" [label="34072088.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\34072088.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\34585347.pdf" [label="34585347.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\34585347.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\34594553.pdf" [label="34594553.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\34594553.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\34614659.pdf" [label="34614659.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\34614659.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\34645882.pdf" [label="34645882.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\34645882.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\34670663.pdf" [label="34670663.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\34670663.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\34683243.pdf" [label="34683243.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\34683243.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\34771952.pdf" [label="34771952.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\34771952.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\34781478.pdf" [label="34781478.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\34781478.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\4 (10).pdf" [label="4 (10).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\4 (10).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\4 (11).pdf" [label="4 (11).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\4 (11).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\4 (12).pdf" [label="4 (12).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\4 (12).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\4 (13).pdf" [label="4 (13).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\4 (13).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\4 (18).pdf" [label="4 (18).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\4 (18).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\4 (19).pdf" [label="4 (19).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\4 (19).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\4 (20).pdf" [label="4 (20).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\4 (20).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\4 (21).pdf" [label="4 (21).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\4 (21).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\4.1 (5).pdf" [label="4.1 (5).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\4.1 (5).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\5 (10).pdf" [label="5 (10).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\5 (10).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\5 (11).pdf" [label="5 (11).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\5 (11).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\5 (9).pdf" [label="5 (9).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\5 (9).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\5.1 (2).pdf" [label="5.1 (2).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\5.1 (2).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\5.1 (3).pdf" [label="5.1 (3).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\5.1 (3).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\5.1 (4).pdf" [label="5.1 (4).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\5.1 (4).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\5.1 (5).pdf" [label="5.1 (5).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\5.1 (5).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\6 (6).pdf" [label="6 (6).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\6 (6).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\6 (7).pdf" [label="6 (7).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\6 (7).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\6 (8).pdf" [label="6 (8).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\6 (8).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\6 (9).pdf" [label="6 (9).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\6 (9).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\6.1 (2).pdf" [label="6.1 (2).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\6.1 (2).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\6.1 (3).pdf" [label="6.1 (3).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\6.1 (3).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\6.1 (4).pdf" [label="6.1 (4).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\6.1 (4).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\6.1 (5).pdf" [label="6.1 (5).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\6.1 (5).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\612.pdf" [label="612.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\612.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\613.pdf" [label="613.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\613.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\7 (6).pdf" [label="7 (6).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\7 (6).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\7 (7).pdf" [label="7 (7).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\7 (7).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\7 (8).pdf" [label="7 (8).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\7 (8).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\7 (9).pdf" [label="7 (9).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\7 (9).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\8 (5).pdf" [label="8 (5).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\8 (5).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\8 (6).pdf" [label="8 (6).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\8 (6).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\8 (7).pdf" [label="8 (7).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\8 (7).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\8 (8).pdf" [label="8 (8).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\8 (8).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\8.1 (2).pdf" [label="8.1 (2).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\8.1 (2).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\8.1 (3).pdf" [label="8.1 (3).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\8.1 (3).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\9 (5).pdf" [label="9 (5).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\9 (5).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\9 (6).pdf" [label="9 (6).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\9 (6).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\q200.pdf" [label="q200.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\training\q200.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation" [label=validation]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\1 (23).pdf" [label="1 (23).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\1 (23).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\1.1 (2).pdf" [label="1.1 (2).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\1.1 (2).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\1.1 (4).pdf" [label="1.1 (4).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\1.1 (4).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\1.1 (5).pdf" [label="1.1 (5).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\1.1 (5).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\2 (14).pdf" [label="2 (14).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\2 (14).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\2 (15).pdf" [label="2 (15).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\2 (15).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\2 (16).pdf" [label="2 (16).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\2 (16).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\2.1 (3).pdf" [label="2.1 (3).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\2.1 (3).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\2.1 (4).pdf" [label="2.1 (4).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\2.1 (4).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\2.1 (5).pdf" [label="2.1 (5).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\2.1 (5).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\2.1 (6).pdf" [label="2.1 (6).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\2.1 (6).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\3 (9).pdf" [label="3 (9).pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\3 (9).pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\340580938.pdf" [label="340580938.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\340580938.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\34794682.pdf" [label="34794682.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\34794682.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\34878714.pdf" [label="34878714.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\34878714.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\34884981.pdf" [label="34884981.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\34884981.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\example.pdf" [label="example.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\example.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\test_document.pdf" [label="test_document.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\test_document.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\test_document2.pdf" [label="test_document2.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\test_document2.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\کوتاژ1403.7.5.pdf" [label="کوتاژ1403.7.5.pdf"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\data\validation\کوتاژ1403.7.5.pdf"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model" [label=layoutlm_farsi_model]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-100" [label="checkpoint-100"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-100"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-135" [label="checkpoint-135"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-135"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-100" [label="checkpoint-100"]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-100\config.json" [label="config.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-100" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-100\config.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-100\model.safetensors" [label="model.safetensors"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-100" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-100\model.safetensors"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-100\optimizer.pt" [label="optimizer.pt"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-100" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-100\optimizer.pt"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-100\rng_state.pth" [label="rng_state.pth"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-100" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-100\rng_state.pth"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-100\scheduler.pt" [label="scheduler.pt"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-100" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-100\scheduler.pt"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-100\special_tokens_map.json" [label="special_tokens_map.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-100" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-100\special_tokens_map.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-100\tokenizer.json" [label="tokenizer.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-100" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-100\tokenizer.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-100\tokenizer_config.json" [label="tokenizer_config.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-100" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-100\tokenizer_config.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-100\trainer_state.json" [label="trainer_state.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-100" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-100\trainer_state.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-100\training_args.bin" [label="training_args.bin"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-100" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-100\training_args.bin"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-100\vocab.txt" [label="vocab.txt"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-100" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-100\vocab.txt"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-135" [label="checkpoint-135"]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-135\config.json" [label="config.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-135" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-135\config.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-135\model.safetensors" [label="model.safetensors"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-135" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-135\model.safetensors"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-135\optimizer.pt" [label="optimizer.pt"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-135" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-135\optimizer.pt"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-135\rng_state.pth" [label="rng_state.pth"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-135" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-135\rng_state.pth"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-135\scheduler.pt" [label="scheduler.pt"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-135" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-135\scheduler.pt"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-135\special_tokens_map.json" [label="special_tokens_map.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-135" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-135\special_tokens_map.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-135\tokenizer.json" [label="tokenizer.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-135" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-135\tokenizer.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-135\tokenizer_config.json" [label="tokenizer_config.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-135" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-135\tokenizer_config.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-135\trainer_state.json" [label="trainer_state.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-135" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-135\trainer_state.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-135\training_args.bin" [label="training_args.bin"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-135" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-135\training_args.bin"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-135\vocab.txt" [label="vocab.txt"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-135" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model\checkpoint-135\vocab.txt"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model_final" [label=layoutlm_farsi_model_final]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model_final\config.json" [label="config.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model_final" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model_final\config.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model_final\model.safetensors" [label="model.safetensors"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model_final" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model_final\model.safetensors"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model_final\special_tokens_map.json" [label="special_tokens_map.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model_final" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model_final\special_tokens_map.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model_final\tokenizer.json" [label="tokenizer.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model_final" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model_final\tokenizer.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model_final\tokenizer_config.json" [label="tokenizer_config.json"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model_final" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model_final\tokenizer_config.json"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model_final\vocab.txt" [label="vocab.txt"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model_final" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\layoutlm_farsi_model_final\vocab.txt"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" [label=logs]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\evaluation_results" [label=evaluation_results]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\evaluation_results"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\predictions" [label=predictions]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\predictions"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\loss_history.png" [label="loss_history.png"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\loss_history.png"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250411_213125.log" [label="training_20250411_213125.log"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250411_213125.log"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250411_215105.log" [label="training_20250411_215105.log"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250411_215105.log"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250411_215347.log" [label="training_20250411_215347.log"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250411_215347.log"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250411_215737.log" [label="training_20250411_215737.log"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250411_215737.log"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250411_215938.log" [label="training_20250411_215938.log"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250411_215938.log"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250411_220334.log" [label="training_20250411_220334.log"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250411_220334.log"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250411_221133.log" [label="training_20250411_221133.log"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250411_221133.log"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250411_221742.log" [label="training_20250411_221742.log"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250411_221742.log"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250411_222355.log" [label="training_20250411_222355.log"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250411_222355.log"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250411_223436.log" [label="training_20250411_223436.log"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250411_223436.log"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250411_232506.log" [label="training_20250411_232506.log"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250411_232506.log"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250411_232800.log" [label="training_20250411_232800.log"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250411_232800.log"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250411_233917.log" [label="training_20250411_233917.log"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250411_233917.log"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250411_234304.log" [label="training_20250411_234304.log"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250411_234304.log"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250412_000656.log" [label="training_20250412_000656.log"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250412_000656.log"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250412_000845.log" [label="training_20250412_000845.log"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250412_000845.log"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250413_210043.log" [label="training_20250413_210043.log"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250413_210043.log"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250413_210211.log" [label="training_20250413_210211.log"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250413_210211.log"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250413_210613.log" [label="training_20250413_210613.log"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250413_210613.log"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250413_211003.log" [label="training_20250413_211003.log"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250413_211003.log"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250413_211302.log" [label="training_20250413_211302.log"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250413_211302.log"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250413_211734.log" [label="training_20250413_211734.log"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250413_211734.log"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250413_213055.log" [label="training_20250413_213055.log"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250413_213055.log"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250413_213256.log" [label="training_20250413_213256.log"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250413_213256.log"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250413_214149.log" [label="training_20250413_214149.log"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250413_214149.log"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250413_214809.log" [label="training_20250413_214809.log"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250413_214809.log"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250413_215214.log" [label="training_20250413_215214.log"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250413_215214.log"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250413_215533.log" [label="training_20250413_215533.log"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250413_215533.log"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250413_220738.log" [label="training_20250413_220738.log"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250413_220738.log"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250413_220833.log" [label="training_20250413_220833.log"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250413_220833.log"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250413_221545.log" [label="training_20250413_221545.log"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250413_221545.log"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250413_221707.log" [label="training_20250413_221707.log"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250413_221707.log"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250413_223335.log" [label="training_20250413_223335.log"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\training_20250413_223335.log"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\evaluation_results" [label=evaluation_results]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\evaluation_results\evaluation_results_20250413_211740.csv" [label="evaluation_results_20250413_211740.csv"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\evaluation_results" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\evaluation_results\evaluation_results_20250413_211740.csv"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\predictions" [label=predictions]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\predictions\prediction_results_20250413_221556.csv" [label="prediction_results_20250413_221556.csv"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\predictions" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\predictions\prediction_results_20250413_221556.csv"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\predictions\prediction_results_20250413_221726.csv" [label="prediction_results_20250413_221726.csv"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\predictions" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\logs\predictions\prediction_results_20250413_221726.csv"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src" [label=src]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\data" [label=data]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\data"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\models" [label=models]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\models"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\utils" [label=utils]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\utils"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\__pycache__" [label=__pycache__]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\__pycache__"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\__init__.py" [label="__init__.py"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\__init__.py"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\data" [label=data]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\data\__pycache__" [label=__pycache__]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\data" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\data\__pycache__"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\data\data_loader.py" [label="data_loader.py"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\data" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\data\data_loader.py"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\data\excel_processor.py" [label="excel_processor.py"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\data" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\data\excel_processor.py"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\data\preprocessor.py" [label="preprocessor.py"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\data" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\data\preprocessor.py"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\data\__init__.py" [label="__init__.py"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\data" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\data\__init__.py"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\data\__pycache__" [label=__pycache__]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\data\__pycache__\data_loader.cpython-312.pyc" [label="data_loader.cpython-312.pyc"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\data\__pycache__" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\data\__pycache__\data_loader.cpython-312.pyc"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\data\__pycache__\excel_processor.cpython-312.pyc" [label="excel_processor.cpython-312.pyc"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\data\__pycache__" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\data\__pycache__\excel_processor.cpython-312.pyc"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\data\__pycache__\preprocessor.cpython-312.pyc" [label="preprocessor.cpython-312.pyc"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\data\__pycache__" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\data\__pycache__\preprocessor.cpython-312.pyc"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\data\__pycache__\__init__.cpython-312.pyc" [label="__init__.cpython-312.pyc"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\data\__pycache__" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\data\__pycache__\__init__.cpython-312.pyc"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\models" [label=models]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\models\__pycache__" [label=__pycache__]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\models" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\models\__pycache__"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\models\document_classifier.py" [label="document_classifier.py"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\models" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\models\document_classifier.py"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\models\layers.py" [label="layers.py"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\models" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\models\layers.py"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\models\__init__.py" [label="__init__.py"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\models" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\models\__init__.py"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\models\__pycache__" [label=__pycache__]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\models\__pycache__\document_classifier.cpython-312.pyc" [label="document_classifier.cpython-312.pyc"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\models\__pycache__" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\models\__pycache__\document_classifier.cpython-312.pyc"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\models\__pycache__\__init__.cpython-312.pyc" [label="__init__.cpython-312.pyc"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\models\__pycache__" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\models\__pycache__\__init__.cpython-312.pyc"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\utils" [label=utils]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\utils\__pycache__" [label=__pycache__]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\utils" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\utils\__pycache__"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\utils\logger.py" [label="logger.py"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\utils" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\utils\logger.py"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\utils\visualization.py" [label="visualization.py"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\utils" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\utils\visualization.py"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\utils\__init__.py" [label="__init__.py"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\utils" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\utils\__init__.py"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\utils\__pycache__" [label=__pycache__]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\utils\__pycache__\logger.cpython-312.pyc" [label="logger.cpython-312.pyc"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\utils\__pycache__" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\utils\__pycache__\logger.cpython-312.pyc"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\utils\__pycache__\visualization.cpython-312.pyc" [label="visualization.cpython-312.pyc"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\utils\__pycache__" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\utils\__pycache__\visualization.cpython-312.pyc"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\utils\__pycache__\__init__.cpython-312.pyc" [label="__init__.cpython-312.pyc"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\utils\__pycache__" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\utils\__pycache__\__init__.cpython-312.pyc"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\__pycache__" [label=__pycache__]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\__pycache__\__init__.cpython-312.pyc" [label="__init__.cpython-312.pyc"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\__pycache__" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\src\__pycache__\__init__.cpython-312.pyc"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests" [label=tests]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\.pytest_cache" [label=".pytest_cache"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\.pytest_cache"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\__pycache__" [label=__pycache__]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\__pycache__"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\dump.txt" [label="dump.txt"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\dump.txt"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\dump_pdf_text.py" [label="dump_pdf_text.py"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\dump_pdf_text.py"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\test_parser.py" [label="test_parser.py"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\test_parser.py"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\.pytest_cache" [label=".pytest_cache"]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\.pytest_cache\v" [label=v]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\.pytest_cache" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\.pytest_cache\v"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\.pytest_cache\.gitignore" [label=".gitignore"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\.pytest_cache" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\.pytest_cache\.gitignore"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\.pytest_cache\CACHEDIR.TAG" [label="CACHEDIR.TAG"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\.pytest_cache" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\.pytest_cache\CACHEDIR.TAG"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\.pytest_cache\README.md" [label="README.md"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\.pytest_cache" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\.pytest_cache\README.md"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\.pytest_cache\v" [label=v]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\.pytest_cache\v\cache" [label=cache]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\.pytest_cache\v" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\.pytest_cache\v\cache"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\.pytest_cache\v\cache" [label=cache]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\.pytest_cache\v\cache\lastfailed" [label=lastfailed]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\.pytest_cache\v\cache" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\.pytest_cache\v\cache\lastfailed"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\.pytest_cache\v\cache\nodeids" [label=nodeids]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\.pytest_cache\v\cache" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\.pytest_cache\v\cache\nodeids"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\.pytest_cache\v\cache\stepwise" [label=stepwise]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\.pytest_cache\v\cache" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\.pytest_cache\v\cache\stepwise"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\__pycache__" [label=__pycache__]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\__pycache__\dump_pdf_text.cpython-312-pytest-8.3.5.pyc" [label="dump_pdf_text.cpython-312-pytest-8.3.5.pyc"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\__pycache__" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\__pycache__\dump_pdf_text.cpython-312-pytest-8.3.5.pyc"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\__pycache__\test_parser.cpython-312-pytest-8.3.5.pyc" [label="test_parser.cpython-312-pytest-8.3.5.pyc"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\__pycache__" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\tests\__pycache__\test_parser.cpython-312-pytest-8.3.5.pyc"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\trained_models" [label=trained_models]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\trained_models\customs_document_classifier.h5" [label="customs_document_classifier.h5"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\trained_models" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\trained_models\customs_document_classifier.h5"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\trained_models\tokenizer.pkl" [label="tokenizer.pkl"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\trained_models" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\trained_models\tokenizer.pkl"
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\__pycache__" [label=__pycache__]
	"C:\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\__pycache__\test_gpu.cpython-312-pytest-8.3.5.pyc" [label="test_gpu.cpython-312-pytest-8.3.5.pyc"]
	C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\__pycache__" -> C:"\Users\Mohsen\PycharmProjects\PythonProject1\pdf_extracter(test)\__pycache__\test_gpu.cpython-312-pytest-8.3.5.pyc"
}

حالا بگو به چه فایلی نیاز داریم عشقم برای ادامه پروژه چیا باید حذف بشن؟ چیارو باز کنم ببینی؟
