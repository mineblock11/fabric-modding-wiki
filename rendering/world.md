---
title: Worldspace Rendering
description: Learn how to render content in the world.
prev:
  text: "Matrices"
  link: "/rendering/matrices"
---

# Worldspace Rendering

::: warning
This page assumes you have a basic understanding of [matrices.](./_assets/matrices)
:::

## What's the challenge?

Lets say you want to mark a location in the world. How are you going to do that?

You could look for existing implementations for this in the minecraft source code, but techniques like that are sparse and don't show up often. That's why this tutorial exists.

## How?

Let's break the problem down first. "We want to draw content in the world" can be simplified down to "We want to render content relative to the camera's look angle", and that already sums it up perfectly: We want to draw content relative to where the camera is looking.

When thinking about the problem, the camera's yaw and pitch might come into mind. In fact, that's what we're going to be using. Assuming that the player is always at the world origin (0, 0, 0), and that the content around the player moves instead of the player himself, we can easily figure out what we need to do:
1. Multiply the Matrix to face into the right direction
2. Render the content relative to the camera's position

## Actually implementing

First, we need an event listener to actually render anything. I'm using `WorldRenderEvents.END` here:

```java
WorldRenderEvents.END.register(context -> {
    // see below
});
```

Next, we will re-use the example from the [Introduction To Rendering](/rendering) page:

```java
WorldRenderEvents.END.register(context -> {
    Matrix4f positionMatrix = matrixStack.peek().getPositionMatrix();
    Tessellator tessellator = Tessellator.getInstance();
    BufferBuilder buffer = tessellator.getBuffer();

    buffer.begin(VertexFormat.DrawMode.QUADS, VertexFormats.POSITION_COLOR_TEXTURE);
    buffer.vertex(positionMatrix, 20, 20, 0).color(1f, 1f, 1f, 1f).texture(0f, 0f).next();
    buffer.vertex(positionMatrix, 20, 60, 0).color(1f, 0f, 0f, 1f).texture(0f, 1f).next();
    buffer.vertex(positionMatrix, 60, 60, 0).color(0f, 1f, 0f, 1f).texture(1f, 1f).next();
    buffer.vertex(positionMatrix, 60, 20, 0).color(0f, 0f, 1f, 1f).texture(1f, 0f).next();

    RenderSystem.setShader(GameRenderer::getPositionColorTexProgram);
    RenderSystem.setShaderTexture(0, new Identifier("examplemod", "icon.png"));
    RenderSystem.setShaderColor(1f, 1f, 1f, 1f);

    tessellator.draw();
});
```

At this point you've probably noticed that the matrixStack no longer exists. We could either use the one from the context (for ease of use), but we'll make our own this time, to walk through what actually happens:

```java
// ...
MatrixStack matrixStack = new MatrixStack();
// TODO: Modify matrixStack to place element properly
Matrix4f positionMatrix = matrixStack.peek().getPositionMatrix();
// ...
```

If you now run this code, nothing seems to appear..?

Correct, our code still has issues that need fixing. The first of those being that the matrixStack isn't transformed correctly. To fix that, we have to rotate it:

```java
// ...
Camera camera = context.camera();
MatrixStack matrixStack = new MatrixStack();
matrixStack.multiply(RotationAxis.POSITIVE_X.rotationDegrees(camera.getPitch()));
matrixStack.multiply(RotationAxis.POSITIVE_Y.rotationDegrees(camera.getYaw() + 180.0F));
// ...
```

We've seen `RotationAxis` before, we're just rotating the rendered content around the X axis based on the pitch, then following that up with rotating it by the yaw **+ 180** on the Y axis. But why + 180?

The yaw ranges from -180 to +180. We need to normalize it prior to actually rendering the content. When we add 180 to the yaw, we get a range from 0 to 360, instead of -180 to +180.

Applying this, we still don't see anything. What's wrong this time?

Well, we haven't really specified any position for the content to render in. Currently, it's just rendering the image 20 blocks above the camera at all times, perfectly perpendicular. That's why you don't see it.

To fix this, let's start by specifying where we actually want to render the image:

```java
// ...
Camera camera = context.camera();
Vec3d targetPosition = new Vec3d(0, 100, 0);
// ...
```

This indicates that we want to render at (0, 100, 0). To actually apply this position tho, we need to do the following things:
1. Subtract the camera's position from the target position
2. Transform the Matrix with that new delta

We need to subtract the camera's position, because we're rendering relative to it. We need to calculate the position delta between the camera and the target render position in the world to correcly position the element relative to the camera.

Adding those 2 things:

