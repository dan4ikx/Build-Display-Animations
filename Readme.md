<div align="center">

# ✦ BDA

### Block Dynamic Animation

**Открытый формат сценических и скелетных анимаций для Minecraft**

[![Format](https://img.shields.io/badge/format-.bda-7c5cff?style=for-the-badge)](#)
[![Version](https://img.shields.io/badge/spec-1.0.0-38bdf8?style=for-the-badge)](#)
[![Encoding](https://img.shields.io/badge/encoding-UTF--8-22c55e?style=for-the-badge)](#)
[![Syntax](https://img.shields.io/badge/syntax-JSON-f59e0b?style=for-the-badge)](#)

**Bedrock-подобные keyframe-анимации, но для целых сцен.**

Несколько персонажей · единая временная шкала · кости · root motion · события · предметы · constraints · камера

</div>

---

## Что такое BDA?

**BDA (`.bda`)** — человекочитаемый формат анимации, созданный для Minecraft.

По духу он похож на Bedrock Animation JSON: движение задаётся через **кости, ключевые точки и координаты**.  
Но BDA проектируется не вокруг одной модели, а вокруг **полноценной сцены**.

Один файл может содержать:

- 👥 несколько персонажей;
- 🦴 несколько скелетов / rig;
- 🎞 несколько animation clips;
- 📍 перемещение персонажей по сцене;
- 🔄 вращение, позицию и масштаб костей;
- 🤝 синхронные взаимодействия между персонажами;
- 🔊 звуки;
- ✨ частицы;
- ⚡ события;
- 🗡 предметы в руках;
- 🎥 камеры;
- 🎭 слои анимации;
- 👀 `look_at`;
- 🦾 IK и другие constraints;
- 🧩 расширения от модов и плагинов.

---

## Почему `.bda`?

Стандартные Minecraft-анимации хорошо подходят для анимации одной модели, но становятся неудобными, когда необходимо описать:

> «Персонаж A подходит к персонажу B, они смотрят друг на друга, одновременно протягивают руки, пожимают их, проигрывается звук, а затем оба возвращаются в исходное положение».

В BDA это одна сцена и одна временная шкала.

```text
0.0s      1.0s      2.0s      3.0s      4.0s      5.0s
│----------│----------│----------│----------│----------│
A   идёт ───────────▶  протягивает руку ──┐
                                           ├─ handshake
B   идёт ───────────▶  протягивает руку ──┘
```

---

## Минимальный BDA

```json
{
  "format": "bda",
  "bda_version": "1.0.0",

  "meta": {
    "name": "hello",
    "duration": 5.0,
    "fps": 20
  },

  "actors": {
    "player": {
      "type": "minecraft:player",
      "rig": "minecraft:humanoid"
    }
  },

  "clips": {
    "main": {
      "duration": 5.0,
      "actors": {
        "player": {
          "head": {
            "rotation": {
              "0.0": [0, 0, 0],
              "2.0": [0, 35, 0],
              "5.0": [0, 0, 0]
            }
          }
        }
      }
    }
  },

  "playback": {
    "entry": "main"
  }
}
```

---

## Keyframes

Простая запись:

```json
"rotation": {
  "0.0": [0, 0, 0],
  "1.0": [0, 45, 0],
  "2.0": [0, 0, 0]
}
```

Расширенная запись:

```json
"1.0": {
  "value": [0, 45, 0],
  "easing": "bezier",
  "bezier": [0.25, 0.1, 0.25, 1.0]
}
```

BDA 1.0 предусматривает:

```text
linear
step
smoothstep
ease_in
ease_out
ease_in_out
bezier
```

---

## Несколько персонажей

```json
"actors": {
  "alice": {
    "type": "minecraft:player",
    "rig": "minecraft:humanoid"
  },

  "bob": {
    "type": "minecraft:player",
    "rig": "minecraft:humanoid"
  }
}
```

Они существуют на **одной timeline**, поэтому взаимодействия остаются синхронными.

---

## События

```json
"events": [
  {
    "time": 2.75,
    "type": "marker",
    "name": "handshake_contact"
  },

  {
    "time": 2.75,
    "type": "sound",
    "sound": "minecraft:entity.player.attack.nodamage",
    "position": {
      "between": ["alice", "bob"]
    }
  }
]
```

Runtime может использовать marker для своей логики:

```text
handshake_contact
attack_hit
left_foot_down
door_open
dialogue_02
camera_cut
```

---

## Координаты

Позиция:

```text
[x, y, z]
```

Вращение:

```text
[pitch, yaw, roll]
```

Масштаб:

```text
[x, y, z]
```

По умолчанию:

| Значение | Единица |
|---|---|
| Position | Minecraft block |
| Rotation | degrees |
| Time | seconds |
| Scale | multiplier |

---

## Структура репозитория

```text
BDA/
├── README.md
├── animation.md
├── examples/
│   └── Hello.bda
├── schema/
│   └── bda.schema.json
├── runtime/
│   ├── parser/
│   ├── evaluator/
│   ├── interpolation/
│   ├── constraints/
│   └── minecraft/
└── tools/
    ├── validator/
    └── converter/
```

---

## Пример: рукопожатие

В репозитории рекомендуется хранить `examples/Hello.bda`.

Сцена демонстрирует:

1. два независимых actor;
2. движение обоих персонажей;
3. повороты корпуса;
4. движение головы;
5. синхронную анимацию правых рук;
6. контакт рук;
7. небольшое движение рукопожатия;
8. events и markers;
9. easing;
10. возврат в idle.

---

## Как должен работать BDA Runtime

```text
              ┌───────────────┐
              │    .bda file  │
              └───────┬───────┘
                      │
                      ▼
              ┌───────────────┐
              │     Parser    │
              └───────┬───────┘
                      │
                      ▼
              ┌───────────────┐
              │   Validator   │
              └───────┬───────┘
                      │
                      ▼
              ┌───────────────┐
              │ Rig Resolver  │
              └───────┬───────┘
                      │
                      ▼
        ┌─────────────────────────────┐
        │     Timeline Evaluator      │
        └──────────────┬──────────────┘
                       │
          ┌────────────┼─────────────┐
          ▼            ▼             ▼
   Interpolation   Constraints     Events
          │            │             │
          └────────────┼─────────────┘
                       ▼
              ┌─────────────────┐
              │ Minecraft Layer │
              └─────────────────┘
```

---

## Цели проекта

BDA задуман как формат, который можно использовать в:

- Paper / Spigot;
- Fabric;
- NeoForge;
- серверных NPC-системах;
- Blockbench-плагинах;
- кат-сценах;
- machinima;
- квестовых системах;
- RPG-серверах;
- Minecraft-картах;
- собственных движках рендера.

---

## Принципы BDA

### Human-readable

Файл можно открыть обычным редактором.

### Deterministic

Одна и та же timeline должна давать одинаковый результат при одинаковых входных условиях.

### Extensible

Неизвестные необязательные поля minor-версий могут игнорироваться runtime.

### Safe by default

`.bda` не является исполняемым кодом.

### Scene-first

Персонажи, события и объекты синхронизируются одной временной шкалой.

---

## Статус

> **BDA 1.0 — experimental specification.**

Формат уже пригоден для создания parser/runtime prototype, но до появления стабильного runtime некоторые поля могут уточняться.

---

## Лицензирование

Для самого формата рекомендуется использовать **MIT** или **Apache-2.0**, чтобы BDA было легко внедрять в моды, плагины, редакторы и коммерческие Minecraft-проекты.

---

<div align="center">

### ✦ BDA

**Animate characters. Build scenes. Keep it readable.**

</div>
