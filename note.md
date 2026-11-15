# Otel rezervasiya datasında mövsümi qanunauyğunluqlar və ləğv analizi

## 1. Metodologiya

**Dataset.** Hotel Booking Demand (Kaggle, jessemostipak), `hotel_bookings.csv`. Tapşırıqda təklif olunan dataset istifadə edilib (başqa datasetlə əvəz olunmayıb). Data 119,390 sətir və 32 sütundan ibarətdir: City Hotel (79,330 rezervasiya) və Resort Hotel (40,060 rezervasiya). Gəliş tarixləri 2015-07-01 ilə 2017-08-31 arasındadır.

**Alətlər.** Analiz tapşırıqda təklif olunan pandas / matplotlib / seaborn əvəzinə R dilində (tidyverse, lubridate, scales, ggplot2) icra olunub. Təhvil: R Markdown notebook (`hotel_bookings_analysis.Rmd` və onun knit olunmuş HTML versiyası).

**Datanın təmizlənməsi.**
- `agent`, `company` və `country` sütunlarındakı boşluqlar faylda "NULL" yazısı kimi saxlanılıb. Fayl oxunarkən bu yazılar NA-ya çevrilib (müvafiq olaraq 16,340 / 112,593 / 488 sətir), sonra `country` "Unknown", `agent` və `company` isə 0 ilə doldurulub.
- `children` sütununda 4 NA var idi (tapşırıqda qeyd olunmayıb), 0 ilə doldurulub.
- Doldurmadan sonra heç bir NA qalmayıb.

**Tarix sütunu.** `arrival_date` sütunu `arrival_date_year`, `arrival_date_month` (ay adı rəqəmə çevrilib) və `arrival_date_day_of_month` sütunlarından yaradılıb. Ay sütunu təqvim sırasında factor-a çevrilib ki, cədvəl və qrafiklərdə aylar əlifba sırası ilə düzülməsin.

**Analiz addımları.**
- Pivot cədvəl: sətir = gəliş ayı, sütun = hotel, dəyər = rezervasiya sayı; heatmap kimi vizuallaşdırılıb.
- Ləğv nisbəti = `is_canceled`-in ortalaması; ay və hotel, ölkə, market segment və deposit tipi üzrə qruplaşdırılıb.
- Ləğv edilmiş və edilməmiş rezervasiyalar üzrə lead_time müqayisəsi (orta və median).
- Ən çox rezervasiyalı 10 ölkə və onların ləğv nisbəti.
- Market segment üzrə bölgü (rezervasiya, ləğv, pay); pie qrafiki ilə göstərilib.
- Bonus: ay və hotel üzrə orta gündəlik qiymət (ADR) dual-axis qrafikdə; ən aşağı ləğv nisbətli market segment.

**Normallaşdırma.** İyul və Avqust datada 3 ildə (2015-2017), digər aylar isə 2 ildə mövcuddur. Ona görə xam aylıq saylar İyul və Avqustu şişirdir. Hər ay üçün il başına orta rezervasiya sayı hesablanıb və aylar arasında müqayisədə istifadə olunub. Nəticələr həm də 2016 ili ilə yoxlanılıb (yeganə tam təqvim ili).

**ADR.** Yalnız ləğv edilməmiş və 0 < adr < 1000 şərtini ödəyən rezervasiyalar götürülüb. `adr <= 0` olan 1,960 sətir və bir ekstremal dəyər (5,400) kənarlaşdırılıb.

## 2. Əsas nəticələr

| Göstərici | Nəticə |
|---|---|
| Ləğv nisbəti | City Hotel 41.7%, Resort Hotel 27.8% |
| Orta lead_time | Ləğv edilənlər 144.8 gün, edilməyənlər 80.0 gün (median 113 vs 45) |
| Hotel üzrə orta lead_time | City 109.7 gün, Resort 92.7 gün |
| Ən çox rezervasiyalı ölkə | PRT: 48,590 rezervasiya (ümumi sayın 40.7%-i), 56.6% ləğv |
| Top 10 ölkə | Bütün rezervasiyaların təxminən 84%-i |
| Ən böyük market segment | Online TA: rezervasiyaların 47.3%-i, 36.7% ləğv |
| Ən yüksək ləğv nisbətli segment | Groups: 61.1% ləğv |
| Ən aşağı ləğv nisbətli segment (bonus) | Complementary 13.1% (743 rezervasiya); böyük segmentlər arasında Direct 15.3% (12,606 rezervasiya) |
| Müştəri tipi | Transient: rezervasiyaların 75.1%-i |

