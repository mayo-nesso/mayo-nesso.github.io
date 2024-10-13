---
title: "[LiP] Metaballs in Unity URP with Custom Render Passes"
weight: 1
tags: ["Unity", "URP", "Metaballs", "Shaders"]
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
canonicalURL: "https://mayonesso.com/blog/2024/09/metaballs/"
disableHLJS: true
disableShare: true
hideSummary: false
summary: "Metaballs using shaders and Custom Render Passes, following URP 17. API"
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: false
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: false
UseHugoToc: true
cover:
    image: "cover.webp"
    # alt: "alt text..."
    # caption: "caption..."
    relative: false
    hidden: false
---

## • Introduction

Metaballs! Yeah, like meatballs, but no meat.

Before jumping into that, I recall, years ago, when a colleague at the office gave a small talk about Fragment Shaders.</br>
At that time, I thought I'd be better off deciphering the Necronomicon (including searching for it in the local library).

Today, my almost absurd and preferred simplification to explain the concept is that it's just a function that returns 4 values, which turn out to be the color of the pixel to be drawn (Red, Green, Blue, and Alpha).

Of course, this is a partial simplification of something more complex than that, but why scare innocent people too early?</br>
All this fits within the world of 'how to draw beautiful things on the screen'.</br>
And this post is precisely about that.

Well, maybe not so beautiful things, but some basic attempts.

## • Metaballs: What are they?

{{< figure
    src="imgs/1_metaballs.gif"
    alt="AI lavalamp effect"
    align="center"
    width="400"
>}}

This brings us back to Metaballs: What are they? The simplest and most direct way is: 'do you remember those lava lamps?'

Those viscous balls that would join and then separate in a somewhat animal-like way?

Well, Metaballs are about that effect, and this article, like many others, is about how to achieve it.

## • How to 'achieve' the effect

Simplifying, I've seen two approaches out there on how to do it.</br>
The mathematical way, and the slightly trickier way.

{{< figure
    src="imgs/2_Math_Lady_meme.jpg"
    alt="Math Lady meme"
    align="center"
    caption="Math what?"
    link="https://en.wikipedia.org/wiki/Math_Lady"
    target="_blank"
    width="400"
>}}

The first approach, more elegant, follows a technique called "Marching Cubes".</br>
It was published in 1987 in a paper called "Marching Cubes: A High Resolution 3D Surface Construction Algorithm".

In a simplified view, it uses information from circumferences and the vertices of a grid of cubes in space to draw.</br>
Draw what? Well, whatever needs to be drawn.

{{< figure
    src="imgs/3_Marching_Squares.png"
    alt="Marching Squares Image"
    align="center"
    caption="Marching Squares (2D version)"
    link="https://en.wikipedia.org/wiki/Marching_squares"
    target="_blank"
    width="600"
>}}

I know that it sounds strange, but let's say on one side we have a grid with sensors at each vertex, and on the other side our balls.</br>
(Plus, a ball can be over multiple sensors).

Whenever a ball is over a sensor, the sensor will be *active*, then, taking a set of sensors, we will draw lines according to which sensors are active in that set. And so on until we complete our entire grid.

