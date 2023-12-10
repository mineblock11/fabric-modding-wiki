---
title: Creating Particle Factories
description: Learn how to enhance particles with custom particle factories.
prev:
  text: "Creating Custom Particles"
  link: "/rendering/particles/"
---

# Creating Particle Factories

Custom particle factories allow for particles to become super powerful. You can add small rotating leaves to a beautiful scene, or huge, powerful explosions to an intense boss battle. Let's learn about them and make one!

You can think of a particle factory as the particle's code. In this example, we are going to make our smiley faced texture from the Creating Custom Particles tutorial fall slowly, spin, and change size, then stop all movement and fade away upon hitting the ground.

## Implementing the Factory Methods

Let's begin by having our class extend the `SpriteBillboardParticle` class.

```java
public class MyParticle extends SpriteBillboardParticle {
  //Add code here
}
```

You then will need to create the methods required for the particle to function.

```java
public class MyParticle extends SpriteBillboardParticle {
  //Leave at least one line of space here for later

  MyParticle(ClientWorld world, double x, double y, double z, double velX, double velY, double velZ, SpriteProvider spriteProvider) {
    super(world, x, y, z);
    //Particle's constructor method
    //Activated once as soon as the particle spawns
  }

  public void tick() {
    //Tick particle method
    //Activated every single tick the particle is alive
  }

  @Override
  public ParticleTextureSheet getType() {
    //Particle texture sheet method
    //Allows for the particle to determine its texture "type" to a degree
  }
}
```

_Depending on your IDE of choice, the ParticleTextureSheet may have automatically been added and returned a value of "null". If that is the case, then let it be for now._

The code above is for some of the methods required to make your particle have motion. There are still more methods to come, however. But first, let's add some code to the stuff we already have.

## Factory Configuration

In the particle's constructor, there are many fields you can choose to edit to your liking. Here is a list of some of the most common variables you may want to edit.

- `age`: Typically the amount of ticks the particle has been in the world.
- `maxAge`: Typically the total amount of ticks the particle has in the world.
- `scale`: The size of the particle.
- `angle`: The angle of the particle.
- `alpha`: The opacity, or transparency, of the particle.
- `collidesWithWorld`: A boolean which determines if the particle should collide with the world or not.

_The following are typically determined by the method caller._

- `x`: The "x" position of the particle.
- `y`: The "y" position of the particle.
- `z`: The "z" position of the particle.
- `velocityX`: The velocity of the particle in the "x" coordinate direction.
- `velocityY`: The velocity of the particle in the "y" coordinate direction.
- `velocityZ`: The velocity of the particle in the "z" coordinate direction.

Once you have figured out which variables you want to adjust, and which ones you want to leave as default, it is time to continue adding code!

## Configuring the Factory

Back to your particle's constructor, it is finally time to add some code! At this point, you should have some variables in mind for what you want to add. However, if there is a variable you missed, you can always come back and add them!  
As mentioned earlier, our goal is to make a particle which falls slowly, spins, and changes size, then stop all movement and fade away upon hitting the ground. This means we're going to want to have some variables to allow for this, like the `angle`, `alpha`, and `collidesWithWorld` boolean.  
There are also variables we'd just want in general, like the `age` and `scale`, as well as the variables related to positioning.

```java
@Environment(EnvType.CLIENT)
public class MyParticle extends SpriteBillboardParticle {
  //The sprite provider which is used to determine the particle's texture
  private final SpriteProvider spriteProvider;

  MyParticle(ClientWorld world, double x, double y, double z, double velX, double velY, double velZ, SpriteProvider spriteProvider) {
    super(world, x, y, z);
    this.spriteProvider = spriteProvider; //Sets the sprite provider from above to the sprite provider in the constructor parameters
    this.maxAge = 200; //200 ticks = 10 seconds
    this.scale = 0.1f;
    this.velocityX = velX; //The velX from the constructor parameters
    this.velocityY = -0.07f; //Allows the particle to slowly fall
    this.velocityZ = velZ;
    this.x = x; //The x from the constructor parameters
    this.y = y;
    this.z = z;
    this.collidesWithWorld = true;
    this.alpha = 1.0f; //Setting the alpha to 1.0f means there will be no opacity change until the alpha value is changed
    this.setSpriteForAge(spriteProvider); //Required
  }
  ...
}
```

