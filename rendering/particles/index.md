---
title: Creating Custom Particles
description: Learn how to add and use a new particle with the registry.
next:
  text: "Creating Particle Factories"
  link: "/rendering/particles/factories"
---

# Creating Custom Particles

Particles are a powerful tool if used correctly. It can add ambience to a beautiful scene, or add tension to an edge of your seat boss battle. Let's add one!

## Register a custom particle


To add a custom particle to your mod, you first need to register a ParticleType in your mod initializer class. 

```java
public class MyMod implements ModInitializer {

  public static final String MOD_ID = "mod_id";
  public static final DefaultParticleType MY_PARTICLE = FabricParticleTypes.simple();

  @Override
  public void onInitialize() {
    Registry.register(Registries.PARTICLE_TYPE, new Identifier(MOD_ID, "my_particle"), MY_PARTICLE);
  }
}
```

The "my_particle" in lowercase letters is the json path for the particle's texture. You will be creating a new json file with that exact name later.

## Client-side registration

After you have registered the particle in the `ModInitializer` entrypoint, you will also need to register the particle in the `ClientModInitializer` entrypoint.

```java
public class MyModClient implements ClientModInitializer {
    
    @Override
    public void onInitializeClient() {
        ParticleFactoryRegistry.getInstance().register(MyMod.MY_PARTICLE, EndRodParticle.Factory::new);
    }
}
```

In this example, we are registering our particle client-side. We are then giving it some life with the end rod particle's factory. This means that the particle will behave exactly like the end rod particle would. You can replace the particle's factory with another particle's factory, or even your own particle factory!

## Creating a json file and adding textures

You will need to create 3 folders in your resources folder.

Let's begin with creating the folders necessary for the particle's texture(s). Add the new `resources/assets/<mod id here>/textures/particle` folders to your directory. Place the particle's textures that you want to use in the `particle` folder.

For this example, we will only be adding one texture, named "myparticletexture.png". It'll be a simple pixel art smiley face.

<div align="center">

![](./_assets/creating_particles_0.png)

<a href="./_assets/creating_particles_0_small.png" target="_blank">Download Texture</a>

</div>

In the `resources/assets/<mod id here>/particles` folder, create a new json file named `my_particle.json` - this contains the textures that your particle will use.

In this file, you will need some code to help Minecraft know which texture(s) to put onto your particle.

```json
{
  "textures": [
    "<mod id here>:myparticletexture"
  ]
}
```

You can add more textures to animate the particle as well!

```json
{
  "textures": [
    "<mod id here>:myparticletexture1",
    "<mod id here>:myparticletexture2",
    "<mod id here>:myparticletexture3"
  ]
}
```

## Testing the new particle

Once you have completed the json file and saved your work, you are good to go! Load up minecraft and test everything out! 

You can see if everything has worked by typing the command `/particle <mod id here>:my_particle ~ ~1 ~`. The particle will spawn inside the player with this command, so you may need to walk backwards to actually see it. You can also use a command block to summon the particle with the exact same command!

![](./_assets/creating_particles_1.png)
