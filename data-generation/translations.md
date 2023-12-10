---
title: Translation Provider
description: Learn how to utilize the FabricLanguageProvider class to generate language files.
---

# Translation Provider

The `FabricLanguageProvider` class allows you to generate translations by passing blocks, entities, enchantments and much more.

## Why generate translations?

Translation generation useful for mods with dynamically generated blocks, items or entities. It is also useful when the language file is too large to maintain.

Programmatically generating translations is usually the go-to in their case.

### Programmatic Translation Example

You can often convert any `Identifier` into a human readable string programmatically. This means that whenever you register something to a registry where it's translation is visible in-game, you can likely convert it's registry ID into an actual string.

If `minecraft:orange_flying_boat` is passed into this method, it undergoes the following transformation:

- `minecraft:orange_flying_boat`
- `orange_flying_boat`
- `orange flying boat`
- `Orange Flying Boat`

```java
public static String generateHumanReadable(Identifier identifier) {
    // Remove the namespace, it's irrelevant.
    String identifier_path = identifier.getPath();

    // Replace all '_' with spaces.
    String lowercase = identifier_path.replace("_", " ");

    // Capitalize all words in the string.
    String capitalized = WordUtils.capitalize(lowercase);
    
    return capitalized;
}
```

Many mods use this already in their English language provider classes. Feel free to use it if it is of any interest to you.

## Using the Provider

::: warning
When creating language providers, make sure that the language file you are generating doesn't already exist in your resources folder!<br />
If it does, you will need to rename the file and merge it via the `TranslationBuilder` class.
:::

To use the provider, create a new class that extends the `FabricLanguageProvider` class. In this example, we'll create an English language provider:

```java
public class EnglishTranslationProvider extends FabricLanguageProvider {
    protected EnglishTranslationProvider(FabricDataOutput dataOutput) {
        // Specifying the language code is optional in this case!
        // 'en_us' is always the default.
        super(dataOutput, "en_us");
    }

    @Override
    public void generateTranslations(TranslationBuilder translationBuilder) {
        // ...
    }
}
```

Make sure to register the class in your entrypoint!

```java
@Override
public void onInitializeDataGenerator(FabricDataGenerator fabricDataGenerator) {
    final FabricDataGenerator.Pack pack = fabricDataGenerator.createPack();
            
    // ...
            
    pack.addProvider(EnglishTranslationProvider::new);
}
```

## Generating Translations

The `TranslationBuilder` class can automatically create translation keys from the following classes:

- `Item`
- `Block`
- `ItemGroup`
- `EntityType<?>`
- `Enchantment`
- `EntityAttribute`
- `StatType<?>`
- `StatusEffect`
- `Identifier`

The provider will automatically sort the translation keys in alphabetical order in the output JSON file.

```java
public static final Block PRISMARINE_LAMP = register("mod_id:prismarine_lamp", ...);

@Override
public void generateTranslations(TranslationBuilder translationBuilder) {
  // An example for the Prismarine Lamp block from the blockstates guide.
  translationBuilder.add(PRISMARINE_LAMP, "Prismarine Lamp");
}
```

It can also accept existing language files - this is particularly useful if you wish to merge your existing language files with the final datagen language files:

::: code-group

```java [Provider]
public static final Block PRISMARINE_LAMP = register("mod_id:prismarine_lamp", ...);

@Override
public void generateTranslations(TranslationBuilder translationBuilder) {
  translationBuilder.add(PRISMARINE_LAMP, "Prismarine Lamp"); 

  // Because it's reading from a file, it's recommended you surround it in a try/catch statement.
  try {
    Optional<Path> path = dataOutput.getModContainer().findPath("assets/mod_id/lang/en_us.unmerged.json");
    translationBuilder.add(path.get());
  } catch (Exception e) {
    LOGGER.info("Failed to merge language file: " + e);
  }
} 
```

```jsonc [en_us.unmerged.json]
// assets/mod_id/lang/en_us.unmerged.json
{
  "item.mod_id.poop": "Poop",
  "block.mod_id.condensed_dirt": "Condensed Dirt",
  "itemGroup.mod_id.item_group": "Fabric Community Wiki Items"
}
```

```jsonc [Output JSON]
// generated/assets/mod_id/lang/en_us.json
{
  "block.mod_id.condensed_dirt": "Condensed Dirt",
  "block.mod_id.prismarine_lamp": "Prismarine Lamp",
  "item.mod_id.poop": "Poop",
  "itemGroup.my_mod.item_group": "Fabric Community Wiki Items"
}
```

:::

