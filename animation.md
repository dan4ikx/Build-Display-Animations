# BDA Animation Guide

> Практическая документация по структуре `.bda`, созданию сцен и реализации runtime.

---

# 1. Общая идея

Файл `.bda` — UTF-8 JSON-документ.

```text
example.bda
```

Внутри находятся:

```text
BDA
├── metadata
├── scene
├── rigs
├── actors
├── clips
│   ├── animation tracks
│   ├── constraints
│   └── events
└── playback
```

Главный принцип:

> Один `.bda` может описывать не только анимацию одной сущности, а целую синхронизированную сцену.

---

# 2. Корневая структура

```json
{
  "format": "bda",
  "bda_version": "1.0.0",

  "meta": {},
  "scene": {},
  "variables": {},
  "rigs": {},
  "actors": {},
  "clips": {},
  "playback": {}
}
```

## Обязательные поля

| Поле | Назначение |
|---|---|
| `format` | всегда `bda` |
| `bda_version` | версия спецификации |
| `meta` | информация о файле |
| `actors` | персонажи / объекты |
| `clips` | анимации |
| `playback` | точка входа |

---

# 3. Metadata

```json
"meta": {
  "name": "hello_handshake",
  "title": "Hello",
  "author": "Example",
  "description": "Two characters shake hands",
  "duration": 6.0,
  "fps": 20,
  "created_with": "BDA Example",
  "units": {
    "position": "blocks",
    "rotation": "degrees",
    "time": "seconds"
  }
}
```

`fps` не должен определять физическую скорость runtime.

Правильно:

```text
time = 2.35 s
```

а не:

```text
frame = 47
```

FPS используется редактором для отображения timeline и экспорта.

---

# 4. Scene

```json
"scene": {
  "space": "world",
  "origin": [0, 64, 0],
  "loop": false,
  "speed": 1.0,
  "gravity": false
}
```

## space

Рекомендуемые значения:

```text
world
local
entity
```

### world

`origin` рассматривается как положение сцены в мире.

### local

Вся сцена существует относительно внешней точки запуска.

---

# 5. Actors

Actor — любой участник сцены.

Это может быть:

```text
minecraft:player
minecraft:zombie
minecraft:armor_stand
bda:camera
bda:object
custom:npc
```

Пример:

```json
"actors": {
  "alice": {
    "type": "minecraft:player",
    "rig": "minecraft:humanoid",

    "spawn": {
      "position": [-2, 0, 0],
      "rotation": [0, -90, 0]
    },

    "properties": {
      "skin": "alice",
      "visible": true
    }
  }
}
```

---

# 6. Rig

Rig описывает иерархию костей.

```json
"rigs": {
  "minecraft:humanoid": {
    "bones": {
      "root": {
        "parent": null
      },

      "body": {
        "parent": "root"
      },

      "head": {
        "parent": "body"
      },

      "right_arm": {
        "parent": "body"
      },

      "left_arm": {
        "parent": "body"
      },

      "right_leg": {
        "parent": "root"
      },

      "left_leg": {
        "parent": "root"
      }
    }
  }
}
```

## Рекомендуемые humanoid bones

```text
root
body
head

right_arm
right_forearm
right_hand

left_arm
left_forearm
left_hand

right_leg
right_lower_leg
right_foot

left_leg
left_lower_leg
left_foot
```

Runtime может поддерживать как простой 6-bone Minecraft rig, так и расширенный rig.

---

# 7. Transform

Для кости могут существовать:

```text
position
rotation
scale
visible
```

## position

```json
"position": {
  "0.0": [0, 0, 0],
  "1.0": [1, 0, 0]
}
```

## rotation

```json
"rotation": {
  "0.0": [0, 0, 0],
  "1.0": [-30, 20, 5]
}
```

Формат:

```text
[pitch, yaw, roll]
```

## scale

```json
"scale": {
  "0.0": [1, 1, 1],
  "2.0": [1.1, 1.1, 1.1]
}
```

---

# 8. Keyframes

## Короткая форма

```json
"1.5": [0, 45, 0]
```

Эквивалентна:

```json
"1.5": {
  "value": [0, 45, 0]
}
```

## Полная форма

```json
"1.5": {
  "value": [0, 45, 0],
  "easing": "bezier",
  "bezier": [0.25, 0.1, 0.25, 1.0]
}
```

Дополнительно:

