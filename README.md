<div align="center">

# BDA 2.0 — More Moving

### More detailed Minecraft character animation format

![Version](https://img.shields.io/badge/BDA-2.0-6d5dfc?style=for-the-badge)
![Profile](https://img.shields.io/badge/profile-More%20Moving-22c55e?style=for-the-badge)
![Syntax](https://img.shields.io/badge/syntax-JSON-f59e0b?style=for-the-badge)

**BDA 2.0** expands the character rig from the old simple layout to a more flexible segmented body.

</div>

---

## Что нового в BDA 2.0

BDA 2.0 **More Moving** даёт более детальную анимацию частей тела.

### Новый набор костей

Теперь персонаж использует **11 костей**:

| Группа | Кости | Кол-во |
|---|---|---:|
| Голова | `head` | 1 |
| Туловище | `chest`, `torso` | 2 |
| Левая рука | `left_shoulder`, `left_arm` | 2 |
| Правая рука | `right_shoulder`, `right_arm` | 2 |
| Левая нога | `left_leg_upper`, `left_leg_lower` | 2 |
| Правая нога | `right_leg_upper`, `right_leg_lower` | 2 |

Итого: **11 костей**.

---

## Схема тела по рисунку

Ниже — обработанный референс на основе присланного изображения:

![BDA 2.0 bone map](docs/images/bda_2_bone_map.png)

Цветовая логика из рисунка:

- **Голова** — бежевый блок;
- **Плечи** — синие блоки;
- **Руки** — оранжевые блоки;
- **Грудь** — зелёный блок;
- **Туловище** — голубой блок;
- **Верх ног** — красный блок;
- **Низ ног** — розовый блок.

---

## Минимальная структура файла

```json
{
  "format": "bda",
  "version": "2.0",
  "profile": "more_moving",
  "animation": {
    "name": "example",
    "duration": 3.0,
    "fps": 20,
    "loop": false,
    "characters": []
  }
}
```

---

## Один персонаж

```json
{
  "id": "player_1",
  "space": {
    "position": {
      "0.0": [0, 0, 0]
    },
    "rotation": {
      "0.0": [0, 180, 0]
    }
  },
  "bones": {
    "head": {},
    "chest": {},
    "torso": {},
    "left_shoulder": {},
    "left_arm": {},
    "right_shoulder": {},
    "right_arm": {},
    "left_leg_upper": {},
    "left_leg_lower": {},
    "right_leg_upper": {},
    "right_leg_lower": {}
  }
}
```

---

## Пример keyframes

```json
"right_arm": {
  "rotation": {
    "0.0": [0, 0, 0],
    "0.6": {
      "value": [-35, -10, -5],
      "easing": "ease_out"
    },
    "1.4": {
      "value": [-75, -18, -8],
      "easing": "ease_in_out"
    },
    "2.0": [0, 0, 0]
  }
}
```

---

## Иерархия костей

BDA 2.0 использует фиксированную иерархию:

```text
torso
├── chest
│   ├── head
│   ├── left_shoulder
│   │   └── left_arm
│   └── right_shoulder
│       └── right_arm
├── left_leg_upper
│   └── left_leg_lower
└── right_leg_upper
    └── right_leg_lower
```

Эта схема делает движения естественнее:

- поворот `torso` переносит верхнюю часть тела;
- `chest` позволяет отдельно работать с верхом корпуса;
- плечи и руки можно двигать независимо;
- верх и низ ног можно анимировать раздельно.

---

## Что лежит в репозитории

```text
BDA-2.0-More-Moving/
├── README.md
├── animation.md
├── docs/
│   └── images/
│       └── bda_2_bone_map.png
├── examples/
│   └── Hello.bda
└── schema/
    └── bda.schema.json
```

---

## Hello.bda

`examples/Hello.bda` демонстрирует анимацию двух персонажей:

1. они идут навстречу друг другу;
2. сгибают верхнюю и нижнюю части ног;
3. отдельно поворачивают `chest` и `torso`;
4. поднимают плечи;
5. протягивают руки;
6. делают несколько движений рукопожатия;
7. возвращаются в нейтральную позу.

---

## Ключевая идея BDA 2.0

**BDA 2.0 More Moving** сохраняет простоту JSON-формата, но даёт больше контроля над телом персонажа за счёт разделения крупных частей на дополнительные кости.