```java
// ...
Vec3d targetPosition = new Vec3d(0, 100, 0);
Vec3d transformedPosition = targetPosition.subtract(camera.getPos());
// ...
matrixStack.multiply(RotationAxis.POSITIVE_Y.rotationDegrees(camera.getYaw() + 180.0F));
matrixStack.translate(transformedPosition.x, transformedPosition.y, transformedPosition.z);
// ...
```

<Callout type="info">
Matrix transformations are subject to transformation by previous entry.
</Callout>

After this, we have to change the vertex coordinates from absolute coordinates to relative coordinates, to render it correctly (and make it easier to work with):

```java
buffer.vertex(positionMatrix, 0, 0, 0).color(1f, 1f, 1f, 1f).texture(0f, 0f).next();
buffer.vertex(positionMatrix, 0, 1, 0).color(1f, 0f, 0f, 1f).texture(0f, 1f).next();
buffer.vertex(positionMatrix, 1, 1, 0).color(0f, 1f, 0f, 1f).texture(1f, 1f).next();
buffer.vertex(positionMatrix, 1, 0, 0).color(0f, 0f, 1f, 1f).texture(1f, 0f).next();
```

Replacing the old vertex calls with these ones, then running the code will finally render something at (0, 100, 0)! But there are still some issues. First of all, why is it upside down?
![](./_assets/world_0.png)

The Y coordinate space on the hud is from top to bottom, increasing as you go down. In the world however, this is flipped upside down (literally): Instead of (for example) 0-100 top to bottom, we have 100-0 top to bottom. But when we flip our Y coordinates in the vertex calls, it disappears..?

That is the 2nd issue: Culling.

You've probably heard about it, it basically prevents faces from rendering when their vertecies are in clockwise order. It exists to prevent faces facing away from the camera from rendering, to save performance.

That is not what we want tho, so we need to disable it with `RenderSystem.disableCull();`. Don't forget to re-enable it again with `RenderSystem.enableCull();` after `tessellator.draw();`!

After doing that, our image finally renders correctly:
![](./_assets/world_1.png)

There are a few remaining minor issues, such as the image not rendering through water, only inside of it and outside of it:
![](./_assets/world_2.png)

And the fact that it doesn't render through walls at all.

This might be undesireable for some applications, but luckily for you, you can fix that as well. Just prefix `tessellator.draw();` with `RenderSystem.depthFunc(GL11.GL_ALWAYS);`, and postfix it with `RenderSystem.depthFunc(GL11.GL_LEQUAL);`, to reset it.

Why this works is out of the scope of this tutorial, but a tldr is that it basically disables the depth check while rendering our element. Resetting it back to `LEQUAL` (short for "less than or equal") will restore the default behaviour of not rendering pixels if their depth value is not less than or equal to the one already in the buffer.

After applying those changes as well, this is how it looks:
![](./_assets/world_3.png)

## Full Example

```java
WorldRenderEvents.END.register(context -> {
    Camera camera = context.camera();
    
    Vec3d targetPosition = new Vec3d(0, 100, 0);
    Vec3d transformedPosition = targetPosition.subtract(camera.getPos());
    
    MatrixStack matrixStack = new MatrixStack();
    matrixStack.multiply(RotationAxis.POSITIVE_X.rotationDegrees(camera.getPitch()));
    matrixStack.multiply(RotationAxis.POSITIVE_Y.rotationDegrees(camera.getYaw() + 180.0F));
    matrixStack.translate(transformedPosition.x, transformedPosition.y, transformedPosition.z);
    
    Matrix4f positionMatrix = matrixStack.peek().getPositionMatrix();
    Tessellator tessellator = Tessellator.getInstance();
    BufferBuilder buffer = tessellator.getBuffer();

    buffer.begin(VertexFormat.DrawMode.QUADS, VertexFormats.POSITION_COLOR_TEXTURE);
    buffer.vertex(positionMatrix, 0, 1, 0).color(1f, 1f, 1f, 1f).texture(0f, 0f).next();
    buffer.vertex(positionMatrix, 0, 0, 0).color(1f, 0f, 0f, 1f).texture(0f, 1f).next();
    buffer.vertex(positionMatrix, 1, 0, 0).color(0f, 1f, 0f, 1f).texture(1f, 1f).next();
    buffer.vertex(positionMatrix, 1, 1, 0).color(0f, 0f, 1f, 1f).texture(1f, 0f).next();

    RenderSystem.setShader(GameRenderer::getPositionColorTexProgram);
    RenderSystem.setShaderTexture(0, new Identifier("examplemod", "icon.png"));
    RenderSystem.setShaderColor(1f, 1f, 1f, 1f);
    RenderSystem.disableCull();
    RenderSystem.depthFunc(GL11.GL_ALWAYS);

    tessellator.draw();

    RenderSystem.depthFunc(GL11.GL_LEQUAL);
    RenderSystem.enableCull();
});
```