**Mövsümilik.**
- Xam saylarda ən yüksək ay Avqust (13,877 rezervasiya), ən aşağı ay Yanvardır (5,929). Lakin bu nəticə İyul və Avqust üçün əlavə ilin datası ilə şişirdilib.
- Normallaşdırmadan sonra (il başına orta rezervasiya) ən yüksək ay May (hər iki hotel üzrə təxminən 5,900), ən aşağı ay Yanvardır (təxminən 3,000).
- City Hotel-də tələbat Apreldən Oktyabra qədər yüksəkdir. Resort Hotel-in həcmi Aprel-Oktyabr arasında demək olar sabitdir (ayda təxminən 1,500-1,800), lakin ADR Yanvardakı 50.2-dən Avqustda 183.7-yə qalxır.
- Aylıq ləğv nisbəti: City Hotel il boyu 36.9% ilə 46.3% arasında sabit qalır; Resort Hotel-də isə Yanvarda 14.8%, Avqustda 33.4%-dir.

## 3. Revenue management üçün insight-lar

**1. Resort Hotel-də mövsümilik həcmdə yox, qiymətdədir.**
*Müşahidə:* ADR Yanvar və Noyabrda təxminən 50-dən Avqustda 184-ə qalxır, il başına orta rezervasiya sayı isə Aprel-Oktyabr arasında ayda təxminən 1,500-1,800 səviyyəsində qalır.
*Tövsiyə:* İyun-Sentyabr aylarında dinamik qiymət tətbiq et və erkən endirimlərdən çəkin; Yanvar və Noyabr-Dekabrda paketlər və aksiyalarla həcmi artır.

**2. Uzun lead_time yüksək ləğv riskinin siqnalıdır.**
*Müşahidə:* Ləğv edilən rezervasiyalar orta hesabla 145 gün öncədən, saxlanılanlar isə 80 gün öncədən edilib. City Hotel daha çox ləğv edir (41.7%).
*Tövsiyə:* Overbooking buferini lead_time aralığına və otelə görə təyin et (City Hotel üçün daha böyük), 100+ gün öncədən edilən rezervasiyalar üçün yenidən təsdiq və ya qismən ödənişi sına.

**3. Non Refund rezervasiyaları gözlənilməz davranır.**
*Müşahidə:* 14,587 rezervasiya (12.2%) Non Refund-dur və onların 99.4%-i ləğv edilib; 97%-i Portekizdən gəlir. No Deposit rezervasiyalar daxilində belə Portekizin ləğv nisbəti (38.9%) digər ölkələrdən (23.2%) yüksəkdir. Groups rezervasiyalarının 46.3%-i Non Refund-dur.
*Tövsiyə:* Non Refund-u ləğvdən qoruyucu hesab etmə; bu rezervasiyaları ayrıca proqnozlaşdır və bu cür ləğvlərin datada necə qeyd olunduğunu data sahibi ilə dəqiqləşdir. Səbəb hipotezdir (məsələn, datanın qeydə alınma qaydası və ya kanal təsiri); bu analiz səbəbi sübut etmir.

**4. Kanal strukturu.**
*Müşahidə:* Direct (15.3%) və Corporate (18.7%) ən etibarlı kanallardır. Online TA ən çox həcmi gətirir (rezervasiyaların 47.3%-i, bütün ləğvlərin 46.9%-i) və orta ləğv nisbətinə malikdir (36.7%). Groups-un ləğv nisbəti 61.1%-dir, lakin onların No Deposit rezervasiyalarında cəmi 28.3%-dir, yəni yüksək rəqəm əsasən Non Refund qrup rezervasiyalarından gəlir.
*Tövsiyə:* Direct və Corporate kanallarını gücləndir, qrup rezervasiyalarının müqavilə şərtlərini yenidən nəzərdən keçir.

## 4. Məhdudiyyətlər

- Data cəmi 26 ayı əhatə edir; 2015 qismən ildir və City Hotel üçün aşağı həcmlidir, ona görə 2015-i əhatə edən aylar üzrə ortalamalar aşağı çəkilir.
- Ləğv nisbətləri və ADR illər üzrə orta dəyərlərdir və zaman ərzində qiymət artımını gizlədə bilər.
- Tapıntılar korrelyasiyadır, səbəb-nəticə sübutu deyil.