Easier to say than done?</br>
[This is an excellent resource to get a grasp of it](https://jurasic.dev/marching_squares/)</br>
(this is the 2D version, squares instead of cubes)

In the second approach, slightly more ingenious tricks are used to calculate or compose the effect of 'closeness' or 'influence', and according to this, we draw!

Specifically, we can use elements that have an opaque color with a semi-transparent gradient border.</br>
Now we can use the alpha values (which indicate how opaque or transparent the color is) to achieve the the final effect.</br>
So, in a post-drawn process, we define some alpha thresholds to draw our Metaballs.</br>
We can define more than one level, so we can have one color for the body and another for the border.

{{< figure
    src="imgs/4_split.png"
    align="center"
    width="250"
>}}

In the case of a single element, there's nothing very interesting to see.</br>
But when two bodies get close, their respective semi-transparent borders will join forces to 'saturate' the alpha channel until they reach the threshold that indicates that we have to draw *something* on the screen.

{{< figure
    src="imgs/5_close.png"
    align="center"
    width="250"
>}}

In summary;

- Marching cubes: 1987 paper called 'Marching Cubes: A High Resolution 3D Surface Construction Algorithm'
- Calculate or draw closeness/influence, paint body and contours according to it

## • Some implementations / Tutorials

In the first approach:

- 1.a. [This tutorial of Jamie Wong is quite clear and illustrative](https://jamie-wong.com/2014/08/19/metaballs-and-marching-squares/)

    Note that this technique works for 3D and also for 2D, hence the name "Marching Cubes" or "Marching Squares".

In the second category, I share with you these four implementations;

- 2.a. [A cool tutorial from Daniel Lilett](https://danielilett.com/2020-03-28-tut5-2-urp-metaballs/) where he uses the metaball positions and radius to calculate the 'closeness' and to draw the 'area of influence'; then 'apply the colors according to it' on a render pass

- 2.b. [A smart and tricky approach where Artjoms Neimanis](https://patomkin.com/blog/metaball-tutorial/) uses the Blur effect to obtain the 'alpha' channel, overlapping balls will saturate the 'area of influence' and an extra camera applies the colors according to the rules on a shader in a dedicated render texture

- 2.c. [An evolved version of the previous one by HuvaaKoodia](https://github.com/HuvaaKoodia/2DMetaballs) here we avoid the use of an extra camera and the render texture, applying the effect on a render pass

- 2.d. [And another tutorial from Bronson Zgeb](https://bronsonzgeb.com/index.php/2021/03/20/pseudo-metaballs-with-scriptable-renderer-features-in-unitys-urp/) similar to the previous one, Blur + URP render pass!

## • So, why another one?

With Marching, there are some limitations, you can't just put different colors on the edges, just 'contours' (Although with some work I think it could be done, for instance; having a second system that would draw a second pass with a more adjusted 'contour' level?).

On the others, I would like to be able to avoid blur or to have to pass information about the location of each metaball on each frame.

And that's the reason for this post; here I'm not coming to sell but to give away.

My idea is to remove the blur, using the alpha trick (which we will achieve with a shader) and then use a render pass with the coloring rules using another shader, similar to the last 2 posts, but this time we will use RenderGraph, which according to the Unity people, is what's in vogue these days...

### Why does the blur bother me?

The blur effect is calculated by copying, moving, and overlaying a semi-transparent version of the texture, giving the desired effect.</br>
And this is done over multiple 'passes'.</br>
On low-end devices, this can negatively affect your frame rate (especially if you're doing a blur on every frame!), so it's sometimes good to look for alternatives.

{{< figure
    src="imgs/6_google_questions.png"
    align="center"
    width="800"
>}}

## • So, what do we need?

- Unity 6
- A Universal Render Project (Since we're in Unity6, the URP version is 17.03)

## • First step: Shader for Circles

Instead of blur, we are going to use a shader to draw our Metaballs with an opaque body and a gradual semi-transparent border (more opaque to the center, more transparent to the 'outside').

The idea is to define which part of our circle will be solid and which part will be our 'area of influence', that is, when two balls are close, their areas of influence will be added to make the linking effect appear.

For this, we have the following code. Pretty simple, but it works.

```c
Shader "Custom/GradientCircle"
{
    Properties
    {
        _Color ("Color", Color) = (1.0, 1.0, 1.0, 1)
        _Radius ("Radius", Range(0, 1)) = 1.0
        _Smoothness ("Smoothness", Range(0, 1)) = 0.8
    }
    
    SubShader
    {
        Tags {
            "RenderType"="Transparent" 
            "RenderPipeline"="UniversalPipeline" 
            "Queue"="Transparent"
        }
        LOD 100
        Blend SrcAlpha OneMinusSrcAlpha
        ZWrite Off
        
        Pass
        {
            HLSLPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

            struct Attributes
            {
                float4 positionOS : POSITION;
                float2 uv : TEXCOORD0;
            };

            struct Varyings
            {
                float2 uv : TEXCOORD0;
                float4 positionCS : SV_POSITION;
            };


            CBUFFER_START(UnityPerMaterial)
                float4 _MainTex_ST;
                float4 _Color;
                float _Radius;
                float _Smoothness;
            CBUFFER_END

            Varyings vert(Attributes IN)
            {
                Varyings OUT;
                OUT.positionCS = TransformObjectToHClip(IN.positionOS.xyz);
                OUT.uv = IN.uv;
                return OUT;
            }

            half4 frag(Varyings IN) : SV_Target
            {
                // Normalize UV coordinates to range from -1 to 1
                const float2 uv = IN.uv * 2.0 - 1.0;

                // Circle center (in normalized coordinates)
                const float2 center = float2(0.0, 0.0);

                // Distance from current point (uv) to circle center
                const float dist = length(uv - center);

                // Smooth gradient: 1.0 at center, 0.0 at edge
                // The smoothstep function produces a smooth transition between 0.0 and 1.0
                // based on the distance from the circle center and the defined radius and smoothness
                const float smooth_circle = smoothstep(_Radius, _Radius - _Smoothness, dist);

                // The final alpha value is the smooth gradient value
                half4 color = _Color;
                color.a *= smooth_circle;
                
                return color;
            }
            ENDHLSL
        }
    }
}
```

</br>
We create a new material with this shader and, voilà:

{{< figure
    src="imgs/7_gradient.png"
    align="center"
    caption="We have a nice radial gradient"
    width="400"
>}}

{{< figure
    src="imgs/8_parameters.png"
    align="center"
    width="500"
    caption="`Radius` and `Smoothness` are some parameters that we can adjust,</br> we also have `Color` but it is not that useful"
>}}

## • Second step: URP Custom Pass | Render Feature

Once we have all the spheres on the screen, we are ready to draw the effect.</br>

{{< figure
    src="imgs/9_a_lot_of_spheres.png"
    align="center"
    width="400"
>}}

To do this, we will create a second shader and a custom render pass. Well, strictly speaking two passes.

This second shader will draw depending on the amount of alpha present, in this way, we can paint the body of the Metaballs and the border.

This time, we will use *Shader Graph*, and we are going to call it `MetaEffect`:

{{< figure
    src="imgs/10_shader_graph.png"
    align="center"
    link="imgs/10_shader_graph.png"
    target="_blank"
    caption="*...click me!*"
>}}

With these settings (The important part: `Material` = `Fullscreen`):

{{< figure
    src="imgs/11_shader_graph_settings.png"
    align="center"
    width="400"
>}}

And with the following properties:

{{< figure
    src="imgs/12_shader_graph_properties.png"
    align="center"
    width="300"
>}}

The `Border Color` and `Body Color` parameters represent the respective colors of the border and the body.</br>
The `Min Alpha Threshold` defines the minimum alpha value that will be considered as part of the border, while the `Body Threshold` specifies the minimum alpha value that will be classified as part of the body.</br>
Any alpha value that falls between the `Min Alpha Threshold` and the `Body Threshold` will be regarded as part of the border.

Now, for our custom render pass!

In the first pass, we draw our semi-transparent balls on a temporary texture.</br>

There are some caveats though.</br>
On one hand, we don't want to show our original balls, just the effect of it.</br>
And we don't want to apply this to everything that is on the screen, only to our balls.</br>

To do this, in our render pass we will work only with a specific layer, so we will create a `FilteringSettings` to only select a specific layer during the process:

```csharp
// Settings to filter which renderers should be drawn
private readonly FilteringSettings _filterSettings = default;
...

_filterSettings = new FilteringSettings(renderQueueRange, layerMask);
...

private void InitRendererLists(ContextContainer frameData, ref LayerRenderPassData layerRenderPassData, RenderGraph renderGraph)
{
    // Access the relevant frame data from the Universal Render Pipeline
    var universalRenderingData = frameData.Get<UniversalRenderingData>();
    var cameraData = frameData.Get<UniversalCameraData>();
    var lightData = frameData.Get<UniversalLightData>();
    var sortingCriteria = cameraData.defaultOpaqueSortFlags;
    
    // Create drawing settings based on the shader tags and frame data
    var drawSettings = RenderingUtils.CreateDrawingSettings(_shaderTagIdList, universalRenderingData, cameraData, lightData, sortingCriteria);
    
    // Create renderer list parameters,
    // here we are using _filterSettings, and that is where we specified the layerMask to use
    var param = new RendererListParams(universalRenderingData.cullResults, drawSettings, _filterSettings);
    
    // Finally create a RenderListHandle 
    layerRenderPassData.RendererListHandle = renderGraph.CreateRendererList(param);
}

```

Then our first pass would look like this:

```csharp
private static void ExecuteLayerRenderPass(LayerRenderPassData data, RasterGraphContext context)
{
    // Draw all renderers in the list
    context.cmd.DrawRendererList(data.RendererListHandle);
}
...

// Set up the layer render pass
const string layerRenderPassName = "Mat2Layer: Layer Render 1/2";
using (var builder = renderGraph.AddRasterRenderPass<LayerRenderPassData>(layerRenderPassName, out var passData))
{
    InitRendererLists(frameData, ref passData, renderGraph);
    builder.UseRendererList(passData.RendererListHandle);
    
    // Set up texture dependencies
    // We are not really using 'srcCamColor' on this pass,
    // but we are going to keep the next line for clarity and documentation... 
    builder.UseTexture(srcCamColor); 
    builder.SetRenderAttachment(temporaryHandle, 0);
    builder.SetRenderAttachmentDepth(srcCamDepth);
    
    builder.SetRenderFunc((LayerRenderPassData data, RasterGraphContext context) => ExecuteLayerRenderPass(data, context));
}
```

> Here is worth note that we follow the structure of:</br>
> a.- setting a source (with `builder.UseTexture(srcCamColor)`),</br>
> b.- setting the destination (with `SetRenderAttachment(temporaryHandle, 0)` and `SetRenderAttachmentDepth(srcCamDepth)`),</br>
> c.- doing some work (with `builder.SetRenderFunc`)</br>
>
> But, as you can see in the comments, we are not really using `srcCamColor` (ie; 'what is rendered in the camera so far'), instead we are instructing our pass to use our `FilteringSettings` defined onto `RendererListHandle`, in this line: `builder.UseRendererList(passData.RendererListHandle);`
</p>

In the second pass, we apply the second shader to what was drawn in the previous pass (stored in our temporary texture handle called `temporaryHandle`) with `BlitTexture`.</br>
And we draw the result on the screen.

Something like this:

```csharp
private static void ExecuteBlitPass(BlitPassData data, RasterGraphContext context)
{
    // Blit the source texture to the current render target using the specified material
    Blitter.BlitTexture(context.cmd, data.Source, ScaleBias, data.Material, 0);
}
...

// Set up the blit pass
const string blitPassName = "Mat2Layer: Blit Pass 2/2";
using (var builder = renderGraph.AddRasterRenderPass<BlitPassData>(blitPassName, out var passData))
{
    // Configure pass data
    passData.Material = _material;
    // Use the output of the previous pass as the input
    passData.Source = temporaryHandle;
    builder.UseTexture(passData.Source);
    
    // Set the render target to the original color buffer
    builder.SetRenderAttachment(srcCamColor, 0);
    builder.SetRenderAttachmentDepth(srcCamDepth);
    
    builder.SetRenderFunc((BlitPassData data, RasterGraphContext context) => ExecuteBlitPass(data, context));
}
```

> All the details are in [`MaterialToLayerRenderPass.cs`](https://github.com/mayo-nesso/urp-metaballs/blob/main/Assets/Scripts/MaterialToLayerRenderPass.cs) take it a look, there are a lot of comments!

What is left is to create a new Layer, and assign our balls to it.</br>
Then, we will unselect that Layer from the rendering process (since we are going to take care of the drawing itself)

## • Unity Scene

In our scene, we will start with a plane to which we will apply the material that we made from our `GradientCircle` shader.

{{< figure
    src="imgs/13_new_plane.png"
    align="center"
    width="400"
>}}

{{< figure
    src="imgs/14_circle_material.png"
    align="center"
    width="400"
>}}

Then, we will place these elements on a new Layer called `Metaballs`.

{{< figure
    src="imgs/15_new_layermask.png"
    align="center"
    width="400"
>}}

{{< figure
    src="imgs/16_set_layer.png"
    align="center"
    width="400"
>}}

On the other hand, we will have our `UniversalRendererData` (We can start with `PC_Renderer`, but remember that if we're going to use this on mobile, then we'll have to modify `MobileRenderer`) here we will first deselect the `Metaballs` layer, with this, the camera will not draw our balls on the screen!

{{< figure
    src="imgs/17_renderer_unselect_layermask.png"
    align="center"
    width="500"
>}}

After that, we add a new render feature; ours!

{{< figure
    src="imgs/18_renderer_add_new_feature.png"
    align="center"
    width="500"
>}}

So we start looking for it:

{{< figure
    src="imgs/19_renderer_search_for_our_pass.png"
    align="center"
    width="300"
>}}

And we add it, and configure it!

{{< figure
    src="imgs/20_add_custom_render_pass.png"
    align="center"
    width="400"
>}}

We configure it by telling it that:

- -`Render Queue` = `All`
- -`LayerMask` = `Metaballs`
- -`Material` = `MetaEffect`
- -`RenderPassEvent` = `After Rendering SkyBox`.

## • Results

So, with one Metaball we should see this;

{{< figure
    src="imgs/21_results_single.png"
    align="center"
    width="300"
>}}

But, more of them;

{{< figure
    src="imgs/22_results_group.png"
    align="center"
    width="300"
>}}

And if we go to the `Frame Debugger` we will see what is happening behind the curtains:

{{< figure
    src="imgs/23_frame_debugger_first_pass.png"
    align="center"
    width="600"
    caption="First pass"
>}}

{{< figure
    src="imgs/24_frame_debugger_second_pass.png"
    align="center"
    width="600"
    caption="Second pass"
>}}

And since we are here, let's take a look at the `Render Graph Viewer`

{{< figure
    src="imgs/25_render_graph_viewer.png"
    align="center"
    width="600"
    caption="Mat2Layer passes 1 and 2..."
>}}

## • Conclusion: Blobs and Beyond

And there you have it. We've ventured from abstract equations to smooth, organic shapes dancing across your screen. Metaballs may seem like simple blobs at first, but behind them lies a world of blending functions, threshold values, and a bit of computational magic.
The next time you see those mesmerizing, fluid visuals in games or animations, remember that it's more than just pixels - it's a mix of math and code working together.
I hope you enjoyed reading this tutorial and, as usual, comments, doubts, questions, suggestions, etc… are all welcome!

## • Source Code

[Here URP Metaballs :)](https://github.com/mayo-nesso/urp-metaballs)
Until next time!
