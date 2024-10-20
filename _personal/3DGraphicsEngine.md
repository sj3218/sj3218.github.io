---
layout: page
name: 3D 그래픽스 엔진
tools: [C++, OpenGL, Custom Engine, 3D]
image: "/assets/images/background/graphics.JPG"
---

# 3D Graphics Engine

<br>
{% include elements/video.html id="gCqtzcakhlA" %}

<br>

<div style="display: flex; gap: 20px;">
  <div style="background-color: #2c2c2c; padding: 20px; border-radius: 8px; color: white; width: 50%;">
    <h2>Description</h2><br>
    <p>
      이 프로젝트는 Modern GPU 구조를 이해하고 OpenGL을 이용한 그래픽스 엔진을 개발하기 위한 목적으로 진행되었습니다. 메인 오브젝트를 중심으로 빛 구체들이 회전하는 씬을 구현했습니다. 
    </p>
  </div>
  <div style="background-color: #2c2c2c; padding: 20px; border-radius: 8px; color: white; width: 50%;">
    <h2>Project Info</h2><br>
    <p>👥 팀 규모: 솔로 프로젝트</p>
    <p>⏳ 개발 기간: 2021.08 ~ 2022.04</p>
    <p>🛠️ Engine: 자체 엔진(C++, OpenGL, ImGui, GLSL)</p>
    <p>⚙️ Git hub: <button onclick="window.location.href='https://github.com/sj3218/Graphics-Project-3';">링크 이동</button></p>
  </div>
</div>

<br>


### Shading 알고리즘

###### 이 프로젝트에서는 Phong Lighting, Phong Shading, Blinn Shading 알고리즘을 구현하였습니다. 
- ###### **Phong Lighting**은 vertex shader에서 계산된 반사 벡터를 사용하여 diffuse와 specular 색을 계산합니다.
- ###### **Phong Shading**은 Phong Lighting과 유사하지만, 빛 계산을 fragment shader에서 수행하는 방식입니다. 
- ###### **Blinn Shading**에서는 반사 벡터를 직접 계산하는 대신, 빛 벡터와 시점 벡터를 더해 이를 normalize시킨 벡터를 사용하여 반사 효과를 처리했습니다.

###### 기존에 반사벡터를 구하기 위해서 아래의 식을 사용하여

![alt text](
/assets/images/graphics_engine/equation.JPG)

###### 이처럼 구현을 하였다면
```glsl
vec3 reflectDir = 2*dot(normal, lightDir) *normal - lightDir;
```
###### Blinn Shading에서는 반사벡터를 이렇게 구하였습니다.
```glsl
vec3 reflectDir = (lightDir + view);
reflectDir = normalize(reflectDir);
```

###### 결과적으로 **Blinn Shading**은 다른 쉐이딩과 비슷한 결과를 제공하고 다른 반사 벡터 계산하는데 비용이 비싼 다른 쉐이딩 알고리즘 보다 더 효율적인 것을 알 수 있었습니다.


<br>

### 환경 맵핑

###### 이 프로젝트에서는 Skybox와 환경맵핑(반사, 굴절)을 보여줍니다. OpenGL에서 제공하는 큐브 맵핑 함수를 쓰지 않고 직접 구현을 해보며 어떤 원리로 작동을 하는지 공부하였습니다. FBO에 6개의 다른 텍스처들을 **skyTexture**에 저장하고 CubeMapping 함수에서 저장된 텍스처들을 활용하여 물체의 표면에 반사 및 굴절 효과를 적용하고, 이를 통해 현실감 있는 장면을 연출 할 수 있었습니다.  

```glsl

uniform sampler2D skyTexture[6];
uniform float blending;
uniform int environmentType; //0 = reflection, 1 = refraction, 2 = combination

vec3 CubeMapping(vec3 pos)
{
    float x = pos.x;
    float y = pos.y;
    float z = pos.z;

    float absX = abs(x);
    float absY = abs(y);
    float absZ = abs(z);

    int isXPositive = x > 0 ? 1 : 0;
    int isYPositive = y > 0 ? 1 : 0;
    int isZPositive = z > 0 ? 1 : 0;

    float maxAxis =  0.f;
    vec2 uv = vec2(0.f);

    vec3 resultTexture = vec3(0.f);

    if((absX >= absY) && (absX >= absZ))
    {
        if(isXPositive == 1) // positive x // right
        {
            maxAxis = absX;

            uv.x = 0.5f * (z / maxAxis + 1.f);
            uv.y = 0.5f * (y / maxAxis + 1.f);

            resultTexture = texture(skyTexture[1], uv).rgb;
        }
        else if(isXPositive == 0) // negative x // left
        {
            maxAxis = absX;

            uv.x = 0.5f * (-z / maxAxis + 1.f);
            uv.y = 0.5f * (y / maxAxis + 1.f);

            resultTexture = texture(skyTexture[0], uv).rgb;
        }
    }
    else if((absY >= absX) && (absY >= absZ))
    {
        if(isYPositive == 1) //positive y // top
        {
            maxAxis = absY;
            
            uv.x = 0.5f * (-z / maxAxis + 1.f);
            uv.y = 0.5f * (x / maxAxis + 1.f);

            resultTexture = texture(skyTexture[3], uv).rgb;
        }
        else if(isYPositive == 0) // negative y // bottom
        {
            maxAxis = absY;
            
            uv.x = 0.5f * (z / maxAxis + 1.f);
            uv.y = 0.5f * (x / maxAxis + 1.f);

            resultTexture = texture(skyTexture[2], uv).rgb;
        }
    }
    else if((absZ >= absX) && (absZ >= absY))
    {
        if(isZPositive == 1) // positive z // front
        {
            maxAxis = absZ;
            
            uv.x = 0.5f * (-x / maxAxis + 1.f);
            uv.y = 0.5f * (y / maxAxis + 1.f);

            resultTexture = texture(skyTexture[5], uv).rgb;
        }
        else if(isZPositive == 0) //negative z // back
        {
            maxAxis = absZ;
            
            uv.x = 0.5f * (x / maxAxis + 1.f);
            uv.y = 0.5f * (y / maxAxis + 1.f);

            resultTexture = texture(skyTexture[4], uv).rgb;
        }
    }
    return resultTexture;
}
```
<br>
{% capture carousel_images %}
/assets/images/graphics_engine/graphics_environment/reflection.JPG
/assets/images/graphics_engine/graphics_environment/refraction.JPG
/assets/images/graphics_engine/graphics_environment/fresnel.JPG
/assets/images/graphics_engine/graphics_environment/chromatic_aberration.JPG
/assets/images/graphics_engine/graphics_environment/blending_reflection.JPG
{% endcapture %}

###### 또한 Hybrid Rendering 기법을 사용해 모든 빛들을 forward 렌더링 기법을 사용하여 버퍼에 저장하고 환경 맵핑을 받는 메인 오브젝트에 deferred 렌더링 기법을 사용해서 렌더링 처리 하였습니다.

### Dynamics 빛 구현

###### Point, Directional, Spot 빛을 구현하였습니다.

{% capture carousel_images %}
/assets/images/graphics_engine/point_light.JPG
/assets/images/graphics_engine/directional.JPG
/assets/images/graphics_engine/spot_light.JPG
{% endcapture %}

<br>
<br>
<br>
<br>
<br>
<br>