```json
{
  "value": [0, 45, 0],
  "easing": "smoothstep",
  "hold": false
}
```

---

# 9. Интерполяция

BDA 1.0:

| easing | Поведение |
|---|---|
| `linear` | равномерное |
| `step` | мгновенный переход |
| `smoothstep` | мягкий старт и конец |
| `ease_in` | плавное ускорение |
| `ease_out` | плавное торможение |
| `ease_in_out` | плавно с двух сторон |
| `bezier` | пользовательская cubic Bézier |

Пример:

```json
"2.0": {
  "value": [0, 90, 0],
  "easing": "bezier",
  "bezier": [0.42, 0.0, 0.58, 1.0]
}
```

---

# 10. Root Motion

`root` — специальная кость.

```json
"root": {
  "position": {
    "0.0": [0, 0, 0],
    "2.0": [2, 0, 0]
  }
}
```

Для actor:

```json
"root_motion": {
  "mode": "apply",
  "collision": "ignore",
  "snap_to_ground": true
}
```

## mode

### apply

Runtime реально перемещает сущность.

### visual

Перемещается только визуальная модель.

### extract

Runtime вычисляет delta, но решение о перемещении принимает интеграция.

---

# 11. Clips

Один BDA может содержать несколько клипов.

```json
"clips": {
  "idle": {},
  "walk": {},
  "wave": {},
  "main_scene": {}
}
```

Каждый clip имеет собственную продолжительность.

```json
"main_scene": {
  "duration": 6.0,
  "loop": false,
  "actors": {}
}
```

---

# 12. Animation tracks нескольких actor

```json
"actors": {
  "alice": {
    "head": {
      "rotation": {
        "0": [0, 0, 0],
        "1": [0, 20, 0]
      }
    }
  },

  "bob": {
    "head": {
      "rotation": {
        "0": [0, 0, 0],
        "1": [0, -20, 0]
      }
    }
  }
}
```

Обе анимации вычисляются по одному `t`.

---

# 13. Синхронизация взаимодействий

Для взаимодействия двух персонажей рекомендуется использовать marker:

```json
{
  "time": 2.7,
  "type": "marker",
  "name": "hand_contact"
}
```

Также можно описать semantic contact:

```json
{
  "time": 2.7,
  "type": "contact",
  "name": "handshake",
  "a": {
    "actor": "alice",
    "socket": "right_hand"
  },
  "b": {
    "actor": "bob",
    "socket": "right_hand"
  }
}
```

`contact` не обязан двигать кости сам. Он сообщает runtime/редактору, что в данный момент указанные sockets должны совпасть или почти совпасть.

---

# 14. Sockets

Socket — именованная точка на кости.

```json
"sockets": {
  "right_hand": {
    "bone": "right_arm",
    "position": [0, -0.75, 0],
    "rotation": [0, 0, 0]
  }
}
```

Применения:

```text
оружие
факел
рукопожатие
IK target
предмет
частицы
камера
```

---

# 15. Attachments

```json
"attachments": {
  "sword": {
    "type": "item",
    "item": "minecraft:diamond_sword",
    "socket": "right_hand"
  }
}
```

Можно использовать:

```text
item
model
entity
particle_anchor
```

---

# 16. Events

## marker

```json
{
  "time": 1.0,
  "type": "marker",
  "name": "walk_started"
}
```

## sound

```json
{
  "time": 1.0,
  "type": "sound",
  "sound": "minecraft:block.grass.step",
  "actor": "alice"
}
```

## particle

```json
{
  "time": 2.0,
  "type": "particle",
  "particle": "minecraft:happy_villager",
  "actor": "alice",
  "socket": "right_hand"
}
```

## command

```json
{
  "time": 5.0,
  "type": "command",
  "command": "say Scene complete",
  "permission": "server"
}
```

Команды должны быть отключены по умолчанию для недоверенных файлов.

## custom

```json
{
  "time": 3.0,
  "type": "custom",
  "namespace": "example",
  "event": "quest_progress",
  "data": {
    "quest": "hello",
    "step": 2
  }
}
```

---

# 17. Constraints

Constraints вычисляются после обычной interpolation.

Рекомендуемый pipeline:

```text
keyframes
    ↓
interpolation
    ↓
layers
    ↓
constraints
    ↓
final pose
```

## look_at

