# Lead Prioritization Challenge

Ранжирование обращений (лидов) по вероятности успешного целевого действия
в течение 5 дней после назначения оператору. Метрика — Daily Average Precision
(AP считается внутри каждого дня, затем усредняется).

**Результат на скрытом тесте: Daily AP = 0.7206** (baseline LogReg — 0.49).

## Структура

```
data/                    # данные организаторов: train.csv, test.csv, events.csv,
                         # sample_submission.csv (в git не входят)
notebooks/
  quickstart.ipynb       # baseline от организаторов
  submission.ipynb       # решение: EDA -> признаки -> эксперименты -> финальная модель
submission_final.csv     # предсказания для test.csv (lead_id, score)
requirements.txt         # зафиксированные версии библиотек
```

## Запуск

```bash
python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
# распаковать данные организаторов в data/
jupyter notebook notebooks/submission.ipynb   # Run All -> submission_final.csv
```

Все random seed зафиксированы (42): повторный запуск воспроизводит
submission_final.csv побайтово. Полный прогон ~50 минут (эксперименты и CV);
финальная модель без экспериментов обучается ~1 минуту.

## Кратко о решении

- **Модель**: CatBoost (lr=0.01, depth=6, early stopping) на 168 признаках.
- **Валидация**: time-based holdout (последние 4 дня train) + expanding window CV;
  локальная оценка (0.729 / 0.706) совпала со скрытым тестом (0.721).
- **Ключевые находки**: скрытый непрерывный сигнал в дробной части счетчиков
  seller_page_views (+2.3 п.п.); признаки из сырых событий строго до момента
  назначения — счетчики, давность, разнообразие контекстов, интервалы, ценовой
  профиль (+16 п.п. суммарно).
- Подробное описание подхода и таблица экспериментов — в notebooks/submission.ipynb.
