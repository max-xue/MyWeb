---
title: Unity3D Shader学习之六—顶点和片段着色器
categories:
  - U3D
url: 327.html
id: 327
date: 2016-10-31 17:05:47
tags:
  - u3d
  - shader
---

基于上一篇文章的内容，将重点放在顶点和片段着色器。 下面是顶点和片段着色器的基本结构：

//着色器的名称，将出现在编辑器中所列出的材质检视器列表中
Shader "MyShaderName" {
//着色器可以有一系列属性。着色器中定义的任何属性都显示在 Unity 内的材质检视器中。典型属性有物体颜色和纹理，或者只是着色器使用的任意值。
Properties {
// ... properties here ...
    }
//数目不定，但至少要有一个，Unity会选择最适合的SubShader运行到指定平台
SubShader {
// ... subshader for graphics hardware A ...
//数据不定，但至少要有一个，通道 (Pass) 块使对象的几何结构被渲染一次
Pass {
// ... pass commands ...
        }
// ... more passes if needed ...
    }
SubShader {
// ... subshader for graphics hardware B ...
    }
// ... Optional fallback ...
//如果所有子着色器都无法在该硬件上运行，就运行此着色器，异常处理
FallBack "VertexLit"
}

各部分详细说明： Name： Shader "_name_" **{** \[Properties\] Subshaders \[Fallback\] **}** 定义一个着色器 可以支持/，在编辑中以目录的形式展示，例如：

Shader "Test/ShaderTest" {}

Properties： 属性 (Properties) { _Property_ \[_Property ..._\] }定义属性块。括号内，多个属性定义如下。

*   _name_ ("_display name_", **Range** (_min_, _max_)) = _number_定义浮点属性，代表检视器中从_最小值 (min)_ 到_最大值 (max)_的滑块。
*   _name_ ("_display name_", **Color**) = (_number_,_number_,_number_,_number_)定义颜色属性。
*   _name_ ("_display name_", **2D**) = "_name_" { _options_ }定义二维纹理属性。
*   _name_ ("_display name_", **Rect**) = "_name_" { _options_ }定义矩形（非 2 的幂）纹理属性。
*   _name_ ("_display name_", **Cube**) = "_name_" { _options_ }定义立方体贴图纹理属性。
*   _name_ ("_display name_", **Float**) = _number_定义浮点属性。
*   _name_ ("_display name_", **Vector**) = (_number_,_number_,_number_,_number_)定义四分量向量属性。

如果您想在着色器程序中访问那些属性中的一些属性，您需要声明一个具有相同名称和匹配类型的 Cg/HLSL 变量。例如，以下这些着色器属性：

_MyColor ("Some Color", Color) = (1,1,1,1) 
_MyVector ("Some Vector", Vector) = (0,0,0,0) 
_MyFloat ("My float", Float) = 0.5 
_MyTexture ("Texture", 2D) = "white" {} 
_MyCubemap ("Cubemap", CUBE) = "" {} 

将在 Cg/HLSL 代码中被声明为：

fixed4 _MyColor; // 精度低的类型对颜色来说已足够
float4 _MyVector;
float _MyFloat; 
sampler2D _MyTexture;
samplerCUBE _MyCubemap;

SubShader: Subshader **{** \[_Tags_\] \[_CommonState_\] _Passdef_ \[_Passdef ..._\] **}**定义子着色器为可选的标记、普通状态和一系列通道定义。 Pass: Pass **{** _\[Name and Tags\] \[RenderSetup\] \[TextureSetup\]_ **}**基本通道命令包含了渲染设置命令的可选列表，后面可跟随一系列纹理以供使用。 Pass结构是这样的，Cg 程序代码片段被编写在 _CGPROGRAM_ 和 _ENDCG_ 之间。

 Pass {
      // ...常用的通道状态设置...

      CGPROGRAM
      // 该代码片段的编译指令，例如：
      #pragma vertex vert
      #pragma fragment frag

      // Cg 代码自身

      ENDCG
      // ...通道设置的剩余部分...
  }

FallBack: Fallback"name"回退到有给定名称的着色器。 Fallback Off 回退关闭,明确表示即使没有子着色器可以在该硬件上运行，也不会有回退以及应打印的警告。 顶点着色器示例：