Now, that's a lot of code! Let's go over it a little bit more. Firstly, we need to make sure to set the `this.x`, `this.y`, and `this.z` variables to constructor's respective variables(`x`, `y`, `z`), to ensure the particle spawns at the right position.  
What about the `velocityY` variable? Why did we set that to a predetermined number? Well, our goal is to make it so that our particle falls at the same rate every time. Setting the `velocityY` variable ahead of time helps reduce repeated code. However, if you think you may want to change the `velocityY` variable depending on when you call it, then it wouldn't be a bad idea to change the variable to be determined by the constructor's parameter.

## Adding Code to The Tick Method

There are a lot of ways you could go about doing the code for the tick method. In this example, we will be making it where our `age` variable is the amount of ticks the particle has been in the world, and our `maxAge` variable is the total amount of ticks the particle should be in the world before despawning. This is how most Minecraft particles work.

```java
  public void tick() {
    this.prevPosX = this.x;
    this.prevPosY = this.y;
    this.prevPosZ = this.z;
    this.prevAngle = this.angle; //required for rotating the particle
    if(this.age++ >= this.maxAge || this.scale <= 0 || this.alpha <= 0>) { //Despawns the particle if the age has reached the max age, or if the scale is 0
      this.markDead(); //Despawns the particle
    } else {
      this.setSpriteForAge(this.spriteProvider); //Animates the particle if needed
      this.move(this.velocityX, this.velocityY, this.velocityZ);
    }
  }
```

We have started by adding what is required. A way to despawn the particle, a way to move the particle, and a way to animate the particle. You can comment out the `this.setSpriteForAge` line to remove animation from the particle, and instead have the particle choose one of the textures!  
Next, let's add some life to our particle!

```java
  public void tick() {
    this.prevPosX = this.x;
    this.prevPosY = this.y;
    this.prevPosZ = this.z;
    this.prevAngle = this.angle; //Required for rotating the particle
    if(this.age++ >= this.maxAge || this.scale <= 0 || this.alpha <= 0) { //Despawns the particle if the age has reached the max age, or if the scale is 0, or if the alpha is 0
      this.markDead(); //Despawns the particle
    } else {
      if(!this.onGround) { //If the particle isn't on the ground
        if(this.age >= this.maxAge / 3) {
          this.scale -= 0.02; //Slowly decreases the particle's size
        } else {
          this.scale += 0.02; //Slowly increases the particle's size
        }
        this.angle = this.prevAngle + 0.07f; //Slowly turns the particle
      } else {
        //Stops all velocity movement
        this.velocityX = 0;
        this.velocityZ = 0;

        this.alpha -= 0.05f; //Slowly fades away upon hitting the ground
      }
      this.setSpriteForAge(this.spriteProvider); //Animates the particle if needed
      this.move(this.velocityX, this.velocityY, this.velocityZ);
    }
  }
```

Woah, what's going on here? A lot of simple checks, actually! Let's break down the code.  

First, we are detecting if the particle is off the ground with the `if(!this.onGround)` statement. 

If the particle _isn't_ on the ground, it will continue the statement. The statement continues by slightly changing the particle's angle. Then it asks if the particle has progressed through a third of its age with the `if(this.age >= this.maxAge / 3)` code. If so, then it starts to shrink the particle. Otherwise, it'll continue to grow the particle.  

What if the particle _is_ on the ground? If the particle is on the ground, the particle losses _most_ velocity. It still has the velocityY, because we never set it to 0. We don't need to. 

The reason why we set it to something in the first place is to let it hit the ground. And now that it is on the ground, and because the particle collides with the world, the particle stops the "y" velocity automatically!  

We also begin to reduce the particle's alpha, until it eventually despawns whenever the alpha hits 0, thanks to the `if(this.age++ >= this.maxAge || this.scale <= 0 || this.alpha <= 0)` statement from earlier.

## Texture Sheet

We are getting very close to completing our particle factory but we need to figure out which TextureSheet is best for us. 

Let's go with `PARTICLE_SHEET_TRANSLUCENT`, as it allows for our particle to have some transparency. 

An alternative is `PARTICLE_SHEET_OPAQUE`, which does not allow for any transparency.

```java
  @Override
  public ParticleTextureSheet getType() {
    return ParticleTextureSheet.PARTICLE_SHEET_TRANSLUCENT; //Allows for the texture to have some transparency
  }
```

## The Factory

