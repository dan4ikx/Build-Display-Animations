# BDA 1.0 — структура формата

Этот документ описывает `.bda` версии **1.0**.

## 1. Что хранит BDA

BDA 1.0 описывает анимацию Minecraft-персонажей:

- одного или нескольких персонажей;
- 6 стандартных костей;
- keyframes;
- вращение костей;
- необязательное локальное смещение костей;
- необязательную анимацию положения персонажа в пространстве.

---

## 2. Корневая структура

```json
{
  "format": "bda",
  "version": "1.0",
  "animation": {
    "name": "example",
    "duration": 3.0,
    "fps": 20,
    "loop": false,
    "characters": []
  }
}
```

Обязательные поля:

- `format` — всегда `bda`;
- `version` — всегда `1.0`;
- `animation`;
- `animation.name`;
- `animation.duration`;
- `animation.characters`.

`fps` и `loop` необязательны.

---

## 3. Character

```json
{
  "id": "player",
  "space": {},
  "bones": {}
}
```

`id` — локальный уникальный идентификатор персонажа внутри файла.

Он не обязан совпадать с UUID или ником Minecraft-игрока.

---

## 4. Ровно шесть костей

Каждый персонаж должен содержать:

```text
head
body
left_arm
right_arm
left_leg
right_leg
```

Полная заготовка:

```json
"bones": {
  "head": {},
  "body": {},
  "left_arm": {},
  "right_arm": {},
  "left_leg": {},
  "right_leg": {}
}
```

В версии 1.0 используется этот фиксированный набор из шести костей.

---

## 5. Bone

Кость может содержать:

```text
rotation
position
```

Пример:

```json
"right_arm": {
  "rotation": {
    "0.0": [0, 0, 0],
    "1.0": [-80, -10, 5]
  }
}
```

`position` — необязательное локальное смещение:

```json
"head": {
  "position": {
    "0.0": [0, 0, 0],
    "1.0": [0, 0.05, 0]
  }
}
```

---

## 6. Векторы

Позиция:

```text
[x, y, z]
```

Вращение:

```text
[pitch, yaw, roll]
```

Вращение задаётся в градусах.

---

## 7. Keyframe

Простая форма:

```json
"1.5": [0, 45, 0]
```

Она эквивалентна:

```json
"1.5": {
  "value": [0, 45, 0],
  "easing": "linear"
}
```

Расширенная форма:

```json
"1.5": {
  "value": [0, 45, 0],
  "easing": "ease_in_out"
}
```

Ключ (`1.5`) — время в секундах.

---

## 8. Easing

BDA 1.0:

```text
linear
step
smoothstep
ease_in
ease_out
ease_in_out
```

По умолчанию используется `linear`.

---

## 9. Space

`space` управляет положением всего персонажа.

```json
"space": {
  "position": {
    "0.0": [-2.5, 0, 0],
    "2.0": [-0.8, 0, 0]
  },
  "rotation": {
    "0.0": [0, 90, 0]
  }
}
```

Это необязательное поле.

Если `space` не указан, внешний runtime сохраняет текущее положение персонажа.

---

## 10. Статическое положение

Достаточно одной точки:

```json
"space": {
  "position": {
    "0.0": [4, 64, -7]
  },
  "rotation": {
    "0.0": [0, 180, 0]
  }
}
```

---

## 11. Несколько персонажей

```json
"characters": [
  {
    "id": "alice",
    "bones": {
      "head": {},
      "body": {},
      "left_arm": {},
      "right_arm": {},
      "left_leg": {},
      "right_leg": {}
    }
  },
  {
    "id": "bob",
    "bones": {
      "head": {},
      "body": {},
      "left_arm": {},
      "right_arm": {},
      "left_leg": {},
      "right_leg": {}
    }
  }
]
```

Все персонажи вычисляются по одному времени `t`.

---

## 12. Интерполяция

Пусть:

```text
A.time <= t <= B.time
```

Тогда:

```text
progress = (t - A.time) / (B.time - A.time)
```

После применения easing:

```text
e = easing(progress)
```

Для каждой координаты:

```text
result = A + (B - A) * e
```

До первой keyframe используется первая точка.

После последней keyframe используется последняя точка.

---

## 13. Пример ходьбы

```json
{
  "id": "player",
  "space": {
    "position": {
      "0.0": [0, 0, 0],
      "2.0": {
        "value": [0, 0, 3],
        "easing": "ease_in_out"
      }
    }
  },
  "bones": {
    "head": {},
    "body": {},
    "left_arm": {
      "rotation": {
        "0.0": [0, 0, 0],
        "0.5": [-25, 0, 0],
        "1.0": [25, 0, 0],
        "1.5": [-25, 0, 0],
        "2.0": [0, 0, 0]
      }
    },
    "right_arm": {
      "rotation": {
        "0.0": [0, 0, 0],
        "0.5": [25, 0, 0],
        "1.0": [-25, 0, 0],
        "1.5": [25, 0, 0],
        "2.0": [0, 0, 0]
      }
    },
    "left_leg": {
      "rotation": {
        "0.0": [0, 0, 0],
        "0.5": [30, 0, 0],
        "1.0": [-30, 0, 0],
        "1.5": [30, 0, 0],
        "2.0": [0, 0, 0]
      }
    },
    "right_leg": {
      "rotation": {
        "0.0": [0, 0, 0],
        "0.5": [-30, 0, 0],
        "1.0": [30, 0, 0],
        "1.5": [-30, 0, 0],
        "2.0": [0, 0, 0]
      }
    }
  }
}
```

---

## 14. Hello.bda

`examples/Hello.bda` содержит двух персонажей.

Они:

1. начинают на расстоянии друг от друга;
2. идут навстречу;
3. останавливаются;
4. слегка поворачивают головы и туловище;
5. одновременно протягивают правые руки;
6. несколько раз делают движение рукопожатия;
7. опускают руки;
8. возвращаются в нейтральную позу.

Файл использует только возможности BDA 1.0.

---

## 15. Минимальный runtime

```text
.bda
  ↓
JSON parser
  ↓
проверка format/version
  ↓
animation.characters
  ↓
sample space
  ↓
sample 6 bones
  ↓
interpolation
  ↓
Minecraft entity/model
```

Псевдокод:

```java
for (Character character : animation.characters()) {
    Transform space = sample(character.space(), time);

    for (Bone bone : SIX_BONES) {
        BonePose pose = sample(character.bones().get(bone), time);
        apply(character.id(), bone, space, pose);
    }
}
```

---

## 16. Валидация

Runtime должен считать файл некорректным, если:

- `format != "bda"`;
- `version != "1.0"`;
- отсутствует `animation`;
- `duration <= 0`;
- нет персонажей;
- ID персонажей повторяются;
- отсутствует хотя бы одна из шести костей;
- присутствует неизвестная кость;
- vector содержит не три числа;
- keyframe имеет отрицательное время;
- keyframe находится позже `animation.duration`.

---

## 17. Разрешённые поля BDA 1.0

```text
format
version
animation
name
duration
fps
loop
characters
id
space
bones
position
rotation
value
easing
```

Так BDA остаётся маленьким, понятным и легко реализуемым.
