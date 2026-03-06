точная спецификация ядра: структура данных, формулы адаптивного алгоритма и числовая экономика наград. Без масштабирования, client-only.

Структура данных (client-side)

Диапазон множителей: 2–9. Храним только уникальные пары (a ≤ b), всего 36 ячеек.

Глобальное состояние:

appState = {
  profile: {
    totalXP: 0,
    level: 1,
    streak: 0,
    bestStreak: 0,
    fatigueIndex: 0,          // 0–1, растёт при замедлении
    lastSessionAt: 0
  },
  session: {
    startAt: 0,
    durationMs: 60000,        // 60–90 сек
    answered: 0,
    correct: 0,
    sumReactionMs: 0,
    mode: "normal"            // normal | blitz | boss
  },
  matrix: {
    "2x2": CellStat,
    "2x3": CellStat,
    ...
    "9x9": CellStat
  }
}

Структура ячейки:

CellStat = {
  a: 2,
  b: 7,
  attempts: 0,
  correct: 0,
  errors: 0,
  avgReactionMs: 0,          // EMA
  lastSeenAt: 0,
  lastCorrectAt: 0,
  masteryScore: 0,           // 0–100
  stabilityIndex: 0,         // 0–1
  intervalDays: 0            // для повторения
}

Обновление avgReactionMs — через экспоненциальное сглаживание:
EMA_new = α * current + (1 − α) * EMA_old
α = 0.25

Это даёт чувствительность к последним ответам.

Адаптивный алгоритм (точные формулы)

Цель: вычислить masteryScore ∈ [0,100].

Целевое время автоматизма:
targetMs = 2000

2.1. Accuracy

accuracy = correct / attempts
если attempts < 5 → accuracyWeight снижается (см. ниже)

2.2. Speed factor

speedFactor = clamp(1 − (avgReactionMs / targetMs), 0, 1)

Если avgReactionMs = 1000 → speedFactor = 0.5
Если ≥2000 → 0

2.3. Forgetting decay (забывание)

daysSinceSeen = (now − lastSeenAt) / 86400000

decay = min(0.15 * daysSinceSeen, 0.4)

Максимальная деградация — 40%.

2.4. Stability index

stabilityIndex = clamp(
(accuracy * 0.7 + speedFactor * 0.3),
0,
1
)

Это показатель устойчивости.

2.5. Mastery Score

Базовая формула:

rawMastery =
60 * accuracy +
30 * speedFactor +
10 * stabilityIndex

masteryScore =
clamp(rawMastery * (1 - decay), 0, 100)

Автоматизм считается достигнутым при:

attempts ≥ 100
accuracy ≥ 0.95
avgReactionMs ≤ 2000
masteryScore ≥ 85

Выбор следующего примера (персонализация)

Вес каждой ячейки:

priority =
(100 - masteryScore) * 0.6 +
decay * 100 * 0.3 +
randomBoost

randomBoost ∈ [0, 10]

Затем используется взвешенный случайный выбор.

Это создаёт:
– фокус на слабых местах
– возврат забытых
– элемент непредсказуемости

Механизм напоминает поведенческую ленту по принципу переменного подкрепления, используемого в TikTok.

Экономика наград (в числах)

4.1. Базовые очки

correctXP = 10
incorrectXP = 0

4.2. Множитель за streak

Каждые 5 правильных подряд:

multiplier = 1 + floor(streak / 5) * 0.2

Пример:
0–4 → x1
5–9 → x1.2
10–14 → x1.4
максимум x2

4.3. Скоростной бонус

Если reactionMs ≤ 1500:

speedBonus = 5 XP

Если ≤1000:

speedBonus = 10 XP

4.4. Переменное подкрепление

Вероятности:

5% → x2 к текущему примеру
3% → blitz-режим (10 быстрых примеров)
1% → boss-задача (x5 XP, но без права на ошибку)

4.5. XP формула за ответ

xpEarned =
(correctXP + speedBonus) *
multiplier *
randomEventMultiplier

Пример:
10 базовых + 5 скорость = 15
streak x1.4 → 21
если x2 событие → 42 XP

4.6. Уровни

XP для уровня:

nextLevelXP =
100 + level^1.5 * 20

Рост нелинейный, но умеренный.

Интервальное повторение

После достижения automated:

intervalDays = 1

Дальше при успешной проверке:

intervalDays *= 2

Последовательность:
1 → 2 → 4 → 8 → 16

Если accuracy < 0.9 в проверке:
intervalDays = max(1, intervalDays / 2)

Это предотвращает иллюзию знания.

Контроль усталости (fatigueIndex)

Если среднее время реакции в сессии растёт >20%:

fatigueIndex += 0.1 (max 1)

При fatigueIndex > 0.7:
вероятность бонус-событий увеличивается в 1.5 раза.

Это управляемая стимуляция вовлечённости.

Поведенческий цикл в цифрах

1 ответ ≈ 1.5–2 сек
1 сессия ≈ 30–40 примеров
1 сессия ≈ 400–800 XP
до автоматизма по одной зоне ≈ 3–5 дней