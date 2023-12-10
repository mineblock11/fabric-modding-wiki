---
title: Loot Table Provider
description: Learn how to generate loot table JSON using data generation.
---

# Loot Table Provider

The loot table provider is super convenient if you forget to create loot tables for your modded blocks! It opens many options when generating loot tables:

- If your mod doesn't have any blocks with non-special loot tables, you can iterate all your blocks and quickly generate generic loot tables.
- You can create complex loot tables that would take too long to create using JSON.
  
## Using The Provider

First, you must create a class that extends `FabricBlockLootTableProvider`:

```java
public class MyBlockLootTableProvider extends FabricBlockLootTableProvider {
    private MyBlockLootTableProvider(FabricDataOutput output) {
        super(output);
    }

    @Override
    public void generate() {
        // ...
    }
}
```

Don't forget to register your provider in your mod entrypoint:

```java
@Override
public void onInitializeDataGenerator(FabricDataGenerator fabricDataGenerator) {
  final FabricDataGenerator.Pack pack = fabricDataGenerator.createPack();
            
  // ...
            
  pack.addProvider(MyBlockLootTableProvider::new);
}
```

## Simple Loot Tables

If you want to generate a loot table that simply drops blocks when a block is mined/blew up you can use the `addDrop` and `drops` methods in the `generate` method of your provider.

It is assumed you have a block item automatically generated, if not, you can pass an alternative item into the `drops` method.

::: code-group

```java [Provider]
public static final Block CONDENSED_DIRT = register("mod_id:condensed_dirt", ...);

@Override
public void generate() {
  addDrop(CONDENSED_DIRT, drops(CONDENSED_DIRT.asItem()))
}
```

```jsonc [Output JSON]
// generated/data/mod_id/loot_tables/blocks/condensed_dirt.json
{
  "type": "minecraft:block",
  "pools": [
    {
      "bonus_rolls": 0.0,
      "conditions": [
        {
          "condition": "minecraft:survives_explosion"
        }
      ],
      "entries": [
        {
          "type": "minecraft:item",
          "name": "mod_id:condensed_dirt"
        }
      ],
      "rolls": 1.0
    }
  ]
}
```

:::

Here is an example if you don't have a block item registered for your block (or you dont want the block item to drop)

::: code-group

```java [Provider]
public static final PillarBlock CONDENSED_OAK_LOG = register("mod_id:condensed_oak_log", ...);
public static final Item POOP = register("mod_id:poop", ...);

@Override
public void generate() {
    addDrop(CONDENSED_OAK_LOG, drops(POOP));
}

```

```jsonc [Output JSON]
// generated/data/mod_id/loot_tables/blocks/condensed_oak_log.json
{
  "type": "minecraft:block",
  "pools": [
    {
      "bonus_rolls": 0.0,
      "conditions": [
        {
          "condition": "minecraft:survives_explosion"
        }
      ],
      "entries": [
        {
          "type": "minecraft:item",
          "name": "mod_id:poop"
        }
      ],
      "rolls": 1.0
    }
  ]
}
```

:::

## Complex Loot Tables

If you want to add some randomness or bias to your loot tables, you can pass an instance of the `LootTable.Builder` class instead.

You can use common premade builders via the `BlockLootTableGenerator` class - this is a vanilla class and contains methods for silk touch, fortune, ect.

This example drops a diamond if the player breaks the block with an item with the silk touch enchantment, but drops the normal condensed dirt block if the item does not have the enchantment.

::: code-group

```java [Provider]
public static final Block CONDENSED_DIRT = register("mod_id:condensed_dirt", ...);

@Override
public void generate() {
  // Gives a positive result if the item has silk touch on it.
  LootCondition.Builder HAS_SILK_TOUCH = MatchToolLootCondition.builder(ItemPredicate.Builder.create()
    .enchantment(new EnchantmentPredicate(Enchantments.SILK_TOUCH, NumberRange.IntRange.ANY)));

  LootTable.Builder builder = LootTable.builder().pool(
    LootPool.builder()
    // If silk touch is used, drop a diamond.
    .with(ItemEntry.builder(Items.DIAMOND)
      .conditionally(HAS_SILK_TOUCH)
      .apply(SetCountLootFunction.builder(ConstantLootNumberProvider.create(1))))

    // If silk touch is not used (invert of with silk touch), drop the normal block.
    .with(ItemEntry.builder(ModBlocks.CONDENSED_DIRT.asItem()).conditionally(HAS_SILK_TOUCH.invert()))
  );

  addDrop(ModBlocks.CONDENSED_DIRT, builder);
}
```

```jsonc [Output JSON]
{
  "type": "minecraft:block",
  "pools": [
    {
      "bonus_rolls": 0.0,
      "entries": [
        {
          "type": "minecraft:item",
          "conditions": [
            {
              "condition": "minecraft:match_tool",
              "predicate": {
                "enchantments": [
                  {
                    "enchantment": "minecraft:silk_touch"
                  }
                ]
              }
            }
          ],
          "functions": [
            {
              "add": false,
              "count": 1.0,
              "function": "minecraft:set_count"
            }
          ],
          "name": "minecraft:diamond"
        },
        {
          "type": "minecraft:item",
          "conditions": [
            {
              "condition": "minecraft:inverted",
              "term": {
                "condition": "minecraft:match_tool",
                "predicate": {
                  "enchantments": [
                    {
                      "enchantment": "minecraft:silk_touch"
                    }
                  ]
                }
              }
            }
          ],
          "name": "mod_id:condensed_dirt"
        }
      ],
      "rolls": 1.0
    }
  ]
}
```

:::
