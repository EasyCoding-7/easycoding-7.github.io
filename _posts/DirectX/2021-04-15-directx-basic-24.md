---
layout: post
title:  "(DirectX : Basic) 24. Normal-Mapping"
summary: ""
author: DirectX
date: '2021-04-15 0:00:00 +0000'
category: ['DirectX']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/directx-thumnail.jpg
#keywords: ['tutorial']
usemathjax: true
permalink: /blog/DirectX/basic-24/
---

* [GetCode](https://github.com/EasyCoding-7/DirectX-Basic/tree/master/16)

---

![](/assets/img/posts/directx/basic-24-1.png){:class="img-fluid"}

위 그림을 Texture로 사용시 그림에서 보듯 밋밋함이 있다.

![](/assets/img/posts/directx/basic-24-2.png){:class="img-fluid"}

이 그림처럼 입체감을 주는 방법을 Normal-Mapping이라 하는데 어떻게 사용하는지 확인해 보자.

---

일단 문제는 뭘까?

```cpp
shared_ptr<Mesh> Resources::LoadCubeMesh()
{
	//...

	vector<Vertex> vec(24);

	// 앞면
    // 한쪽면의 텍스쳐 내에 Normal값이 일정하니 빛 반사를 모두 일정하게 하며 평면적으로 보이게 된다.
	vec[0] = Vertex(Vec3(-w2, -h2, -d2), Vec2(0.0f, 1.0f), Vec3(0.0f, 0.0f, -1.0f), Vec3(1.0f, 0.0f, 0.0f));
	vec[1] = Vertex(Vec3(-w2, +h2, -d2), Vec2(0.0f, 0.0f), Vec3(0.0f, 0.0f, -1.0f), Vec3(1.0f, 0.0f, 0.0f));
	vec[2] = Vertex(Vec3(+w2, +h2, -d2), Vec2(1.0f, 0.0f), Vec3(0.0f, 0.0f, -1.0f), Vec3(1.0f, 0.0f, 0.0f));
	vec[3] = Vertex(Vec3(+w2, -h2, -d2), Vec2(1.0f, 1.0f), Vec3(0.0f, 0.0f, -1.0f), Vec3(1.0f, 0.0f, 0.0f));

    // ...
```

흠 ... 어떻게 해결하지? 구처럼 Vertex를 여러개라도 둬야하나?? 코드가 너무 복잡해질텐데? 또 효율성은 어떻고??<br>
아니 그냥 Texture이미지 자체에 Normal 값을 저장해두자.

![](/assets/img/posts/directx/basic-24-3.png){:class="img-fluid"}

이 Texture를 읽어서 빛 반사 연산을 하면 간단하지 않나??<br>
이게 Normal-Mapping이다.

---

어떻게 동작하면 될까?

```
struct VS_IN
{
    float3 pos : POSITION;
    float2 uv : TEXCOORD;       // 입력으로 uv값을 받아와 빛을 어떻게 반사할지 계산할때 사용하게 된다.
    float3 normal : NORMAL;
    float3 tangent : TANGENT;
};
```

Vertext Shader에서 넘겨준 uv값을 Reasterizer를 통해 각 Pixel Shader에서 Pixel연산을 하게 되는데 그 때 Normal-Mapping된 이미지의 값을 쓰게 하면 될꺼 같다

굿. 이제 구현만 남음 ... ;;;

---

```cpp
struct MaterialParams
{
	void SetInt(uint8 index, int32 value) { intParams[index] = value; }
	void SetFloat(uint8 index, float value) { floatParams[index] = value; }
	void SetTexOn(uint8 index, int32 value) { texOnParams[index] = value; }

	array<int32, MATERIAL_INT_COUNT> intParams;
	array<float, MATERIAL_FLOAT_COUNT> floatParams;
	array<int32, MATERIAL_TEXTURE_COUNT> texOnParams;   // 텍스쳐를 사용할 것인가 체크
    // 쉐이더 내부에서는 체크가 불가능
};
```

```
float4 PS_Main(VS_OUT input) : SV_Target
{
    float4 color = float4(1.f, 1.f, 1.f, 1.f);
    if (g_tex_on_0)
        color = g_tex_0.Sample(g_sam_0, input.uv);

    float3 viewNormal = input.viewNormal;
    if (g_tex_on_1)
    {
        // [0,255] 범위에서 [0,1]로 변환
        float3 tangentSpaceNormal = g_tex_1.Sample(g_sam_0, input.uv).xyz;
        // [0,1] 범위에서 [-1,1]로 변환
        tangentSpaceNormal = (tangentSpaceNormal - 0.5f) * 2.f;
        float3x3 matTBN = { input.viewTangent, input.viewBinormal, input.viewNormal };
        viewNormal = normalize(mul(tangentSpaceNormal, matTBN));
    }

    LightColor totalColor = (LightColor)0.f;

    for (int i = 0; i < g_lightCount; ++i)
    {
         LightColor color = CalculateLightColor(i, viewNormal, input.viewPos);
         totalColor.diffuse += color.diffuse;
         totalColor.ambient += color.ambient;
         totalColor.specular += color.specular;
    }

    color.xyz = (totalColor.diffuse.xyz * color.xyz)
        + totalColor.ambient.xyz * color.xyz
        + totalColor.specular.xyz;

     return color;
}
````