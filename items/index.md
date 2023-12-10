---
title: Creating Your First Item
description: Learn how to register a simple item and how to texture, model and name it.
next:
  text: "Food Items"
  link: "/items/food"
---

# Creating Your First Item

This page will introduce you into some key concepts relating to items, and how you can register, texture, model and name them.

If you aren't aware, everything in Minecraft is stored in registries, and items are no exception to that.

## Preparing your items class

To simplify the registering of items, you can create a method that accepts an instance of an item and a string identifier.

You can put this method in a class called `ModItems` (or whatever you want to name the class!). 

Mojang does this with their items as well! Check out the `Items` class for inspiration.

```java
public class ModItems {
    // We can use generics to make it so we dont need to 
    // cast to an item when using this method.
    public static <T extends Item> T register(T item, String ID) {
        // Create the identifier for the item.
        Identifier itemID = new Identifier("mod_id", ID);
        
        // Register the item.
        T registeredItem = Registry.register(Registries.ITEM, itemID, item);

        // Return the registered item!
        return registeredItem;
    }
}
```

## Registering an item

You can now register an item using the method now.

The item constructor takes in an instance of the `Items.Settings` class as a parameter. This class allows you to:

- Assign the item's `ItemGroup`.
- Make the item edible by passing a `FoodComponent`.
- Set the item's durability.
- and other miscelaneous properties.

```java
// Registers a blank item with the ID of "modid:poop"
// I use FabricItemSettings - for the sake of it.
// You can use Item.Settings if you wish.
public static final Item POOP = register(
    new Item(new FabricItemSettings()), 
    "poop"
);
```

However, when you go in-game, you can see that our item doesn't exist! This is because you don't statically initialize the class.

To do this, you can add a public static initialize method to your class and call it from your `ModInitializer` class. Currently, this method doesn't need anything inside of it.

```java
public static void initialize() {}
```

```java
public class MyMod implements ModInitializer {
    @Override
    public void onInitialize() {
        // Statically initialize the class.
        ModItems.initialize();
    }
}
```

Calling a method on a class statically initializes it if it hasn't been used before - this means that all `final` fields are evaluated.

## Adding the item to an item group

::: info
The traditional way to register an Item to an ItemGroup was to pass it to the Item.Settings instance, but this has changed in 1.19.3 because you can now specifically order items!
:::

You can append your item into the ingredients item group - a later page will describe how to create your own item group.

To add to the group, you will need to use Fabric API's item group events - specifically `ItemGroupEvents.modifyEntriesEvent`

This can be done in the `initialize` method of your items class.

```java
public static void initialize() {
    ItemGroupEvents
        // Register a "modify" event for the Ingredients item group.
        .modifyEntriesEvent(ItemGroups.INGREDIENTS)
        // Add the item to the group when you get access to it.
        .register((itemGroup) -> itemGroup.add(ModItems.POOP));
}
```

Loading into the game, you can see that our item has been registered, and is in the Ingredients item group:

![](./_assets/first_item_0.png)

However, it's missing the following:

- Item Model
- Texture
- Translation (name)

## Naming the item

The item currently doesn't have a translation, so you will need to add one. The translation key has already been provided by Minecraft: `item.mod_id.poop`

Create a new JSON file at: `src/main/resources/assets/<mod id here>/lang/en_us.json` and put in the translation key and it's value:

```json
{
    "item.mod_id.poop": "Poop"
}
```

You can either restart the game or build your mod and press <kbd>F3 + T</kbd> to apply changes.

## Adding a texture and model

The item will still have the placeholder purple and black checkerboard texture until you create a texture and model for it.

To do this, place a 16x16 texture in the `assets/<mod id here>/textures/item` folder that has the same name of the item you've just registered.

For the example, the texture is called `poop.png`; I just used the poop emoji as a texture for now.

<div align="center">

![](./_assets/first_item_1.png)

<a target="_blank" href="./_assets/first_item_1_small.png">
Download Texture
</a>

</div>

When restarting/reloading the game - you should see that the item still has no texture, that's because you will need to add a model that uses this texture.

You're going to create a simple `item/generated` model, which takes in an input texture and nothing else.

Create the model JSON in the `assets/<mod id here>/models/item` folder, with the same name as the item; `poop.json`

```json
{
    "parent": "item/generated",
    "textures": {
        "layer0": "mod_id:item/poop"
    }
}
```

If you now reload the game (<kbd>F3</kbd> + <kbd>T</kbd>) you can now see your item texture!

![](./_assets/first_item_2.png)