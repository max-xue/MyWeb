---
title: Unity3D Shader学习之九—纹理
categories:
  - U3D
url: 620.html
id: 620
date: 2016-11-05 12:11:14
tags:
  - u3d
  - shader
---

纹理使你的网格、粒子、和界面更生动！它们是你覆盖或环绕着对象的图片或影片文件。由于它们是如此的重要，它们有很多的属性。 详情参考官方文档：[Textures](https://docs.unity3d.com/Manual/class-TextureImporter.html) Texture Type 纹理类型：

*   Texture:通常这是适用于所有纹理的最常用设置;
*   **法线纹理**（Normal map):将把颜色通道变成一个适合于实时法向映射的格式;
*   GUI 图形用户界面(已经废弃)：如果你的纹理是要用于任何HUD（平面显示器） / GUI的控制，使用该项;
*   立方图纹理(Cubemap):也称Reflection 反射,用于创建纹理的反射（使用全景图片，创建天空盒）;
*   Cookie:这将设置带基本参数的纹理用于你的光源的Cookie
*   Advanced 高级:当你想要有纹理的具体参数并想拥有纹理的完全控制的时候选择该项。

属性：

*   Wrap Mode:当纹理坐标超出\[0,1\]如何显示 Repeat:纹理重复（平铺）本身； Clamp：超出截取到\[0,1\]纹理的边缘得到延伸（创建天空盒时，纹理设置此选项，否则有空隙）
*   Filter Mode：选择纹理通过三维变换得到拉伸时如何过滤。 Point：纹理在近距离变成块状； Bilinear：纹理在近距离变模糊； Trilinear：类似双线性，但纹理也在不同的mipmap层次之间变模糊。 性能消耗依次增大。
*   Aniso Level 在一个过高角度看纹理时提高纹理质量。适用于地板与地面纹理。
*   Alpha from Grayscale:透明通道的值将会由每个像素的灰度值生成,将依据图像的现有明暗值产生一个alpha透明度通道
*   Bumpiness 凹凸:控制凹凸的总量
*   Filtering 过滤: 确定凹凸如何计算出来 1.Smooth 平滑:这会产生比较平滑的法线贴图; 2.Sharp 锐化:也称为索贝尔过滤。这会产生出比标准更清晰的法线贴图
*   Mapping 映射：这将决定纹理如何映射到一个立方体贴图上 1. Sphere Mapped 球面映射：纹理映射到一个"球状" 立方体贴图上 2. Cylindrical 圆柱：纹理映射到一个圆柱体，当你要使用类似圆柱对象的反射时，使用该项。 3. Simple Sphere 简单球形：纹理映射到一个简单的球形，当你旋转它时变形反射 4.Nice Sphere 精细球形：纹理映射到一个球形，当你旋转它时变形，但你仍然可以看到纹理的外观
*   Light Type 光源类型 纹理将应用的光源类型。（可以是聚光灯、点光源或方向光）。对于方向光纹理将平铺，所以在纹理检视面板中必须设置边缘模式为重复（Repeat）；对于聚光灯，你应该保持你的cookie纹理的边缘为纯黑色，以获得正确的效果。在纹理检视面板中，设置边缘模式为钳制（Clamp）
*   Non Power of 2 不是2的幂：如果纹理大小不是2的幂，这将定义在导入时的缩放行为 1.None 无：纹理将被填充到下一个较大的2的幂大小以便与GUI纹理组件使用 2.To nearest 到最近的：纹理在导入时将被缩放到最近的幂大小。例如257x511纹理将成为256x512。请注意，PVRTC格式要求纹理是正方形（宽度与高度相等），因此最终大小将变换到512x512。 PVRTC是一种有损的纹理压缩技术，主要用于iPhone，iPod touch和iPad。 3.To larger 到较大的：纹理在导入时将被缩放到下一个较大的幂大小。例如257x511纹理将成为512x512。 4.To smaller 到较小的：纹理在导入时将被缩放到下一个较小的幂大小。例如257x511纹理将成为256x256。
*   Generate Cube Map 生成立方贴图：使用不同的生成方法从一个纹理生成一个立方体贴图。
*   Read/Write Enabled 读/写 启用 选择此项将允许从脚本（GetPixels，SetPixels和其他Texture2D函数）访问纹理数据。但是注意，一个纹理数据副本将产生，由此必将为纹理资源消耗双倍的内存量。只有在绝对必要时使用。默认情况下禁用。
*   Generate Mip Maps 生成Mip Maps 选择此项将启用Mipmap生成。当纹理在屏幕上非常小的时候，Mipmaps是可供使用的纹理的较小版本。欲了解更多信息，请参阅下文的Mip maps。
*   Correct Gamma 校正伽马 启用每Mip级别伽玛校正
*   Border Mip Maps 边缘Mip Maps 为了避免色彩渗出到mip较低层次的边缘。用于光源cookies
*   Mip Map Filtering：Mip Map过滤，mip map过滤的两种方式可供优化图像质量： 1. Box 盒：最简单的方式淡出mipmap – 随着尺寸的减小mip级别变得更平滑。 2. Kaiser 凯撒：凯撒算法是随着尺寸的减小锐化mip maps运行。如果你的纹理在远距离变模糊，试试这个选项。
*   Fade Out Mips 淡出Mips 启用此项将使mipmaps随着mip级别的进展褪色为灰色，这个用于细节贴图。最左边的滚动条是开始淡出的第一个mip级别。最右边的滚动条定义mip级别在哪里完全变灰。
*   Generate Normal Map 生成法线贴图 启用此项将转变颜色通道成一个适合于实时法线贴图的格式
*   Normal Map 法线贴图 如果你想了解法线贴图如何被应用到你的纹理，选择此项。
*   Lightmap 光照贴图 如果你想作为光照贴图使用纹理，选择此项。

