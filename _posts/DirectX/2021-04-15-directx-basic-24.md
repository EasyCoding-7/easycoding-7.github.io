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
    // 한쪽면의 텍스쳐 내에 Normal Vector값이 일정하니 빛 반사를 모두 일정하게 하며 평면적으로 보이게 된다.
    // 아래서 세 번째 요소가 Normal Vector이다
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

여기서 혹시 궁금할꺼 같아 정리를 하자면 Normal-Mapping된 Texture의 색상은 왜 푸른가?<br>
Normal-Mapping Texture의 경우 자체적인 좌표계 tangent좌표계를 사용한다<br>
이 좌표계의 Up이 Blue로 매핑되어 있어 푸르게 나타나며,

자체적인 좌표계를 쓰는 이유는 어느 한 좌표계 예를들어 Local, World 등에 영향을 받을 시<br>
움직임에 따라 빛반사를 하는 것이아니라 해당좌표계에 따라 빛반사를 하기에 어색해 진다.

---

그럼, Normal-Mapping Texture는 어떻게 동작하면 될까?

```
// Normal-Mapping Texture의 Tangent Space(좌표계)에서 View Space로 옮기려면
// 아래 Matrix를 연산해주면된다.
Tx Ty Tz
Bx By Bz
Nx Ny Nz
```

```cpp
shared_ptr<Scene> SceneManager::LoadTestScene()
{
	// ...

    shared_ptr<Shader> shader = make_shared<Shader>();
    shared_ptr<Texture> texture = make_shared<Texture>();
    shared_ptr<Texture> texture2 = make_shared<Texture>();
    shader->Init(L"..\\Resources\\Shader\\default.hlsli");
    texture->Init(L"..\\Resources\\Texture\\Leather.jpg");
    texture2->Init(L"..\\Resources\\Texture\\Leather_Normal.jpg");
    // Normal Texture를 받는다
```

```
cbuffer MATERIAL_PARAMS : register(b2)
{
    int     g_int_0;
    int     g_int_1;
    int     g_int_2;
    int     g_int_3;
    int     g_int_4;
    float   g_float_0;
    float   g_float_1;
    float   g_float_2;
    float   g_float_3;
    float   g_float_4;

    // 쉐이더에서는 Texutre의 null체크가 불가능하기에
    // on이라는 변수를 두어 null인지 아닌지 확인한다.
    int     g_tex_on_0;
    int     g_tex_on_1;
    int     g_tex_on_2;
    int     g_tex_on_3;
    int     g_tex_on_4;
};
```

```cpp
void SetTexture(uint8 index, shared_ptr<Texture> texture) 
{ 
    _textures[index] = texture;
    // texture가 없다면 0 
    // 있다면 1로 넣어달라
    _params.SetTexOn(index, (texture == nullptr ? 0 : 1));
}
```

```
VS_OUT VS_Main(VS_IN input)
{
    VS_OUT output = (VS_OUT)0;

    output.pos = mul(float4(input.pos, 1.f), g_matWVP);
    output.uv = input.uv;

    output.viewPos = mul(float4(input.pos, 1.f), g_matWV).xyz;

    // normal, tangent, binormal을 넣는다
    output.viewNormal = normalize(mul(float4(input.normal, 0.f), g_matWV).xyz);
    output.viewTangent = normalize(mul(float4(input.tangent, 0.f), g_matWV).xyz);
    output.viewBinormal = normalize(cross(output.viewTangent, output.viewNormal));

    return output;
}
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

        // TBN 순서로 생성된 Matrix를 곱해주게 된다.
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
```