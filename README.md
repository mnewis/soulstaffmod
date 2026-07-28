# Soul Staff Mod (Посох душ)

Мод для Minecraft Java Edition, Fabric, версия **1.20.1**.

## Что добавляет мод

### 🗡️ Посох душ (`soulstaffmod:soul_staff`)
- Урон: **12**
- Перезарядка удара: **15 секунд**
- При убийстве любого моба этим посохом с вероятностью **25%** с моба выпадает **предмет-спавнер** этого моба
  (спавнер уже настроен на нужный тип моба — его можно поставить в мире).
- Предмет **нельзя зачаровать** ни одним зачарованием (наковальня и стол зачарований его отвергнут).

**Крафт:**
```
[Изумрудный блок] [Изумрудный блок] [Изумрудный блок]
[Изумруд]          [Звезда Незера]  [Изумруд]
[Осколок души]     [Осколок души]   [Осколок души]
```

### ✨ Осколок души (`soulstaffmod:soul_shard`)
- Используется как ингредиент для крафта Посоха душ.

**Крафт:**
```
[Песок душ]        [Песок душ]           [Песок душ]
[Песок душ]        [Голова визер-скелета][Песок душ]
[Песок душ]        [Песок душ]           [Песок душ]
```

## Текстуры

Текстуры обоих предметов взяты из присланных изображений, обработаны (удалён чёрный фон,
приведены к формату 32×32 с прозрачностью) и лежат тут:
```
src/main/resources/assets/soulstaffmod/textures/item/soul_staff.png
src/main/resources/assets/soulstaffmod/textures/item/soul_shard.png
```

## Структура проекта

```
soulmod/
├── build.gradle
├── gradle.properties
├── settings.gradle
├── LICENSE
└── src/main/
    ├── java/net/soulstaff/mod/
    │   ├── SoulStaffMod.java          — точка входа мода
    │   └── item/
    │       ├── ModItems.java          — регистрация предметов и вкладки
    │       ├── SoulStaffItem.java     — логика посоха (урон, кулдаун, дроп спавнера, запрет чар)
    │       └── SoulShardItem.java     — осколок души (простой предмет)
    └── resources/
        ├── fabric.mod.json
        ├── pack.mcmeta
        ├── assets/soulstaffmod/
        │   ├── lang/{ru_ru,en_us}.json
        │   ├── models/item/{soul_staff,soul_shard}.json
        │   └── textures/item/{soul_staff,soul_shard}.png
        └── data/soulstaffmod/
            ├── recipes/{soul_staff,soul_shard}.json
            └── advancements/recipes/{soul_staff,soul_shard}.json
```

## Как собрать

Gradle Wrapper уже включён в архив — устанавливать Gradle отдельно не нужно.

1. Убедитесь, что установлен **JDK 17** (это единственное, что нужно поставить самостоятельно).
2. В корне проекта выполните:
   ```bash
   ./gradlew build
   ```
   (на Windows — `gradlew.bat build`)
3. Готовый `.jar` появится в `build/libs/soulstaffmod-1.0.0.jar`.
4. Установите **Fabric Loader** и **Fabric API** для Minecraft 1.20.1, затем положите
   собранный jar в папку `mods` вашего клиента/сервера.

Первый запуск займёт несколько минут — Gradle скачает сам Gradle-дистрибутив,
Minecraft, Yarn-маппинги и Fabric API.

## Технические детали реализации

- **Урон и скорость атаки** заданы через переопределение `getAttributeModifiers` —
  это даёт точный контроль над итоговыми значениями (12.0 урона, кулдаун 15 сек = 300 тиков),
  независимо от базовых характеристик выбранного материала.
- **Шанс дропа спавнера 25%** проверяется через `serverWorld.getRandom().nextFloat()`
  перед созданием предмета-спавнера в `postHit`.
- **Запрет зачарований** реализован через `isEnchantable() = false`, `getEnchantability() = 0`
  и `canBeEnchantedWith() = false` — предмет не примет ни одно зачарование ни на столе,
  ни через наковальню/книгу.
- **Дроп спавнера** реализован в `postHit`: после удара, ставшего смертельным, создаётся
  `ItemStack` блока `minecraft:spawner` с записанным в `BlockEntityTag.SpawnData.entity.id`
  типом убитого моба, и этот предмет выбрасывается в мир на месте гибели моба.
