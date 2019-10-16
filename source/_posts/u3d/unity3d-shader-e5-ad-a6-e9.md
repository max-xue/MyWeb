---
title: Unity3D Shader学习之八—光照
categories:
  - U3D
url: 612.html
id: 612
date: 2016-11-03 12:06:16
tags:
  - u3d
  - shader
---

部分摘录自：[Unity 5 中的全局光照技术详解](http://www.cocoachina.com/game/20150701/12339.html) **全局光照(GI):是一个用来模拟光的互动和反弹等复杂行为的算法**

*   实时照明(REALTIME LIGHTING)  代表这些灯源会把光线照射到场景并以每一帧的频率更新，由于光源是可以在场景内移动的对象，场景灯光的更新是实时的，你可以在游戏窗口和场景窗口看到改变

*   烘焙全局光照(BAKEDGI LIGHTING）

*   预计算全局光照(PRECOMPUTED REALTIME GI LIGHTING) 能帮我们实时运算复杂的场景光源互动，透过这种方法，就能建立昏暗的环境带有丰富的全局光照反射，并实时反映光源的改变

**着色路径**

*   前向渲染（Forward Rendering） 每个对象著色是根据每个影响对象的光，透过”Pass”来著色，所以有可能一个对象被重复著色了好几次，取决于有几盏灯在作用范围里。 优点：快速，也代表硬件需求低，此外，这种正向著色提供了广泛的自定义”着色模型”，可以快速处理透明度，也支持像是多重采样柔边(MSAA)的硬件功能，等等有些在其他路径上是无法实现的功能，对于图形质量有很大的影响。 缺点：要为每盏灯光付出相对应的成本，也就是说，对象被越多盏灯光影响，花费的运算成本就越高，有些类型的游戏必需要大量的光源，就会令人望之却步，反观如果你能管理好你的灯光数量，那这个路径会是一个非常快速的解决方案。
*   延迟渲染（Deferred） 延迟了光的遮蔽与混合信息直到第一次接收到的表面的位置 法线 以及材质数据著色到一个”几何缓冲器”(G-buffer)作为一个屏幕空间的贴图，最后合成这些结果 优点：照明的著色成本是和像素数量成正比，而非灯光数量，因此你不用再管控场景灯光数量，某些游戏类型将会是一个关键优势。 缺点：呈现可预见的效能特点，但通常需要较强大的硬件，对于手机平台支持度也较低。

**色彩空间(COLOR SPACE)** 色彩空间决定采用哪种算法来计算照明或材质加载时的颜色混合，这会对游戏的画面真实感有很大的影响，但大多数情况下，太超过的色彩空间设定可能会被目标平台的硬件强制限制。

*   Linear 接近真实的色彩空间,优点是会让场景内的提供给着色器的颜色也会因为光强度增加变亮 另一个好处是着色器能在没有Gamma补偿的情况下对贴图进行取样，这有助于确保颜色质量在经过著色管道还能保持一致性，能提高色彩和计算的精度，最后屏幕的输出结果更为真实 缺点：有些手机平台甚至有些游戏机不支持，应该说PC或是一些新手机硬件和次世代游戏机才会支持Linear颜色空间

*   Gamma 缺点：亮度马上会转为以白色做为参考，这将不利于图像的质量

**高动态范围(HDR)：** **色调映射(TONEMAPPING)：** **环境光(AmbientLighting)：** **反射源(REFLECTIONSOURCE)：** **反射探头(REFLECTIONPROBES)：**

###### **光源类型**

*   定向光（DIRECTIONALLIGHTS） 无坐标、只有方向；光线平行、无衰减，适用于模拟太阳光
*   点光源（POINT LIGHTS） 亮度从中心最强一直到范围属性(Range)设定的距离递减到0为止，光的强度从光源到距离成反比；适合用来制作像是灯泡, 武器发光或是从物体发射出来的爆炸效果； 开启阴影运算是很耗效能；预计算GI时，不支持阴影的间接反射；
*   聚光灯（SPOTLIGHTS） 投射一个锥体在他的Z轴前方，这个锥体的宽度由投射角度(Spot Angle)属性控制着，光线会从源头到设定的范围慢慢衰减到0，同时越靠近锥体边缘也会衰减，把投射角度的值加大会让锥体宽度加大，同时也让边缘淡化的力度变大，这现象学名叫做”半影”； 用来模拟路灯, 壁灯,手电筒；预计算GI时，不支持间接光阴影；
*   区域光(AREA LIGHTS) 可以当作是摄影用的柔光灯，在Unity里面他们被定义为单面往Z轴发射光线的矩形，目前只能和烘焙GI一起使用，区域光会均匀的照亮作用区域，虽然区域光没有范围属性可以调整，但是光的强度也是会随着距离光源越远而递减；适用于建立柔和的照明效果；适合当作天花板壁灯或是背光灯；
*   发光材质(EMISSIVE MATERIALS) 可以让物体表面发光，他们可以反射场景内像是颜色或是光强度等等能在游戏内改变的光源，自发光(Emission)是一个在标准着色器(Standard Shader)内的属性，允许静态对象成为一个发光体，预设情况下是0，代表指定了这个材质并不会有任何的自发光反应；没有范围属性，但从材质发出的光会以二的次方速度递减，自发光材质只会作用在有标记为”Static”或”LightmapStatic”卷标的对象；适合来模拟霓虹灯等类似的光源；
*   光探头(LIGHT PROBES) 静态对象只被Unity全局光照GI系统计算，光探头可用于动态对象能够和静态场景接收到的光影信息互动；光探头允许移动对象接受由全局光照GI所计算出来复杂的反射光源，对象在著色网格的时候会判断附近光探头的位置并且把光的信息一并融合计算，这是透过找寻由光探头所产生的一个四面体，然后决定哪个四面体的落入对象的轴向，这样就能让场景内的动态对象正确地接受光信息，如果没有放置光探头，动态对象就无法接受全局光照的信息，造成动态对象比场景还要暗。

###### **光照模型**

着色：根据材质属性、光源信息（方向，辐照度），使用一个等式去计算沿某个观察方向的出射度的过程。这个等式称为光照模型。

*   自发光c_emissive_：当给定一个方向时，一个表面本身会向该方向发射多少辐照量。通常在实时渲染中，不会照亮别的物体，计算时使用材质的自发光颜色： **c**_emissive_ = **m**_emissive                                               _
*   高光反射**c**_specular_:当光线从光源照射到模型表面时，该表面会在完全镜面反射方向散射多少辐射量。反射方向公式： **r** = 2(**n**.**l**)**n**-**l** Phong模型高光反射计算公式： **c**_specular_ = (**c**_light_.**m**_specular_)pow(max(0,**v**.**r**),**m**_gloss_) 其中**m**gloss是材质的光泽度、反光度，用于控制亮点大小。**m**_specular_材质的高光反射颜色，clight光源颜色。**v**是视角方向。Blinn模型： **h** = (**v** +**l**)/|**v**+**l**| **c**_specular_ = (**c**_light_.**m**_specular_)pow(max(0,**n**.**h**),**m**_gloss_) 逐顶点光照实现：
    
    v2f vert(a2v v) {
    	v2f o;
    	// Transform the vertex from object space to projection space
    	o.pos = mul(UNITY\_MATRIX\_MVP, v.vertex);
    				
    	// Get ambient term
    	fixed3 ambient = UNITY\_LIGHTMODEL\_AMBIENT.xyz;
    				
    	// Transform the normal from object space to world space
    	fixed3 worldNormal = normalize(mul(v.normal, (float3x3)unity_WorldToObject));
    	// Get the light direction in world space
    	fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);
    				
    	// Compute diffuse term
    	fixed3 diffuse = \_LightColor0.rgb * \_Diffuse.rgb * saturate(dot(worldNormal, worldLightDir));
    				
    	// Get the reflect direction in world space
    	fixed3 reflectDir = normalize(reflect(-worldLightDir, worldNormal));
    	// Get the view direction in world space
    	fixed3 viewDir = normalize(\_WorldSpaceCameraPos.xyz - mul(unity\_ObjectToWorld, v.vertex).xyz);
    				
    	// Compute specular term
    	fixed3 specular = \_LightColor0.rgb * \_Specular.rgb * pow(saturate(dot(reflectDir, viewDir)), _Gloss);
    				
    	o.color = ambient + diffuse + specular;
    							 	
    	return o;
    }
    
    逐像素光照实现：
    
    v2f vert(a2v v) {
    	v2f o;
    	// Transform the vertex from object space to projection space
    	o.pos = mul(UNITY\_MATRIX\_MVP, v.vertex);
    				
    	// Transform the normal from object space to world space
    	o.worldNormal = mul(v.normal, (float3x3)unity_WorldToObject);
    	// Transform the vertex from object spacet to world space
    	o.worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;
    				
    	return o;
    }
    			
    fixed4 frag(v2f i) : SV_Target {
    	// Get ambient term
    	fixed3 ambient = UNITY\_LIGHTMODEL\_AMBIENT.xyz;
    				
    	fixed3 worldNormal = normalize(i.worldNormal);
    	fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);
    				
    	// Compute diffuse term
    	fixed3 diffuse = \_LightColor0.rgb * \_Diffuse.rgb * saturate(dot(worldNormal, worldLightDir));
    				
    	// Get the reflect direction in world space
    	fixed3 reflectDir = normalize(reflect(-worldLightDir, worldNormal));
    	// Get the view direction in world space
    	fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz - i.worldPos.xyz);
    	// Compute specular term
    	fixed3 specular = \_LightColor0.rgb * \_Specular.rgb * pow(saturate(dot(reflectDir, viewDir)), _Gloss);
    				
    	return fixed4(ambient + diffuse + specular, 1.0);
    }
    