```json
{
  "type": "look_at",
  "actor": "alice",
  "bone": "head",

  "target": {
    "actor": "bob",
    "bone": "head"
  },

  "weight": 0.7,

  "limits": {
    "yaw": [-70, 70],
    "pitch": [-35, 45]
  }
}
```

## socket_match

Для рукопожатия:

```json
{
  "type": "socket_match",

  "source": {
    "actor": "alice",
    "socket": "right_hand"
  },

  "target": {
    "actor": "bob",
    "socket": "right_hand"
  },

  "weight": {
    "0.0": 0.0,
    "2.5": 0.0,
    "2.8": 1.0,
    "4.0": 1.0,
    "4.3": 0.0
  }
}
```

Это позволяет редактору или IK-runtime аккуратно убрать небольшой визуальный зазор между руками.

---

# 18. IK

Предлагаемые типы:

```text
ik_2bone
look_at
socket_match
ground
copy_transform
distance
parent
```

Пример:

```json
{
  "type": "ik_2bone",
  "actor": "alice",

  "chain": [
    "right_arm",
    "right_forearm",
    "right_hand"
  ],

  "target": {
    "actor": "bob",
    "socket": "right_hand"
  },

  "weight": 1.0
}
```

---

# 19. Layers

Можно смешивать несколько клипов.

```json
"layers": [
  {
    "clip": "walk",
    "weight": 1.0,
    "mask": [
      "root",
      "right_leg",
      "left_leg"
    ]
  },

  {
    "clip": "look",
    "weight": 0.8,
    "mask": [
      "head"
    ]
  }
]
```

Так персонаж одновременно:

```text
идёт
+
смотрит в сторону
+
держит предмет
```

---

# 20. Variables

```json
"variables": {
  "walk_speed": 1.0,
  "look_weight": 0.8,
  "handshake_strength": 1.0
}
```

Для BDA 1.1 рекомендуется безопасная AST-система выражений вместо `eval`.

Например:

```json
"value_expr": [
  "mul",
  ["var", "walk_speed"],
  30
]
```

---

# 21. Playback

```json
"playback": {
  "entry": "main",
  "speed": 1.0,
  "start_offset": 0.0,
  "blend_in": 0.15,
  "blend_out": 0.2
}
```

---

# 22. Полный жизненный цикл runtime

## Шаг 1 — загрузка

```text
Hello.bda
```

## Шаг 2 — JSON parse

Runtime читает UTF-8 JSON.

## Шаг 3 — проверка версии

```text
1.x.x → поддерживается BDA 1 runtime
2.x.x → runtime должен отказаться, если BDA 2 неизвестен
```

## Шаг 4 — schema validation

Проверяются:

```text
actors
clips
references
rigs
keyframes
events
```

## Шаг 5 — resolve

Связываются:

```text
actor → rig
bone → parent
attachment → socket
constraint → actor/socket
playback.entry → clip
```

## Шаг 6 — playback

На каждом tick/frame:

```text
t = current animation time
```

Для каждого actor:

```text
sample root
sample body
sample head
sample arms
sample legs
```

## Шаг 7 — interpolation

Значение вычисляется между соседними keyframes.

## Шаг 8 — constraints

Исправляется pose:

```text
IK
look_at
socket_match
grounding
```

## Шаг 9 — apply

Результат передаётся Minecraft implementation.

## Шаг 10 — events

Runtime проверяет события:

```text
previous_t < event.time <= current_t
```

---

# 23. Псевдокод evaluator

```java
Pose evaluateActor(Actor actor, Clip clip, double time) {
    Pose pose = actor.bindPose();

    for (BoneTrack track : clip.tracks(actor.id())) {
        Transform transform = sample(track, time);
        pose.apply(track.bone(), transform);
    }

    pose = applyLayers(pose, time);
    pose = solveConstraints(pose, time);

    return pose;
}
```

---

# 24. Псевдокод keyframe interpolation

```java
Keyframe a = previousKey(time);
Keyframe b = nextKey(time);

double progress =
    (time - a.time()) /
    (b.time() - a.time());

double eased = easing(a, progress);

Vector3 result = lerp(
    a.value(),
    b.value(),
    eased
);
```

---

# 25. Организация проекта

Рекомендуемая структура:

