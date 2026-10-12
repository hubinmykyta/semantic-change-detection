# EPS 2026 · Change Detection

Навчальний дослідницький проєкт для виявлення та класифікації змін на геометрично вирівняних аеро- й супутникових знімках «До» / «Після». Модель **BIT-CD** має дві голови: наявність зміни та її тип.

**Автори:** [@hubinmykyta](https://github.com/hubinmykyta), [@o-haid](https://github.com/o-haid), [@anbar-root](https://github.com/anbar-root).

| Клас | Джерело навчальної розмітки |
|---|---|
| Зруйнований / знесений об'єкт | xBD Global; FOTBCD-Instances `DEMOLISHED` |
| Нова вирва | Синтетичний генератор |
| Свіжа фортифікація | Синтетичний генератор |
| Нова забудова | FOTBCD-Instances `NEW` |

Результат: маска класів, карта ймовірності, зображення з рамками та список об'єктів із класом, площею й оцінкою впевненості. Gradio-демо дозволяє завантажити результати як PNG і JSON.

> Вирви й фортифікації навчаються та оцінюються на синтетиці. Це не підтверджує якість на реальних знімках цих об'єктів. На xBD-Ukraine якість низька; рішення є дослідницьким прототипом.

## Файли проєкту

| Файл | Призначення |
|---|---|
| [EPS_ChangeDetection_BITCD_Semantic_UA.ipynb](EPS_ChangeDetection_BITCD_Semantic_UA.ipynb) | Підготовка даних, синтетика, аудит, навчання, validation / test та звіт |
| [EPS_ChangeDetection_Demo_Gradio_UA.ipynb](EPS_ChangeDetection_Demo_Gradio_UA.ipynb) | Інтерактивне демо з готовою моделлю |
| [requirements.txt](requirements.txt) | Залежності для локального запуску |
| [.gitignore](.gitignore) | Виключення ваг, датасетів, кешів і локального середовища з Git |
| README.md | Опис, запуск, результати та джерела |

Ноутбуки збережено без outputs для зручного перегляду змін. Збережений звіт наведено нижче.

## Швидкий запуск демо в Colab

1. Відкрийте [Google Colab](https://colab.research.google.com/) та завантажте `EPS_ChangeDetection_Demo_Gradio_UA.ipynb` через **File → Open notebook → Upload** або відкрийте його через вкладку **GitHub** за посиланням на репозиторій.
2. Виберіть **Runtime → Change runtime type → T4 GPU**.
3. Залиште `WEIGHTS_SOURCE='shared'` у першій комірці та виконайте **Run all**.
4. У Gradio завантажте пару «До» / «Після» та натисніть **«Знайти зміни»**. За наявності вкажіть GSD у м/піксель.

Модель і поріг завантажуються автоматично з Google Drive, перевіряються за SHA-256 та кешуються. Особистий Drive, навчальні дані й повторне навчання для демо не потрібні.

| Файл на Drive | Призначення |
|---|---|
| [best.pt](https://drive.google.com/file/d/1jtceZDX4cNY9udOLS_6zsN8ptSNVQozj/view) | Навчена модель із головою типів, епоха `29` |
| [threshold.json](https://drive.google.com/file/d/1RAjId11DcGvu1hq0K4JN4hJhDT32R1Bs/view) | Поріг `0.31` для цієї моделі |

[Спільна папка артефактів](https://drive.google.com/drive/folders/1zWs2m9bG5eTe_NGnyp4ElNFQKbk0Ah4w). Посилання й контрольні суми записані в `SHARED_FILES` усередині кожного ноутбука; сторонні файли з репозиторію для їх запуску не потрібні.

Для **власної моделі** встановіть `WEIGHTS_SOURCE='drive'`. У Colab підключиться особистий Drive; файли очікуються в `MyDrive/EPS_2026/runs/bitcd_semantic_v7/best.pt` та `threshold.json`. Необов'язкові приклади розміщуйте в `MyDrive/EPS_2026/demo_pairs/` як `<назва>_pre.png`, `<назва>_post.png` і `<назва>_label.png`. Без власного файлу порога демо використовує `0.5`.

Демо запускається з `share=True` і створює тимчасове публічне посилання Gradio. Для локального інтерфейсу змініть цей параметр на `False` у фінальній комірці.

## Навчання та оцінювання

Відкрийте основний ноутбук у Colab із T4 GPU. Для експерименту, порівнюваного зі звітом, додайте `FOTBCD-Instances.zip` або папку `FOTBCD-Instances/` у `MyDrive/EPS_2026/`. У розділі **1.2 «Конфігурація»** перевірте шляхи, `RUN_NAME` та прапорці запуску, потім виконайте **Run all**.

Основний ноутбук використовує [BIT_LEVIR_best_ckpt.pt](https://drive.google.com/file/d/1Qoh-0_Ub3xOdwRphfT_I4lXksm9nEZMK/view) — початкові ваги бінарної BIT-CD, навченої на LEVIR-CD, для донавчання й базового порівняння. Демо використовує готові `best.pt` та `threshold.json`.

- Початкові LEVIR-ваги та xBD Global завантажуються автоматично; архів xBD займає приблизно 8,4 ГБ. Дані й кеш потребують додаткового місця.
- За відсутності FOTBCD-Instances режим `auto` переходить на FOTBCD-Binary, і експеримент відрізняється від збереженого.
- `last.pt` дозволяє відновити навчання. Для нового експерименту використовуйте новий `RUN_NAME`.
- За `RUN_TRAIN=False` і `DOWNLOAD_SHARED_WEIGHTS=True` відсутній `best.pt` завантажується з посилання команди. Оцінювання все одно потребує даних, а їхній відбиток має збігатися з checkpoint.
- Поріг обирається на validation; test виконується один раз із `FORCE_RETEST=False`. Без кешу `val_hist.npz` основний ноутбук перерахує validation і поріг.

## Результати та обмеження

Наведено результати збереженого експерименту. Checkpoint: епоха `29`, поріг `0.31`, вікно `256`, крок `192`, `TTA=False`, відбиток даних `622605ccb7ea0133`.

| Test-вибірка | Тип даних | Бінарний піксельний F1 |
|---|---|---:|
| FOTBCD-Instances | Реальні | 0.764 |
| xBD-Global | Реальні | 0.642 |
| xBD-Ukraine | Реальні, інший домен | 0.039 |
| SYN-ID | Синтетика | 0.754 |
| SYN-SHIFT | Синтетика зі зміненими параметрами генератора | 0.763 |
| FOTBCD + xBD | Реальні, разом | 0.713 |

На Tesla T4 для пари `1024×1024`: середнє **0.151 с**, p95 **0.164 с**, максимум **0.165 с** за 20 вимірювань. Це час від масивів у пам'яті до карти класів і списку об'єктів; читання файлів та робота інтерфейсу до нього не входять.

Знімки мають бути геометрично вирівняні; зміна розміру в демо не виконує їхню реєстрацію. Реальна розмітка руйнувань стосується переважно будівель. Якість залежить від домену й роздільності; площа в м² — від правильності GSD. Оцінка впевненості не є відкаліброваною ймовірністю.

<details>
<summary>Повний збережений звіт: покласові та об'єктні метрики</summary>

### Підсумок: Change Detection (кейс 4.2)

- Модель: BIT-CD (LEVIR init) + голова типу змін; checkpoint epoch 29
- Поріг (обрано на validation): 0.31; інференс: вікно 256, крок 192, TTA=False
- Дані: FOTBCD (instances), xBD Global (спліт: spatial), синтетика (ID + Shift), xBD-Ukraine (лише test)
- Відбиток даних: 622605ccb7ea0133

#### Точність (TEST, вимога F1 ≥ 0,75)

| Джерело | Тип даних | F1 (піксельний, бінарний) | Статус |
|---|---|---|---|
| FOTBCD-Instances | реальні | 0.764 (P 0.809, R 0.723) | PASS |
| xBD-Global | реальні | 0.642 (P 0.662, R 0.623) | FAIL |
| xBD-Ukraine | реальні, OOD | 0.039 (P 0.053, R 0.030) | FAIL |
| SYN-ID | синтетика | 0.754 (P 0.717, R 0.796) | PASS |
| SYN-SHIFT | синтетика (зсув генератора) | 0.763 (P 0.739, R 0.789) | PASS |
| REAL (FOTBCD+xBD) | реальні, разом | 0.713 (P 0.747, R 0.682) | FAIL |

| Джерело | Клас | F1 піксельний | F1 об'єктний | Статус (піксельний) |
|---|---|---|---|---|
| FOTBCD-Instances | зруйнований об'єкт | 0.635 | 0.235 | FAIL |
| FOTBCD-Instances | нова забудова (інша зміна) | 0.791 | 0.672 | PASS |
| xBD-Global | зруйнований об'єкт | 0.644 | 0.605 | FAIL |
| xBD-Ukraine | зруйнований об'єкт | 0.050 | 0.024 | FAIL |
| SYN-ID | зруйнований об'єкт | 0.506 | 0.300 | FAIL |
| SYN-ID | нова вирва | 0.857 | 0.785 | PASS |
| SYN-ID | свіжа фортифікація | 0.777 | 0.712 | PASS |
| SYN-ID | нова забудова (інша зміна) | 0.716 | 0.576 | FAIL |
| SYN-SHIFT | зруйнований об'єкт | 0.573 | 0.457 | FAIL |
| SYN-SHIFT | нова вирва | 0.852 | 0.771 | PASS |
| SYN-SHIFT | свіжа фортифікація | 0.728 | 0.719 | FAIL |
| SYN-SHIFT | нова забудова (інша зміна) | 0.695 | 0.564 | FAIL |

**Shortcut-тест синтетики:** SYN-ID: хибних спрацювань на «старих» об'єктах 2.8% об'єктів / 1.3% пікселів; SYN-SHIFT: хибних спрацювань на «старих» об'єктах 6.8% об'єктів / 2.9% пікселів

#### Швидкодія (вимога ≤ 2 с на пару 1024×1024)

- Tesla T4: середнє 0.151 с, p95 0.164 с, максимум 0.165 с → **PASS** (реальні тестові пари 1024×1024)
- з TTA×4: максимум 0.578 с; читання PNG окремо ≈ 0.086 с

#### Довідково: zero-shot LEVIR BIT-CD (validation, поріг 0,5)

FOTBCD-Instances: F1 0.380, xBD-Global: F1 0.043, SYN-ID: F1 0.146, REAL (FOTBCD+xBD): F1 0.304

</details>

## Локальний запуск

Основний сценарій — Colab. Для локального Jupyter:

```bash
python -m venv .venv
# PowerShell: .\.venv\Scripts\Activate.ps1
# Linux / macOS: source .venv/bin/activate
python -m pip install -r requirements.txt
jupyter lab
```

Для GPU встановіть сумісну пару PyTorch / torchvision за [офіційною інструкцією](https://pytorch.org/get-started/locally/). `requirements.txt` містить прямі залежності, а не точний lock-файл експерименту.

Поза Colab `EPS_WORK` задає робочу папку. У демо з режимом `shared` ваги зберігаються в `WORK/assets/runs/bitcd_semantic_v7/`; у режимі `drive` папку з власними файлами задає `EPS_DRIVE_ROOT`. Повний навчальний конвеєр на Windows може потребувати адаптації багатопроцесної обробки.

## Джерела та умови використання

- **BIT-CD:** Hao Chen, Zipeng Qi, Zhenwei Shi, *Remote Sensing Image Change Detection with Transformers*, 2021, [DOI](https://doi.org/10.1109/TGRS.2021.3095166), [код і початкові LEVIR-ваги](https://github.com/justchenhao/BIT_CD). В оригінальному [розділі License](https://github.com/justchenhao/BIT_CD#license) зазначено некомерційне й дослідницьке використання.
- **FOTBCD:** Abdelrrahman Moubane, 2026, [стаття](https://arxiv.org/abs/2601.22596), [офіційний репозиторій і дані](https://github.com/abdelpy/FOTBCD-datasets), [FOTBCD-Binary](https://huggingface.co/datasets/retgenai/FOTBCD-Binary). Для FOTBCD-Binary / Instances зазначено CC BY-NC-SA 4.0.
- **xBD:** [оригінальне джерело](https://xview2.org/dataset), [використане дзеркало xBD Global](https://huggingface.co/datasets/DrNerd/xBD-Training-Data), [xBD-Ukraine](https://huggingface.co/datasets/sda-kr/xbd-ukraine). xBD-Ukraine використовується лише для оцінювання перенесення в інший домен.

Синтетичні об'єкти генеруються ноутбуком на фонах відповідних датасетів. Сторонні дані, ваги й бібліотеки мають власні умови використання.
