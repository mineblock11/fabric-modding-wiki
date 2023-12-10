---
title: Access Wideners
description: Learn how to access parts of Minecraft's code you're not meant to.
order: 5
---

# Access Wideners

Access wideners are used to remove protections from private, protected or final fields so you can access them.

They can act as an alternative to the `@Accessor` mixin annotation.

## Creating your Access Widener

Your mod can only have one access widener file - create one in your `src/main/resources` folder - it should have the `.accesswidener` extension.

To load the access widener, add the following property to your `fabric.mod.json` file:

```json
"accessWidener" : "file-name-here.accesswidener"
```

You will also need to configure Fabric Loom to load the access widener - or your mod will not compile if you use anything that has been modified.

```gradle
loom {
    accessWidenerPath = file("src/main/resources/file-name-here.accesswidener")
}
```

## Syntax

::: info
**Notice For Minecraft Development IntelliJ Plugin Users**

As of version `1.5.22` you can simply right click any field, method or class and click the `Copy AW Entry` button in the dropdown - you can paste the copied entry into your access widener file.
:::

The access widener file syntax is pretty simple to understand.

### Access Widener Declaration

At the top of the file is the access widener declaration - this must always exist as it tells Fabric Loader how to parse the access widener entries.

The most common declaration is `accessWidener   v1  named`
The declaration format is as follows:

```
accessWidener   {version}   {mappings namespace}
```

Mappings type refers to one of the following:

- `named` - Mappings are named, eg: `TitleScreen`
- `intermediary` - Mappings are not named, eg: `class_2920`

::: warning
The namespace is usually always `named` for normal mods.
:::

### Comments

Comments are supported, the line must start with a hashtag:

```
# This is a comment.
```

### Class Access

To widen access to a class, the following format must be used:

```
{access type} class {class name}
```

Only two access types can be used on class entries:

- `accessible` - Used when you want to access the class from another class.
  + The class itself becomes public.
  + All methods in the class are made public - if the method is private, the method is made final.
  + All fields in the class become public.
- `extendable` - Used when you want to extend the class or override a method.
  + The class itself becomes public and final is removed if it exists on the class.
  + All methods are made protected and final is removed if it exists on the method.

```
# This is an example of a class entry.
accessible class net/minecraft/item/FoodComponent
```

### Method Access

Similarly, to widen access to a method, the following format must be used:

```
{access type}   method   {class name}   {method name}   {method descriptor}
```

Only two access types can be used on method entries:

- `accessible`
- `extendable`

More information on method descriptors [can be found here](https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-4.html#jvms-4.3.3) - usually you can use [Linkie](https://linkie.shedaniel.me/mappings) or the IntelliJ Minecraft Development plugin to automatically generate it for you.

```
# This is an example of a method entry.
# FoodComponent.getHunger() returns integer
accessible method net/minecraft/item/FoodComponent getHunger ()I
```

### Field Access

Finally, to widen access to a field, the following format must be used:

```
{access type}   field   {class name}   {field name}   {field descriptor}
```

Only two access types can be used on field entries:

- `accessible`
- `mutable` - Removes final from the field.
  
If you want to make a field non-final and accessible, you must create two seperate entries:

```
# This is an example of a field entry - the field is private and final.
# MinecraftClient.soundManager
accessible field net/minecraft/client/MinecraftClient soundManager Lnet/minecraft/client/sound/SoundManager;
mutable field net/minecraft/client/MinecraftClient soundManager Lnet/minecraft/client/sound/SoundManager;
```

More information on field descriptors [can be found here.](https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-4.html#jvms-4.3.2)

## Linkie

Linkie is an underrated tool that allows you to browse the classes, fields and methods of all the major mappings - it also lets you generate access wideners and mixin targets:

https://linkie.shedaniel.dev/mappings

![](./_assets/access-wideners_0.png)