BlinnPhong：

v2f vert(a2v v) {
	v2f o;
	// Transform the vertex from object space to projection space
	o.pos = mul(UNITY\_MATRIX\_MVP, v.vertex);
				
	// Transform the normal from object space to world space
	o.worldNormal = mul(v.normal, (float3x3)unity_WorldToObject);
				
	// Transform the vertex from object spacet to world space
	o.worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;
				
	return o;
}
			
fixed4 frag(v2f i) : SV_Target {
	// Get ambient term
	fixed3 ambient = UNITY\_LIGHTMODEL\_AMBIENT.xyz;
				
	fixed3 worldNormal = normalize(i.worldNormal);
	fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);
				
	// Compute diffuse term
	fixed3 diffuse = \_LightColor0.rgb * \_Diffuse.rgb * max(0, dot(worldNormal, worldLightDir));
				
	// Get the view direction in world space
	fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz - i.worldPos.xyz);
	// Get the half direction in world space
	fixed3 halfDir = normalize(worldLightDir + viewDir);
	// Compute specular term
	fixed3 specular = \_LightColor0.rgb * \_Specular.rgb * pow(max(0, dot(worldNormal, halfDir)), _Gloss);
				
	return fixed4(ambient + diffuse + specular, 1.0);
}

BlinnPhong Use Build In Functions：

