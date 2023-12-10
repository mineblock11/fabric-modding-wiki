---
title: Block States
description: Learn why blockstates are a great way to add visual functionality to your blocks.
prev:
  text: "Creating Your First Block"
  link: "/blocks/"
next:
  text: "Block Entities"
  link: "/blocks/block-entities"
---

# Block States

A block state is a piece of data attached to a singular block in the Minecraft world containing information on the block in the form of properties - some examples of properties vanilla stores in block states:

- Rotation: Mostly used for logs and other natural blocks.
- Activated: Heavily used in redstone devices, and blocks such as the furnace or smoker.
- Age: Used in crops, plants, saplings, kelp etc.
  
You can probably see why they are useful - they avoid the need to store NBT data in a block entity - reducing the world size, and preventing TPS issues!

Blockstate definitions are found in the `assets/<mod id here>/blockstates` folder.

## Example: Pillar Block

Minecraft has some custom classes already that allow you quickly create certain types of blocks - this example goes through the creation of a block with the `axis` property by creating a "Condensed Oak Log" block.

The vanilla `PillarBlock` class allows the block to be placed in the X, Y or Z axis.

```java
public static final PillarBlock CONDENSED_OAK_LOG = register(
    new PillarBlock(
        AbstractBlock.Settings.create().sounds(BlockSoundGroup.WOOD)
    ), "condensed_oak_log", true
);
```

Pillar blocks have two textures, top and side - they use the `block/cube_column` model.

As always, with all block textures, the texture files can be found in `assets/<mod id here>/textures/block`

<div align="center">

![](./_assets/blockstates_0_large.png)

<a href="./_assets/blockstates_0.zip" target="_blank">
Download Textures
</a>

</div>

Since the pillar block has two positons, horizontal and vertical, we'll need to make two seperate model files:

- `condensed_oak_log_horizontal.json` which extends the `block/cube_column_horizontal` model.
- `condensed_oak_log.json` which extends the `block/cube_column` model.

```json
{
    "parent": "block/cube_column",
    "textures": {
        "end": "mod_id:block/condensed_oak_log_top",
        "side": "mod_id:block/condensed_oak_log"
    }
}
```

The blockstate file is where the magic happens - pillar blocks have three axis, so we'll use these models accordingly.

```jsonc
{
  "variants": {
    "axis=x": {
      "model": "mod_id:block/condensed_oak_log_horizontal",
      // We'll rotate the model so that it faces towards the positive-x direction
      "x": 90,
      "y": 90
    },
    "axis=y": {
      "model": "mod_id:block/condensed_oak_log"
    },
    "axis=z": {
      "model": "mod_id:block/condensed_oak_log_horizontal",
      // Rotate the model so it faces towards the positive-x direction.
      "x": 90
    }
  }
}
```

As always, you'll need to create a translation for your block, and an item model which parents either of the two models.

![](./_assets/blockstates_1.png)

## Custom Block States

Custom block states are great if your block has unique properties - sometimes you may find that your block can re-use vanilla properties.

This example will create a unique boolean property called `activated` - when a player right clicks on the block, the block will go from `activated=false` to `activated=true` - changing it's texture accordingly.

### Creating the property

Firstly, you'll need to create the property itself - since this is a boolean, we'll use the `BooleanProperty.of` method.

```java
public class PrismarineLampBlock extends Block {
    public static final BooleanProperty ACTIVATED = BooleanProperty.of("activated");
    
    // ...
}
```

Next, we have to append the property to the blockstate manager in the `appendProperties` method. You'll need to override the method to access the builder:

```java
@Override
protected void appendProperties(StateManager.Builder<Block, BlockState> builder) {
  builder.add(ACTIVATED);
}
```

You'll also have to set a default state for the `activated` property in the constructor of your custom block.

```java
public PrismarineLampBlock(...) {
  // ... super() etc.

  setDefaultState(getDefaultState().with(ACTIVATED, false));
}
```

### Using the property

This example flips the boolean `activated` property when the player interacts with the block. We can override the `onUse` method for this:

```java
@Override
public ActionResult onUse(BlockState state, World world, BlockPos pos, PlayerEntity player, Hand hand, BlockHitResult hit) {
  // Ignore the interaction if it was run on the client.
  if (world.isClient) {
    return ActionResult.SUCCESS;
  }
        
  // Get the current value of the "activated" property
  boolean activated = state.get(ACTIVATED);
        
  // Flip the value of activated and save the new blockstate.
  world.setBlockState(pos, state.with(ACTIVATED, !activated));

  return ActionResult.SUCCESS;
}
```

### Visualizing the property

Since you created a new property, you will have to update the blockstate file for the block to account for that property.

If you have multiple properties on a block, you'll have to account for all possible combinations, eg: `activated` and `axis` would lead to `6` combinations (two possible values for `activated` and three possible values for `axis`) 

Since this block only has two possible variants - due to the fact it only has one property, `activated` - the blockstate JSON will look something like this:

```jsonc
{
  "variants": {
    "activated=false": { 
      "model": "mod_id:block/prismarine_lamp" 
    },
    "activated=true": { 
      "model": "mod_id:block/prismarine_lamp_activated" 
    }
  }
}
```

![](./_assets/blockstates_2.webp)

<hr />

Since the example block is a lamp, we also need to make it emit light when the activated property is true - this can be done through the block settings passed to the constructor:

```java
AbstractBlock.Settings.of(...)
  .luminance((state) -> {
    // Get the value of the "activated" property.
    boolean activated = state.get(ACTIVATED);

    // Return a light level if activated = true
    return activated ? 15 : 0;
  });
```