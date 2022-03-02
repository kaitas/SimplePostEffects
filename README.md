# SimplePostEffects
keijiro神のURP SimplePostEffectsをみて勉強している

## 環境

- Unity2021.2.13f1

## ShaderGraphを試してみる

InverseをMaterialのShaderGraphのOneMinusで書いているのでRemapしてみたよ！

もともとのレンダリング結果（Test.unity）

![image](https://user-images.githubusercontent.com/5110708/156341418-e457cd01-dc31-42a5-bb9a-28381c2c8f85.png)

![image](https://user-images.githubusercontent.com/5110708/156344781-c5a03082-dc29-4ac7-92f3-7c558ec0c6fb.png)

Asset/Posteffectの ``Invert`` を開く。左はマテリアルで、``Shader Graphs/Invert`` を指定している。Surface Inputとして ``MainTex`` を指定して、これをShaderGraph内で操作するのね。

![image](https://user-images.githubusercontent.com/5110708/156344989-4e53607e-2003-4101-aa98-43efaf37f980.png)


![image](https://user-images.githubusercontent.com/5110708/156341443-bcf22dd8-29d9-46af-8760-33d755242f65.png)

![image](https://user-images.githubusercontent.com/5110708/156341459-46b923db-3a2f-41a5-8501-86e91b496659.png)

![image](https://user-images.githubusercontent.com/5110708/156341493-1efa1272-55e6-4625-a28d-1091b211528d.png)


左上の"Save Asset"を押すのが必要なのに気づかなかった…が、わかればこれは楽しい…。

HDRなShaderも簡単やん

![image](https://user-images.githubusercontent.com/5110708/156341310-de29ece3-f94d-4d60-9fd2-ebb3a8c3252b.png)


## シーンを分解してみる

Camera
- [x] Post Processing
- [x] Dithering
- [ ] Ocullusion Culling (check off)

![image](https://user-images.githubusercontent.com/5110708/156348496-eed2c1fb-679c-4b67-8f08-a0fcc278a18a.png)

カメラの下には ``Postprocess`` という名前のGameobject、``Volume`` → ``Global Volume`` で作成できる。

`Profile` には ``Assets/Misc/Postprocess.asset`` がアタッチされており、

- [x] Bloom
- [x] Tonemapping
- [x] Chromatic Aberration (色収差)
- [x] Shadows Midtones Highlights

が設定されている。


![image](https://user-images.githubusercontent.com/5110708/156345455-c719e6a7-b765-4e7c-b555-31ba98232feb.png)


スクリプトとしては ``UniversalAdditionalCameraData.cs`` 

### PostEffectFeature.cs

```
using UnityEngine;
using UnityEngine.Rendering;
using UnityEngine.Rendering.Universal;

namespace UrpTestbed {

sealed class PostEffectPass : ScriptableRenderPass
{
    public Material material;

    public override void Execute
      (ScriptableRenderContext context, ref RenderingData data)
    {
        if (material == null) return;
        var cmd = CommandBufferPool.Get("PostEffect");
        Blit(cmd, ref data, material, 0);
        context.ExecuteCommandBuffer(cmd);
        CommandBufferPool.Release(cmd);
    }
}

public sealed class PostEffectFeature : ScriptableRendererFeature
{
    public Material material;

    PostEffectPass _pass;

    public override void Create()
      => _pass = new PostEffectPass
           { material = material,
             renderPassEvent = RenderPassEvent.AfterRenderingPostProcessing };

    public override void AddRenderPasses
      (ScriptableRenderer renderer, ref RenderingData data)
      => renderer.EnqueuePass(_pass);
}

} // namespace UrpTestbed
```