There is one last method we need to add before testing our particle. It is the method which will be called whenever registering a particle with our factory. Add the following code to the bottom of your particle class.

```java
  @Environment(EnvType.CLIENT)
  public static class Factory implements ParticleFactory<DefaultParticleType> {
    //The factory used in a particle's registry
    private final SpriteProvider spriteProvider;
    public Factory(SpriteProvider spriteProvider) {
      this.spriteProvider = spriteProvider;
    }
    public Particle createParticle(DefaultParticleType defaultParticleType, ClientWorld clientWorld, double x, double y, double z, double velX, double velY, double velZ) {
      return new MyParticle(clientWorld, x, y, z, velX, velY, velZ, this.spriteProvider);
    }
  }
```

Once you are done adding that, your particle code is complete! **But wait!** Don't load up your game yet, as you still need to register your particle using this new particle factory.

## Using The New Factory

In your `ClientModInitializer` class, edit your particle register to include our new particle factory.

```java
ParticleFactoryRegistry.getInstance().register(ModParticles.MY_PARTICLE, MyParticle.Factory::new);
```

Now that your particle factory is being used in the particle registry, you can load up your game and test out the particle!

## Final Result

Try spawning the particle a block in the air, 5 blocks in the air, and 10 whole blocks in the air!

![](./_assets/creating_particle_factories_1.png)  

![](./_assets/creating_particle_factories_2.png)

## Full Example

```java
@Environment(EnvType.CLIENT)
public class MyParticle extends SpriteBillboardParticle {
  //The sprite provider which is used to determine the particle's texture
  private final SpriteProvider spriteProvider;

  MyParticle(ClientWorld world, double x, double y, double z, double velX, double velY, double velZ, SpriteProvider spriteProvider) {
    super(world, x, y, z);
    this.spriteProvider = spriteProvider; //Sets the sprite provider from above to the sprite provider in the constructor method
    this.maxAge = 200; //20 ticks = 1 second
    this.scale = 0.1f;
    this.velocityX = velX; //The velX from the constructor parameters
    this.velocityY = -0.07f; //Allows the particle to slowly fall
    this.velocityZ = velZ;
    this.x = x; //The x from the constructor parameters
    this.y = y;
    this.z = z;
    this.collidesWithWorld = true;
    this.alpha = 1.0f; //Setting the alpha to 1.0f means there will be no opacity change until the alpha value is changed
    this.setSpriteForAge(spriteProvider); //Required
  }

  public void tick() {
    this.prevPosX = this.x;
    this.prevPosY = this.y;
    this.prevPosZ = this.z;
    this.prevAngle = this.angle; //Required for rotating the particle
    if(this.age++ >= this.maxAge || this.scale <= 0 || this.alpha <= 0) { //Despawns the particle if the age has reached the max age, or if the scale is 0, or if the alpha is 0
      this.markDead(); //Despawns the particle
    } else {
      if(!this.onGround) { //If the particle isn't on the ground
        if(this.age >= this.maxAge / 3) {
          this.scale -= 0.02; //Slowly decreases the particle's size
        } else {
          this.scale += 0.02; //Slowly increases the particle's size
        }
        this.angle = this.prevAngle + 0.07f; //Slowly turns the particle
      } else {
        //Stops all velocity movement
        this.velocityX = 0;
        this.velocityZ = 0;

        this.alpha -= 0.05f; //Slowly fades away upon hitting the ground
      }
      this.setSpriteForAge(this.spriteProvider); //Animates the particle if needed
      this.move(this.velocityX, this.velocityY, this.velocityZ);
    }
  }

  @Override
  public ParticleTextureSheet getType() {
    return ParticleTextureSheet.PARTICLE_SHEET_TRANSLUCENT; //Allows for the texture to have some transparency
  }

  @Environment(EnvType.CLIENT)
  public static class Factory implements ParticleFactory<DefaultParticleType> {
    //The factory used in a particle's registry
    private final SpriteProvider spriteProvider;
    public Factory(SpriteProvider spriteProvider) {
      this.spriteProvider = spriteProvider;
    }
    public Particle createParticle(DefaultParticleType defaultParticleType, ClientWorld clientWorld, double x, double y, double z, double velX, double velY, double velZ) {
      return new MyParticle(clientWorld, x, y, z, velX, velY, velZ, this.spriteProvider);
    }
  }
}
```