平台相关：纹理的最大尺寸（Max Size）和格式(Format) [查看文档](http://www.ceeger.com/Manual/Textures.html) 细节：

*   Supported Formats 支持的格式 Unity支持下面的文件格式：PSD, TIFF, JPG, TGA, PNG, GIF, BMP, IFF, PICT。应注意，Unity可以导入多层PSD和TIFF文件，在导入时，层将自动被塌陷，因此你不必浪费时间，直接使用源文件类型。这点很重要，允许只有一个纹理拷贝，使用从Photoshop，三维建模程序导入到Unity。
*   Texture Sizes 纹理大小 这些尺寸如下：2, 4, 8, 16, 32, 64, 128, 256, 512, 1024 或 2048像素。纹理可不必是正方形，即宽度和高度可以不同。 可以使用其他非二次方纹理尺寸，当用于GUI纹理时，非二次方纹理最好，但是，如果在别的使用，它们将被转换为未压缩的RGBA32位格式。这意味着它们将使用更多的内存（相比PVRT(iOS)/DXT(Desktop)压缩的纹理），将使较慢加载和和较慢渲染（如果是iOS模式）。一般来说，非二次方的纹理仅用于GUI。 非二次方纹理资源可以在导入时在导入设置中高级纹理类型使用Non Power of 2（非二次方)选项设置缩放。Unity将按需缩放纹理，并在游戏中，它们的行为就像其他纹理一样，因此他们仍然可以被压缩，加载非常快。

*   UV Mapping（UV贴图） 当映射一个2D纹理到一个3D模型上，要设定循环模式（平铺方式）。这就是三维建模程序中，被称为UV贴图。在Unity，可以使用Materials缩放移动纹理。缩放法线和地形细节贴图尤其有用。 纹理映射坐标：定义了该顶点在纹理中的对应2D坐标，通过用二维变量(u,v)来表示，其中u是横向坐标，v是纵向坐标，也叫UV坐标，在Unity中，原点在左下角(?) 