```text
BDA/
│
├── README.md
├── animation.md
│
├── examples/
│   ├── Hello.bda
│   ├── Walk.bda
│   └── Fight.bda
│
├── schema/
│   └── bda.schema.json
│
├── runtime/
│   ├── api/
│   ├── parser/
│   ├── model/
│   ├── timeline/
│   ├── interpolation/
│   ├── constraints/
│   ├── events/
│   └── platform/
│       ├── paper/
│       ├── fabric/
│       └── neoforge/
│
├── editor/
│
└── tools/
    ├── validator/
    └── converters/
```

---

# 26. Java model

В Java runtime удобно представить файл так:

```java
record BdaDocument(
    String format,
    String bdaVersion,
    Meta meta,
    Scene scene,
    Map<String, Rig> rigs,
    Map<String, Actor> actors,
    Map<String, Clip> clips,
    Playback playback
) {}
```

---

# 27. Как создать свою первую анимацию

## 1. Создайте файл

```text
MyAnimation.bda
```

## 2. Добавьте metadata

```json
{
  "format": "bda",
  "bda_version": "1.0.0",
  "meta": {
    "name": "my_animation",
    "duration": 3.0,
    "fps": 20
  }
}
```

## 3. Добавьте actor

```json
"actors": {
  "player": {
    "type": "minecraft:player",
    "rig": "minecraft:humanoid"
  }
}
```

## 4. Создайте clip

```json
"clips": {
  "main": {
    "duration": 3.0,
    "actors": {}
  }
}
```

## 5. Добавьте движение

```json
"actors": {
  "player": {
    "right_arm": {
      "rotation": {
        "0.0": [0, 0, 0],
        "1.0": [-120, 0, 0],
        "2.0": [-120, 0, 0],
        "3.0": [0, 0, 0]
      }
    }
  }
}
```

## 6. Укажите entry clip

```json
"playback": {
  "entry": "main"
}
```

Готово.

---

# 28. Пример сцены Hello.bda

`Hello.bda` демонстрирует рукопожатие двух персонажей.

Timeline:

```text
0.00 ───────────────────────────────────────────── 6.00

Alice:
idle ─ walk ─ stop ─ reach ─ shake ─ release ─ idle

Bob:
idle ─ walk ─ stop ─ reach ─ shake ─ release ─ idle

                         ▲
                         │
                  handshake_contact
```

Основные marker:

```text
approach_start
approach_end
handshake_reach
handshake_contact
handshake_release
scene_complete
```

---

# 29. Рекомендации редактору

BDA editor должен иметь:

```text
Outliner
Timeline
Dope Sheet
Curve Editor
Scene View
Actor List
Bone List
Event Track
Constraint Track
Properties
```

Для нескольких персонажей желательно:

```text
Alice
├── root
├── body
├── head
├── right_arm
└── ...

Bob
├── root
├── body
├── head
├── right_arm
└── ...
```

---

# 30. Безопасность

Нельзя автоматически доверять:

```text
command
custom event
external resource
remote URL
```

Runtime должен иметь:

```text
allow_commands=false
allow_remote_resources=false
trusted_namespaces=[]
```

---

# 31. Совместимость версий

Рекомендуемое правило:

```text
runtime 1.2 поддерживает:
1.0
1.1
1.2
```

Но:

```text
runtime 1.x
```

не обязан открывать:

```text
BDA 2.x
```

---

# 32. Что можно добавить в BDA 1.1+

План расширения:

- animation state machines;
- curves;
- quaternion rotations;
- additive animation;
- inverse kinematics;
- physics tracks;
- facial expressions;
- camera cuts;
- subtitles;
- dialogue tracks;
- reusable sub-clips;
- external animation libraries;
- procedural motion;
- animation retargeting;
- Blockbench import/export;
- binary `.bdac` compiled format.

---

# 33. `.bda` и будущий `.bdac`

Для разработки удобен:

```text
scene.bda
```

Для runtime можно позже ввести скомпилированный:

```text
scene.bdac
```

Pipeline:

```text
scene.bda
   ↓
validator
   ↓
compiler
   ↓
scene.bdac
   ↓
Minecraft runtime
```

`.bda` остаётся source format.

---

# 34. Главное правило формата

**BDA должен оставаться понятным человеку.**

Если простую анимацию нельзя написать вручную без специального редактора — формат стал слишком сложным.

Поэтому базовая анимация всегда должна выглядеть примерно так:

```json
"head": {
  "rotation": {
    "0": [0, 0, 0],
    "1": [0, 30, 0],
    "2": [0, 0, 0]
  }
}
```

А продвинутые возможности должны быть необязательными.