v2f vert(a2v v) {
	v2f o;
	o.pos = mul(UNITY\_MATRIX\_MVP, v.vertex);
				
	// Use the build-in funtion to compute the normal in world space
	o.worldNormal = UnityObjectToWorldNormal(v.normal);
				
	o.worldPos = mul(unity_ObjectToWorld, v.vertex);
				
	return o;
}
			
fixed4 frag(v2f i) : SV_Target {
	fixed3 ambient = UNITY\_LIGHTMODEL\_AMBIENT.xyz;
				
	fixed3 worldNormal = normalize(i.worldNormal);
	//  Use the build-in funtion to compute the light direction in world space
	// Remember to normalize the result
	fixed3 worldLightDir = normalize(UnityWorldSpaceLightDir(i.worldPos));
				
	fixed3 diffuse = \_LightColor0.rgb * \_Diffuse.rgb * max(0, dot(worldNormal, worldLightDir));
				
	// Use the build-in funtion to compute the view direction in world space
	// Remember to normalize the result
	fixed3 viewDir = normalize(UnityWorldSpaceViewDir(i.worldPos));
	fixed3 halfDir = normalize(worldLightDir + viewDir);
	fixed3 specular = \_LightColor0.rgb * \_Specular.rgb * pow(max(0, dot(worldNormal, halfDir)), _Gloss);
				
	return fixed4(ambient + diffuse + specular, 1.0);
}

 

