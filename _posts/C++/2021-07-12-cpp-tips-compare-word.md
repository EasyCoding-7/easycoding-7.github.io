---
layout: post
title:  "(C++ : TIPS) VisualStudio에서 cpp/h 파일 이외의 파일 변화 감지하기 (Ex> .proto)"
summary: ""
author: C++
date: '2021-07-12 0:00:00 +0000'
category: ['Cpp']
#tags: ['C++', 'tag-test1']
thumbnail: /assets/img/posts/cpp-thumnail.png
keywords: ['notify', 'wait']
usemathjax: true
permalink: /blog/cpp/tips/compare-word/
---

.vcproj파일을 편집기에서 연다

```
// ...

    <ClInclude Include="Struct.pb.h" />
  </ItemGroup>
  <ItemGroup>
    <None Include="..\Common\Protobuf\bin\Enum.proto" />
    <None Include="..\Common\Protobuf\bin\GenPackets.bat" />
    <None Include="..\Common\Protobuf\bin\Protocol.proto" />
    <None Include="..\Common\Protobuf\bin\Struct.proto" />
  </ItemGroup>
  <ItemGroup>
    <UpToDateCheckInput Include="..\Common\Protobuf\bin\Enum.proto" />
    <UpToDateCheckInput Include="..\Common\Protobuf\bin\Protocol.proto" />
    <UpToDateCheckInput Include="..\Common\Protobuf\bin\Struct.proto" />
  </ItemGroup>
  <Import Project="$(VCTargetsPath)\Microsoft.Cpp.targets" />
  <ImportGroup Label="ExtensionTargets">
  </ImportGroup>
</Project>
```

```
  <ItemGroup>
    <UpToDateCheckInput Include="..\Common\Protobuf\bin\Enum.proto" />
    <UpToDateCheckInput Include="..\Common\Protobuf\bin\Protocol.proto" />
    <UpToDateCheckInput Include="..\Common\Protobuf\bin\Struct.proto" />
  </ItemGroup>
```

를 추가하면 파일의 변화를 감지후 재 빌드해준다.