---
title: Unity3D Shader学习之五—Shader三种类型
categories:
  - U3D
url: 324.html
id: 324
date: 2016-10-28 14:56:55
tags:
---

Unity Shader有三种类型，分别是固定功能着色器 (Fixed Function Shaders)、表面着色器（Surface Shaders）、顶点和片段着色器。 **固定功能着色器 (Fixed Function Shaders)**适用于不支持可编程着色器的旧硬件，固定功能着色器完全使用 **ShaderLab** 语言（类似于 Microsoft's .FX 文件或 NVIDIA's CgFX）进行编写。 **表面着色器（Surface Shaders）**如需让您的着色器受到光线和阴影的影响，那么表面着色器 (Surface Shaders) 是您最好的选择，表面着色器可让您以简单的方法编写复杂的着色器 - 这是与 Unity 照明管道的更高层次交互提取。大部分表面着色器自动支持正向和延迟照明。可用数行 **Cg/HLSL** 编写表面着色器，且其中将生成更多代码。如果着色器未对光线进行任何处理，请勿使用表面着色器 **顶点和片段着色器**如果着色器无需与照明互动，或者需要制作一些表面着色器无法实现的非凡效果，则需使用顶点和片段着色器。按此方法编写的着色器程序可让您以最灵活的方式制作所需的效果，也可使用 **Cg/HLSL** 编写这些着色器。 固定功能着色器主要针对旧硬件，图形接口API仅支持通过修改配置进行控制，这些完全可以通过编程API来实现，在Unity内部也会把固定功能着色器编译为顶点和片段着色器。 同样的表面着色器最终也会编译为顶点和片段着色器，故对于Unity Shader可以认为只有一种类型，学习时把主要精力放在顶点和片段着色器就行了（个人观点）。 固定功能着色器示例：

Shader "Tutorial/Basic" {
    Properties {
        _Color ("Main Color", Color) = (1,0.5,0.5,1)
    }
    SubShader {
        Pass {
            Material {
              Diffuse \[_Color\]
            }
           Lighting On
        }
    }
}

表面着色器示例

 Shader "Example/Diffuse Simple" {
    SubShader {
      Tags { "RenderType" = "Opaque" }
      CGPROGRAM
      #pragma surface surf Lambert
      struct Input {
          float4 color : COLOR;
      };
      void surf (Input IN, inout SurfaceOutput o) {
          o.Albedo = 1;
      }
      ENDCG
    }
    Fallback "Diffuse"
  }

顶点和片段着色器示例

Shader "VertexInputSimple" {
  SubShader {
    Pass {
      CGPROGRAM
      #pragma vertex vert
      #pragma fragment frag
      #include "UnityCG.cginc"

      struct v2f {
          float4 pos : SV_POSITION;
          fixed4 color : COLOR;
      };

      v2f vert (appdata_base v)
      {
          v2f o;
          o.pos = mul (UNITY\_MATRIX\_MVP, v.vertex);
          o.color.xyz = v.normal * 0.5 + 0.5;
          o.color.w = 1.0;
          return o;
      }

      fixed4 frag (v2f i) : COLOR0 { return i.color; }
      ENDCG
    }
  } 
}

表面着色和顶点和片段着色器的区别是：表面着色器的定义是在SubShader块中，顶点着色器的定义是在Pass中。