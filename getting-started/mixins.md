---
title: Mixins
description: Mixins are essential for modding in the Fabric ecosystem, enabling the injection of custom code into existing Minecraft classes while maintaining compatibility with other mods.
order: 6
---

# Mixins

::: info
For advanced techniques and in-depth documentation, you can refer to official resources such as the [SpongePowered Mixin Wiki](https://github.com/SpongePowered/Mixin/wiki), join the [Fabric Discord](https://discord.gg/SYaxVkfU3v), or explore the [Fabric Wiki](https://fabricmc.net/wiki/tutorial:mixin_introduction). 

Additionally, [MixinExtras by LlamaLad7](https://github.com/LlamaLad7/MixinExtras) provides useful annotations like `@WrapWithCondition` for further mixin capabilities.
:::

Mixins are a powerful tool that allows mod you to modify existing Minecraft code. By injecting custom logic, removing mechanics, or modifying values, mixins enable a wide range of possibilities for enhancing gameplay and extending functionality.

Fabric Loader loads Mixins for you! You can skip the mixin environment set up and can get straight into creating mixins.

## The `@Mixin` Annotation

The `@Mixin` annotation is a crucial element in defining a class as a mixin. Any properties or methods inherited or contained within the mixin class will be injected into the target class. This provides a seamless way to extend and modify the base game behavior without directly modifying the original Minecraft source code.

Example of using the `@Mixin` annotation:

```java

@Mixin(MinecraftServer.class)
public class ExampleMixin {
    // Custom code to be injected into MinecraftServer class
}
```


## The `@Inject` Annotation

The `@Inject` annotation plays a central role in mixins, allowing you to modify code at specific injection points. By using various parameters such as `at`, `method`, and `slice`, you can control the precise location and conditions of code injections.

Example of using the `@Inject` annotation:

```java

@Inject(at = @At("HEAD"), method = "loadWorld", cancellable = false)
private void init(CallbackInfo info) {
    // Custom code to be injected at the start of MinecraftServer.loadWorld()
}
```


## The `@Shadow` Annotation

The `@Shadow` annotation grants access to fields and methods from the target class within the mixin. This allows you to interact with the target class properties while preserving compile-time type safety without requiring explicit casts.

Example of using the `@Shadow` annotation:

```java

@Mixin(MinecraftServer.class)
public abstract class MinecraftServerMixin {
    @Shadow
    @Final
    public long[] tickTimes;
}
```


## Registering Mixins

Mixins must be registered to be active in the mod. This is accomplished through the `.mixin.json` file, which is located in the `src/main/resources` folder. You need to specify the package, mixins to load, and applicable environments (`client`, `server`, or both).

Sample `.mixin.json` configuration:

```jsonc

{
    "required": true,
    "package": "com.yourmodpackage.mixin",
    "compatibilityLevel": "JAVA_17",
    "injectors": {
        "defaultRequire": 1
    },
    "mixins": [
        "ExampleMixin"
    ],
    "client": [],
    "server": []
}
```


## Additional Mixin Applications

Apart from injecting code, mixins can be used for various other purposes. 

For example, the `@Pseudo` annotation allows mixins to work with other mods without causing crashes if the target is unavailable. 

However, you need to exercise caution with certain annotations like `@Redirect` and `@Overwrite` to ensure compatibility with other mods.