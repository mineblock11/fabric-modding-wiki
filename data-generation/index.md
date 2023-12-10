---
title: Data Generation Setup
description: This page will show you how to setup your project with Fabric API's data generation API.
---

# Data Generation Setup

Data Generation is used by many mods, and mojang themselves, to quickly create JSON files such as blockstates, translations and other various files that can be found in the `assets/<namespace>/data` folder of your project.

## Why should you use Data Generation?

Although Data Generation is not really quicker for small projects with 1-2 JSON files, when you start expanding your mod and adding more blocks, biomes, dimensions etc. you might find yourself overwhelmed with JSON files. 

Data Generation ensures those JSON files are kept in the correct format between game versions, and is easier to edit. It also allows you to create complex algorithms that would have been impossible to do by hand, eg: generating 100s of variants of a model due to different combinations of materials used to craft a block.

## What are Data Providers?

Data Providers are where the actual data is consumed and converted into JSON, some examples include:

- `FabricTranslationProvider`
- `FabricAdvancementProvider`
- `FabricCodecProvider`

A full list of providers that Fabric API provides *hah pun!* can be found [here on their GitHub repository.](https://github.com/FabricMC/fabric/tree/1.20.1/fabric-data-generation-api-v1/src/main/java/net/fabricmc/fabric/api/datagen/v1/provider)

Vanilla also has it's own providers, from the `ModelProvider` to the bare-bones `DataProvider` which can be used to produce custom JSON files.

## Creating the Loom run configuration

Loom doesn't create the data generation run configuration by default, so you'll need to create it in your `build.gradle` like so:

::: info
Make sure to replace `<mod-id-here>` with your mod's ID!
:::

```groovy
// Add the datagen output to the jar resources.
sourceSets {
    main {
        resources {
            srcDirs += [
                    'src/main/generated'
            ]
        }
    }
}

// Create the datagen runs.
loom {
    // ...
    runs {
        datagenClient {
            client()
            name "Data Generation Client"
            vmArg "-Dfabric-api.datagen"
            vmArg "-Dfabric-api.datagen.output-dir=${file("src/main/generated")}"
            vmArg "-Dfabric-api.datagen.modid=<mod-id-here>"

            ideConfigGenerated = true
            runDir "build/datagen"
        }
    }
    // ...
}
```

## Creating the Entrypoint

You'll need to create a class that implements the `DataGeneratorEntrypoint` interface, this is an entrypoint, similar to the `ModInitializer` and `ClientModInitializer` entrypoints.

It's recommended you keep this in a sub-package called `datagen` to stay organized.

```java
public class MyModDatagen implements DataGeneratorEntrypoint {
    @Override
    public void onInitializeDataGenerator(FabricDataGenerator fabricDataGenerator) {
 
    }
}
```

As with all entrypoints, you'll need to register it in your `fabric.mod.json` file.

```json
{
    // ...
    "entrypoints": {
        "fabric-datagen": [
            "com.examplemod.datagen.MyModDatagen"
        ],
    },
    // ...
}
```

## Testing

Before continuing, it is really helpful to quickly test the Data Generation entrypoint without any providers registered, this can prevent you scratching your head in the future when you don't realize you have messed up the configuration.

To check if you have implemented Data Generation correctly into your project, run the `runDatagenClient` command, after it has completed, the `src/main/generated` folder should appear.

## Registering Data Providers

To register **any** provider, you must first create a `Pack` instance using the `fabricDataGenerator.createPack();` method in your entrypoint:

```java
@Override
public void onInitializeDataGenerator(FabricDataGenerator fabricDataGenerator) {
    final FabricDataGenerator.Pack pack = dataGenerator.createPack();

    // ...
}
```

::: info
You should check the sidebar for a full list of providers, the `MyModSoundProvider` does not exist, and is used here for example purposes.
:::

This pack can be used to register (add) providers, any providers added to the pack will be executed during data generation. You can then call the `addProvider` method to add providers.

```java
final FabricDataGenerator.Pack pack = dataGenerator.createPack();

// MyModSoundProvider will be executed on data generation, and should produce a lovely sound.json file!
pack.addProvider(new MyModSoundProvider(fabricDataGenerator));
```