*   漫反射**c**_diffuse_:当光线从光源照射到模型表面时，该表面会向每个方向散射多少辐射量。符合**兰伯特定律**：反射光线的强度与表面法线和光源方向之间夹角的余弦值成正比，计算公式： **c**_diffuse_ = (**c**_light_ . **m**_diffuse_)max(0,**n**.**l**) **n**是表面法线，**l**是指向光源的单位矢量，**m**_diffuse_是材质的漫反射颜色，**c**_light_是光源颜色 逐顶点光照实现：
    
    v2f vert(a2v v) {
    	v2f o;
    	// Transform the vertex from object space to projection space
    	o.pos = mul(UNITY\_MATRIX\_MVP, v.vertex);
    				
    	// Get ambient term
    	fixed3 ambient = UNITY\_LIGHTMODEL\_AMBIENT.xyz;
    				
    	// Transform the normal from object space to world space
    	fixed3 worldNormal = normalize(mul(v.normal, (float3x3)unity_WorldToObject));
    	// Get the light direction in world space
    	fixed3 worldLight = normalize(_WorldSpaceLightPos0.xyz);
    	// Compute diffuse term
    	fixed3 diffuse = \_LightColor0.rgb * \_Diffuse.rgb * saturate(dot(worldNormal, worldLight));
    				
    	o.color = ambient + diffuse;
    				
    	return o;
    }
    
    逐像素光照实现：
    
    v2f vert(a2v v) {
    	v2f o;
    	// Transform the vertex from object space to projection space
    	o.pos = mul(UNITY\_MATRIX\_MVP, v.vertex);
    
    	// Transform the normal from object space to world space
    	o.worldNormal = mul(v.normal, (float3x3)unity_WorldToObject);
    
    	return o;
    }
    			
    fixed4 frag(v2f i) : SV_Target {
    	// Get ambient term
    	fixed3 ambient = UNITY\_LIGHTMODEL\_AMBIENT.xyz;
    				
    	// Get the normal in world space
    	fixed3 worldNormal = normalize(i.worldNormal);
    	// Get the light direction in world space
    	fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);
    				
    	// Compute diffuse term
    	fixed3 diffuse = \_LightColor0.rgb * \_Diffuse.rgb * saturate(dot(worldNormal, worldLightDir));
    				
    	fixed3 color = ambient + diffuse;
    				
    	return fixed4(color, 1.0);
    }
    
    **半兰伯特定律： c**_diffuse_ **= (c**_light_**.m**_diffuse_**)(α(n.l) + β)** **α,β**通常为0.5 **c**_diffuse_ **= (c**_light_**.m**_diffuse_**)(0.5(n.l) + 0.5)** 把** n.l**结果从【-1，1】映射到【0，1】，该定律无物理依据，仅仅是一个视觉加强技术
    
    v2f vert(a2v v) {
    	v2f o;
    	// Transform the vertex from object space to projection space
    	o.pos = mul(UNITY\_MATRIX\_MVP, v.vertex);
    				
    	// Transform the normal from object space to world space
    	o.worldNormal = mul(v.normal, (float3x3)unity_WorldToObject);
    				
    	return o;
    }
    			
    fixed4 frag(v2f i) : SV_Target {
    	// Get ambient term
    	fixed3 ambient = UNITY\_LIGHTMODEL\_AMBIENT.xyz;
    				
    	// Get the normal in world space
    	fixed3 worldNormal = normalize(i.worldNormal);
    	// Get the light direction in world space
    	fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);
    				
    	// Compute diffuse term
    	fixed halfLambert = dot(worldNormal, worldLightDir) * 0.5 + 0.5;
    	fixed3 diffuse = \_LightColor0.rgb * \_Diffuse.rgb * halfLambert;
    				
    	fixed3 color = ambient + diffuse;
    				
    	return fixed4(color, 1.0);
    }
    
     
*   环境光**c**_ambient_:用于描述其他所有的间接光照。通常是一个全局变量： **c**_ambient_ = **g**_ambient_ 在unity中通过Windows->Lights->Ambient Source/Ambient Color/Ambient Intensity控制；在Shader中通过UNITY\_LIGHTMODEL\_AMBIENT获取颜色及强度

  待续 参考： [Introduction to Lighting and Rendering](https://unity3d.com/cn/learn/tutorials/topics/graphics/introduction-lighting-and-rendering?playlist=17102) （中文：[Unity 5 中的全局光照技术详解](http://www.cocoachina.com/game/20150701/12339.html)）《UnityShader 入门精要》