Shader "VertexInputSimple" {
  SubShader {
    Pass {
      CGPROGRAM
      #pragma vertex vert
      #pragma fragment frag
      #include "UnityCG.cginc"

      //vertex to fragment
      struct v2f {
          float4 pos : SV_POSITION;
          fixed4 color : COLOR;
      };

      //appdata_base 包含application to vertex参数
      v2f vert (appdata_base v)
      {
          v2f o;
          o.pos = mul (UNITY\_MATRIX\_MVP, v.vertex);
          o.color.xyz = v.normal * 0.5 + 0.5;
          o.color.w = 1.0;
          return o;
      }

      //
      fixed4 frag (v2f i) : COLOR0 { return i.color; }
      ENDCG
    }
  } 
}

看示例中的蓝色部分： 1.#pragma vertex vert #pragma指令，指定vertex函数为vert,常用的指令有：

*   #pragma vertex _名称_ \- 将函数_名称_编译为顶点着色器。
*   #pragma fragment _名称_ \- 将函数_名称_编译为片元着色器。
*   #pragma geometry _名称_ \- 将函数_名称_编译为 DX10 几何结构着色器。要使这个选项自动打开 #pragma target 4.0
*   #pragma hull _名称_ \- 将函数_名称_编译为 DX11 外壳着色器。要使这个选项自动打开 #pragma target 5.0
*   #pragma domain _名称_ \- 将函数_名称_编译为 DX11 域着色器。要使这个选项自动打开 #pragma target 5.0

2.#include "UnityCG.cginc" Unity可包含引入预定义变量和帮助函数的文件，其他内置文件有：

*   `HLSLSupport.cginc`\- （自动包含） 用于跨平台着色器编译的帮助宏和定义。
*   `UnityCG.cginc`\- 常用的全局变量和帮助函数。
*   `AutoLight.cginc`\- 光照和阴影功能，例如，表面着色器在内部使用此文件。
*   `Lighting.cginc`\- 标准表面着色器光照模型；当您编写表面着色器时自动将其包含。
*   `TerrainEngine.cginc`\- 用于地形 (Terrain) 和植被 (Vegetation) 着色器的帮助函数。

3.appdata_base 是一个数据结构，提供顶点数据给 Cg/HLSL 顶点程序，几个常用的顶点结构定义在UnityCG.cginc 包含文件，大多数情况下只使用它们就足够了。其它结构有：

*   appdata_base: 顶点由位置、法线和一个纹理坐标构成。
*   appdata_tan: 顶点由位置、切线、法线和一个纹理坐标构成。
*   appdata_full: 顶点由位置、切线、法线、两个纹理坐标以及颜色构成

结构成员必须是下面列出的成员： float4 vertex 是顶点位置 float3 normal 是顶点法线 float4 texcoord 是第一个 UV 坐标 float4 texcoord1 是第二个 UV 坐标 float4 tangent 是切线向量（用于法线贴图） float4 color 是逐顶点颜色

顶点着色器是如何知道这些成员的含义的呢？是由成员定义时后跟冒号加字符串指定的！这些字符串就是语义。例如appdata_base定义如下（请查看UnityCG.cginc源文件）：

 struct appdata_full {
 float4 vertex : POSITION;//模型空间的顶点位置
 float4 tangent : TANGENT;//模型空间的顶点切线
 float3 normal : NORMAL;//模型空间的顶点法线
 float4 texcoord : TEXCOORD0;//顶点的第一组纹理坐标
 float4 texcoord1 : TEXCOORD1;//顶点的第2组纹理坐标
 float4 texcoord2 : TEXCOORD2;//顶点的第3组纹理坐标
 float4 texcoord3 : TEXCOORD3;//顶点的第4组纹理坐标
#if defined(SHADER\_API\_XBOX360)
 half4 texcoord4 : TEXCOORD4;//顶点的第5组纹理坐标
 half4 texcoord5 : TEXCOORD5;//顶点的第6组纹理坐标
#endif
 fixed4 color : COLOR;//顶点颜色
};

4.SV\_POSITION，COLOR0 上面提到了语义，SV\_POSITION也是语义。但这个是顶点到片段着色器的语义。顶点到片段着色器的语义列表：

SV_POSITION：裁剪空间坐标，必须包含
COLOR0:输出第一组顶点颜色，可选
COLOR1:输出的第二组颜色，可选
TEXCOORD0-TEXCOORD7:输出纹理坐标，可选

5.UNITY\_MATRIX\_MVP Unity 内置的空间变换矩阵，位于HLSLSupport.cginc，上面提到了是自动包含的，所以着色器可以直接使用。其他变换矩阵及其含义，在下一篇会详细介绍！ 要想了解每个部分的详细含义及内容，请查看官方文档。