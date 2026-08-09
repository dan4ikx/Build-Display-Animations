<div align="center">

# BDA

### Bedrock-like Dynamic Animation

Простой открытый формат анимации Minecraft-персонажей.

**Версия формата: `1.0`**

![Version](https://img.shields.io/badge/BDA-1.0-6d5dfc?style=for-the-badge)
![Format](https://img.shields.io/badge/file-.bda-22c55e?style=for-the-badge)
![Syntax](https://img.shields.io/badge/syntax-JSON-f59e0b?style=for-the-badge)

</div>

---

## Что такое BDA?

**BDA (`.bda`)** — простой JSON-формат для хранения анимации Minecraft-персонажей.

BDA 1.0 предназначен для описания анимации Minecraft-персонажей:

- один или несколько персонажей;
- единая временная шкала;
- ключевые точки по времени;
- вращение и локальное смещение костей;
- при необходимости — положение и вращение персонажа в пространстве;
- простая интерполяция.

---

## Шесть костей

Каждый персонаж имеет ровно эти 6 костей:

| Кость | Часть |
|---|---|
| `head` | голова |
| `body` | туловище |
| `left_arm` | левая рука |
| `right_arm` | правая рука |
| `left_leg` | левая нога |
| `right_leg` | правая нога |

---

## Минимальный пример

```json
{
  "format": "bda",
  "version": "1.0",
  "animation": {
    "name": "wave",
    "duration": 2.0,
    "characters": [
      {
        "id": "player",
        "bones": {
          "head": {},
          "body": {},
          "left_arm": {},
          "right_arm": {
            "rotation": {
              "0.0": [0, 0, 0],
              "0.5": [-130, 0, 10],
              "1.0": [-130, 0, -10],
              "1.5": [-130, 0, 10],
              "2.0": [0, 0, 0]
            }
          },
          "left_leg": {},
          "right_leg": {}
        }
      }
    ]
  }
}
```

---

## Положение персонажа в пространстве

`space` полностью необязателен:

```json
"space": {
  "position": {
    "0.0": [-2.0, 0.0, 0.0],
    "2.0": [-0.8, 0.0, 0.0]
  },
  "rotation": {
    "0.0": [0.0, 90.0, 0.0]
  }
}
```

Форматы:

```text
position = [x, y, z]
rotation = [pitch, yaw, roll]
```

Позиция задаётся в Minecraft-блоках, вращение — в градусах.

Если `space` отсутствует, runtime не должен менять положение сущности в мире.

---

## Keyframes

Простая форма:

```json
"1.0": [0, 45, 0]
```

Расширенная:

```json
"1.0": {
  "value": [0, 45, 0],
  "easing": "ease_in_out"
}
```

Поддерживаемые easing:

- `linear`
- `step`
- `smoothstep`
- `ease_in`
- `ease_out`
- `ease_in_out`

---

## Несколько персонажей

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

Все персонажи используют одну timeline, поэтому их движения легко синхронизировать.

---

## Структура репозитория

```text
BDA-1.0/
├── README.md
├── animation.md
├── examples/
│   └── Hello.bda
└── schema/
    └── bda.schema.json
```

`Hello.bda` демонстрирует, как два персонажа подходят друг к другу и пожимают правые руки.

---

## Основной принцип

BDA должен оставаться настолько простым, чтобы небольшую анимацию можно было написать вручную:

```json
"head": {
  "rotation": {
    "0.0": [0, 0, 0],
    "1.0": [0, 30, 0],
    "2.0": [0, 0, 0]
  }
}
```

**BDA 1.0 = персонажи + 6 костей + keyframes + необязательное движение в пространстве.**
