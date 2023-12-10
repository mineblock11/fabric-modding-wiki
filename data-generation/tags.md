---
title: Tag Provider
description: Learn how to utilize the FabricTagProvider class to automatically populate tags with blocks, items, and other content.
---

# Tag Provider

The `FabricTagProvider<T>` class allows you to automatically add content to existing or new tags.

This is useful as it removes the need for you to create the nessecary tag files yourself, alongside any directories - you would also have to manually write the identifiers of all the content you want to add to the tag.

With the tag provider, you can just pass an instance of the object you want to add to the tag.

## Using The Provider

This example will use the `FabricTagProvider.BlockTagProvider` class to generate block tags.

There are other types available to use as well:

- `EnchantmentTagProvider`
- `EntityTypeTagProvider`
- `FluidTagProvider`
- `GameEventTagProvider`
- `ItemTagProvider`

You should create a new class that extends the TagProvider that you want to use:

```java
public class MyBlockTagProvider extends FabricTagProvider.BlockTagProvider {
  public MyBlockTagProvider(FabricDataOutput output, CompletableFuture<RegistryWrapper.WrapperLookup> registriesFuture) {
    super(output, registriesFuture);
  }

  @Override
  protected void configure(RegistryWrapper.WrapperLookup arg) {
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
      
  pack.addProvider(MyBlockTagProvider::new);
}
```

## Adding/Creating Tags

To create a new tag, you need to create a reference to the tag in the form of a `TagKey`:

```java
public static final TagKey<Block> AWESOME_BLOCKS = TagKey.of(
  RegistryKeys.BLOCK, 
  new Identifier("mod_id:awesome_blocks")
);
```

Once you've created the TagKey instance, you can call the `getOrCreateTagBuilder()` method in the `configure` method of your tag provider, passing the instance as the argument or you can use an already existing tag instance instead.

::: code-group

```java [Provider]
@Override
protected void configure(RegistryWrapper.WrapperLookup arg) {
  getOrCreateTagBuilder(AWESOME_BLOCKS)
    .add(Blocks.DIAMOND_BLOCK, ModBlocks.CONDENSED_OAK_LOG)
    .addOptionalTag(BlockTags.DIRT)
    // An optional tag value does not cause the tag to fail when loading if the
    // ID in question doesn't exist in the registry.
    .addOptional(new Identifier("create:cogwheel"))
    // You can add existing tags to your tag, ignoring whether its unloaded or invalid.
    // It's recommened to use addOptionalTag instead of this.
    .forceAddTag(BlockTags.BUTTONS);
}
```

```jsonc [Output JSON]
// Found at generated/data/mod_id/tags/blocks/awesome_blocks.json
{
  "replace": false,
  "values": [
    "minecraft:diamond_block",
    "mod_id:condensed_oak_log",
    {
      "id": "#minecraft:dirt",
      "required": false
    },
    {
      "id": "create:cogwheel",
      "required": false
    },
    "#minecraft:buttons"
  ]
}
```

:::