*   Mip Maps 多级纹理 多级纹理是逐步缩小图像版本的一个列表，用来优化实时3D引擎的性能。远离相机的物体使用较小的纹理版本。使用多级纹理，将多使用33％以上的内存，但不使用它们将有巨大的的性能损失。应该为游戏总是使用多级纹理，唯一例外的是，用于不会缩小的纹理（例如GUI纹理）。多级渐远纹理技术(mipmap)：有点类似于LOD技术，但是不同的是，LOD针对的是模型资源，而Mipmap针对的纹理贴图资源。使用Mipmap后，贴图会根据摄像机距离的远近，选择使用不同精度的贴图。 缺点：会占用内存，因为mipmap会根据摄像机远近不同而生成对应的八个贴图，所以必然占内存！ 优点：会优化显存带宽，用来减少渲染，因为可以根据实际情况，会选择适合的贴图来渲染，距离摄像机越远，显示的贴图像素越低，反之，像素越高！在纹理导入设置中Texture Type选择Advanced，选择Generate Mip Maps。
*   Normal Maps 法线贴图 法线贴图用于法线贴图着色器，使低多边形模型看起来有更多的细节。Unity使用的法线贴图作为RGB图像编码，还可以选择从一个灰度高度图来生成一个法线贴图。
*   Detail Maps 细节贴图 如果想创建一个地形，通常使用主纹理来显示那些草、岩砂区域，等等。如果地形非常大，它最终会非常模糊。细节纹理隐藏实际的淡出，小细节作为更接近的主纹理。 但绘制细节纹理，中性灰是不可见的，白色是主纹理两倍亮，黑色是主纹理完全变黑。
*   Reflections (Cube Maps) 反射（立方体贴图） 如果你想使用纹理用于反射贴图（例如使用内置的反射着色器），必须使用[立方图](http://www.ceeger.com/Components/class-Cubemap.html)纹理。
*   Anisotropic filtering 各项异性过滤 当从掠角（grazing angle）观看，各向异性过滤提高纹理质量，有一些渲染成本消耗（完全是显卡成本）。为地面和地板纹理，增加各向异性等级通常是一个好主意。在质量设置中各向异性过滤，可以强制用于所有纹理或完全禁用。**纹理应用于Shader**
*   单张纹理 单张纹理应用
    
    v2f vert(a2v v) {
    	v2f o;
    	o.pos = mul(UNITY\_MATRIX\_MVP, v.vertex);
    				
    	o.worldNormal = UnityObjectToWorldNormal(v.normal);
    				
    	o.worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;
    				
    	o.uv = v.texcoord.xy * \_MainTex\_ST.xy + \_MainTex\_ST.zw;
    	// Or just call the built-in function
    //o.uv = TRANSFORM\_TEX(v.texcoord, \_MainTex);
    				
    	return o;
    }
    			
    fixed4 frag(v2f i) : SV_Target {
    	fixed3 worldNormal = normalize(i.worldNormal);
    	fixed3 worldLightDir = normalize(UnityWorldSpaceLightDir(i.worldPos));
    				
    	// Use the texture to sample the diffuse color
    	fixed3 albedo = tex2D(\_MainTex, i.uv).rgb * \_Color.rgb;
    				
    	fixed3 ambient = UNITY\_LIGHTMODEL\_AMBIENT.xyz * albedo;
    				
    	fixed3 diffuse = _LightColor0.rgb * albedo * max(0, dot(worldNormal, worldLightDir));
    				
    	fixed3 viewDir = normalize(UnityWorldSpaceViewDir(i.worldPos));
    	fixed3 halfDir = normalize(worldLightDir + viewDir);
    	fixed3 specular = \_LightColor0.rgb * \_Specular.rgb * pow(max(0, dot(worldNormal, halfDir)), _Gloss);
    				
    	return fixed4(ambient + diffuse + specular, 1.0);
    }
    

*   高度纹理 用高度图实现凹凸映射，存在的是强度值。颜色浅越凸，深则凹。
*   法线纹理 模型下的法线纹理：直观、实现简单；边界平滑； 切线空间下的法线纹理：顶点是原点，x轴是切线方向t，y轴是副切线方向b,z轴是法线方向n；自由度高；可实现uv动画；法线纹理可重用；可压缩。 切线空间下计算：
    
    v2f vert(a2v v) {
    	v2f o;
    	o.pos = mul(UNITY\_MATRIX\_MVP, v.vertex);
    				
    	o.uv.xy = v.texcoord.xy * \_MainTex\_ST.xy + \_MainTex\_ST.zw;
    	o.uv.zw = v.texcoord.xy * \_BumpMap\_ST.xy + \_BumpMap\_ST.zw;
    				
    	// Compute the binormal
    //				float3 binormal = cross( normalize(v.normal), normalize(v.tangent.xyz) ) * v.tangent.w;
    //				// Construct a matrix which transform vectors from object space to tangent space
    //				float3x3 rotation = float3x3(v.tangent.xyz, binormal, v.normal);
    	// Or just use the built-in macro
    	TANGENT\_SPACE\_ROTATION;
    				
    	// Transform the light direction from object space to tangent space
    	o.lightDir = mul(rotation, ObjSpaceLightDir(v.vertex)).xyz;
    	// Transform the view direction from object space to tangent space
    	o.viewDir = mul(rotation, ObjSpaceViewDir(v.vertex)).xyz;
    				
    	return o;
    }
    			
    fixed4 frag(v2f i) : SV_Target {				
    	fixed3 tangentLightDir = normalize(i.lightDir);
    	fixed3 tangentViewDir = normalize(i.viewDir);
    				
    	// Get the texel in the normal map
    	fixed4 packedNormal = tex2D(_BumpMap, i.uv.zw);
    	fixed3 tangentNormal;
    	// If the texture is not marked as "Normal map"
    //				tangentNormal.xy = (packedNormal.xy * 2 - 1) * _BumpScale;
    //				tangentNormal.z = sqrt(1.0 - saturate(dot(tangentNormal.xy, tangentNormal.xy)));
    				
    	// Or mark the texture as "Normal map", and use the built-in funciton
    	tangentNormal = UnpackNormal(packedNormal);
    	tangentNormal.xy *= _BumpScale;
    	tangentNormal.z = sqrt(1.0 - saturate(dot(tangentNormal.xy, tangentNormal.xy)));
    				
    	fixed3 albedo = tex2D(\_MainTex, i.uv).rgb * \_Color.rgb;
    				
    	fixed3 ambient = UNITY\_LIGHTMODEL\_AMBIENT.xyz * albedo;
    				
    	fixed3 diffuse = _LightColor0.rgb * albedo * max(0, dot(tangentNormal, tangentLightDir));
    
    	fixed3 halfDir = normalize(tangentLightDir + tangentViewDir);
    	fixed3 specular = \_LightColor0.rgb * \_Specular.rgb * pow(max(0, dot(tangentNormal, halfDir)), _Gloss);
    				
    	return fixed4(ambient + diffuse + specular, 1.0);
    }
    
    世界空间下计算：
    
    v2f vert(a2v v) {
    	v2f o;
    	o.pos = mul(UNITY\_MATRIX\_MVP, v.vertex);
    				
    	o.uv.xy = v.texcoord.xy * \_MainTex\_ST.xy + \_MainTex\_ST.zw;
    	o.uv.zw = v.texcoord.xy * \_BumpMap\_ST.xy + \_BumpMap\_ST.zw;
    				
    	float3 worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;  
    	fixed3 worldNormal = UnityObjectToWorldNormal(v.normal);  
    	fixed3 worldTangent = UnityObjectToWorldDir(v.tangent.xyz);  
    	fixed3 worldBinormal = cross(worldNormal, worldTangent) * v.tangent.w; 
    				
    	// Compute the matrix that transform directions from tangent space to world space
    	// Put the world position in w component for optimization
    	o.TtoW0 = float4(worldTangent.x, worldBinormal.x, worldNormal.x, worldPos.x);
    	o.TtoW1 = float4(worldTangent.y, worldBinormal.y, worldNormal.y, worldPos.y);
    	o.TtoW2 = float4(worldTangent.z, worldBinormal.z, worldNormal.z, worldPos.z);
    				
    	return o;
    }
    			
    fixed4 frag(v2f i) : SV_Target {
    	// Get the position in world space		
    	float3 worldPos = float3(i.TtoW0.w, i.TtoW1.w, i.TtoW2.w);
    	// Compute the light and view dir in world space
    	fixed3 lightDir = normalize(UnityWorldSpaceLightDir(worldPos));
    	fixed3 viewDir = normalize(UnityWorldSpaceViewDir(worldPos));
    				
    	// Get the normal in tangent space
    	fixed3 bump = UnpackNormal(tex2D(_BumpMap, i.uv.zw));
    	bump.xy *= _BumpScale;
    	bump.z = sqrt(1.0 - saturate(dot(bump.xy, bump.xy)));
    	// Transform the narmal from tangent space to world space
    	bump = normalize(half3(dot(i.TtoW0.xyz, bump), dot(i.TtoW1.xyz, bump), dot(i.TtoW2.xyz, bump)));
    				
    	fixed3 albedo = tex2D(\_MainTex, i.uv).rgb * \_Color.rgb;
    				
    	fixed3 ambient = UNITY\_LIGHTMODEL\_AMBIENT.xyz * albedo;
    				
    	fixed3 diffuse = _LightColor0.rgb * albedo * max(0, dot(bump, lightDir));
    
    	fixed3 halfDir = normalize(lightDir + viewDir);
    	fixed3 specular = \_LightColor0.rgb * \_Specular.rgb * pow(max(0, dot(bump, halfDir)), _Gloss);
    				
    	return fixed4(ambient + diffuse + specular, 1.0);
    }
    

*   渐变纹理 实践：
    
    v2f vert(a2v v) {
    	v2f o;
    	o.pos = mul(UNITY\_MATRIX\_MVP, v.vertex);
    				
    	o.worldNormal = UnityObjectToWorldNormal(v.normal);
    				
    	o.worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;
    				
    	o.uv = TRANSFORM\_TEX(v.texcoord, \_RampTex);
    				
    	return o;
    }
    			
    fixed4 frag(v2f i) : SV_Target {
    	fixed3 worldNormal = normalize(i.worldNormal);
    	fixed3 worldLightDir = normalize(UnityWorldSpaceLightDir(i.worldPos));
    				
    	fixed3 ambient = UNITY\_LIGHTMODEL\_AMBIENT.xyz;
    				
    	// Use the texture to sample the diffuse color
    	fixed halfLambert  = 0.5 * dot(worldNormal, worldLightDir) + 0.5;
    	fixed3 diffuseColor = tex2D(\_RampTex, fixed2(halfLambert, halfLambert)).rgb * \_Color.rgb;
    				
    	fixed3 diffuse = _LightColor0.rgb * diffuseColor;
    				
    	fixed3 viewDir = normalize(UnityWorldSpaceViewDir(i.worldPos));
    	fixed3 halfDir = normalize(worldLightDir + viewDir);
    	fixed3 specular = \_LightColor0.rgb * \_Specular.rgb * pow(max(0, dot(worldNormal, halfDir)), _Gloss);
    				
    	return fixed4(ambient + diffuse + specular, 1.0);
    }
    
*   遮罩纹理 实践：
    
    v2f vert(a2v v) {
    	v2f o;
    	o.pos = mul(UNITY\_MATRIX\_MVP, v.vertex);
    				
    	o.uv.xy = v.texcoord.xy * \_MainTex\_ST.xy + \_MainTex\_ST.zw;
    				
    	TANGENT\_SPACE\_ROTATION;
    	o.lightDir = mul(rotation, ObjSpaceLightDir(v.vertex)).xyz;
    	o.viewDir = mul(rotation, ObjSpaceViewDir(v.vertex)).xyz;
    				
    	return o;
    }
    			
    fixed4 frag(v2f i) : SV_Target {
    	fixed3 tangentLightDir = normalize(i.lightDir);
    	fixed3 tangentViewDir = normalize(i.viewDir);
    
    	fixed3 tangentNormal = UnpackNormal(tex2D(_BumpMap, i.uv));
    	tangentNormal.xy *= _BumpScale;
    	tangentNormal.z = sqrt(1.0 - saturate(dot(tangentNormal.xy, tangentNormal.xy)));
    
    	fixed3 albedo = tex2D(\_MainTex, i.uv).rgb * \_Color.rgb;
    				
    	fixed3 ambient = UNITY\_LIGHTMODEL\_AMBIENT.xyz * albedo;
    				
    	fixed3 diffuse = _LightColor0.rgb * albedo * max(0, dot(tangentNormal, tangentLightDir));
    				
    	fixed3 halfDir = normalize(tangentLightDir + tangentViewDir);
    	// Get the mask value
    	fixed specularMask = tex2D(\_SpecularMask, i.uv).r * \_SpecularScale;
    	// Compute specular term with the specular mask
    	fixed3 specular = \_LightColor0.rgb * \_Specular.rgb * pow(max(0, dot(tangentNormal, halfDir)), _Gloss) * specularMask;
    			
    	return fixed4(ambient + diffuse + specular, 1.0);
    }
    
     

参考(部分摘录)：[二维纹理 Texture 2D](http://www.ceeger.com/Manual/Textures.html)