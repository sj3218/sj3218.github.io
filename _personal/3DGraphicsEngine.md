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


<details>
<summary>Phong Shading.frag</summary>
<div markdown="1" style="padding-left:20px;">

```glsl
#version 450 core

#define PI 4.0f * atan(1.0f)

in vec3 fragPosition;
in vec3 outNormal;
in vec2 texCoord;

out vec4 fragColor;

struct lightInformation
{
    int lightType; // dir, point, spot
    float innerAngle;
    float outerAngle;
    float fallOff;
    vec3 ambientColor;
    vec3 diffuseColor;
    vec3 specularColor;
    vec3 lightPosition;
    vec3 direction;
};

layout (std140, binding =0) uniform global
{
    int numbLights;
    float far;
    float near;
    vec3 attenuation;
    vec3 fogColor;
    vec3 globalAmbientColor;
};

layout (std140, binding =1) uniform light
{
    lightInformation lights[16];
};

uniform float ambientStrength;
uniform bool isTexture;
uniform sampler2D diffuseTexture;
uniform sampler2D specularTexture;
uniform vec3 ambientMaterial;
uniform vec3 emissive;
uniform vec3 cameraPosition;
uniform vec3 color;

uniform bool calculateUV_GPU;
uniform int projectionMode;
uniform bool entities;

uniform mat4 transform;
uniform mat4 cameraMatrix;
uniform mat4 ndcMatrix;

vec3 CalculateDirectionalLight(int index, vec3 normal, vec3 view, vec2 tex)
{
    int Ns = 64;
    vec3 lightDir = normalize(-lights[index].direction);
    vec3 reflectDir = 2*dot(normal, lightDir)*normal - lightDir;

    float diffuseValue = max(dot(normal, lightDir),0.f);
    float specularValue = pow(max(dot(view, reflectDir), 0.f), Ns);

    vec3 ambient = lights[index].ambientColor * ambientMaterial * ambientStrength;
    vec3 diffuse = lights[index].diffuseColor * diffuseValue;
    vec3 specular = lights[index].specularColor * specularValue;

    if(isTexture)
    {
        diffuse *= vec3(texture(diffuseTexture, tex).rgb);
        specular *= vec3(texture(specularTexture, tex).rgb);
    }

    return ambient + diffuse + specular;
}

vec3 CalculatePointLight(int index, vec3 normal, vec3 view, vec3 pos, vec2 tex)
{
    float Ns = 64;

    vec3 lightDir = lights[index].lightPosition - pos;
    float lightDirLength = length(lightDir);
    
    float attenuationValue = min(1/(attenuation.x + attenuation.y * lightDirLength + attenuation.z * lightDirLength*lightDirLength), 1.f);

    lightDir = normalize(lightDir);

    vec3 reflectDir = 2*dot(normal, lightDir)*normal - lightDir;

    float diffuseValue = max(dot(normal, lightDir), 0.f);
    float specularValue = pow(max(dot(view, reflectDir), 0.f), Ns);

    vec3 ambient = lights[index].ambientColor * ambientMaterial * ambientStrength;
    vec3 diffuse = lights[index].diffuseColor * diffuseValue;
    vec3 specular = lights[index].specularColor * specularValue;

    if(isTexture)
    {
        diffuse *= vec3(texture(diffuseTexture, tex).rgb);
        specular *= vec3(texture(specularTexture, tex).rgb);
    }

    return (ambient + diffuse + specular) * attenuationValue;
}

vec3 CalculateSpotLight(int index, vec3 normal, vec3 view, vec3 pos, vec2 tex)
{
    int Ns = 64;
    vec3 lightDir = lights[index].lightPosition- pos;
    float lightDirLength = length(lightDir);

    float attenuationValue = min(1/(attenuation.x + attenuation.y * lightDirLength + attenuation.z * lightDirLength*lightDirLength), 1.f);

    lightDir = normalize(lightDir);

    vec3 reflectDir = 2*dot(normal, lightDir)*normal - lightDir;

    float diffuseValue = max(dot(normal, lightDir), 0.f);
    float specularValue = pow(max(dot(view, reflectDir), 0.f), Ns);

    float cosAlpha = dot(lightDir, normalize(-lights[index].direction));

    float spotLightEffect =0;
    if(cosAlpha < cos(lights[index].outerAngle))
    {
        spotLightEffect = 0.f;
    }
    else if (cosAlpha > cos(lights[index].innerAngle))
    {
        spotLightEffect = 1.f;
    }
    else
    {
        spotLightEffect = pow((cosAlpha - cos(lights[index].outerAngle))/(cos(lights[index].innerAngle)-cos(lights[index].outerAngle)), lights[index].fallOff);
    }

    vec3 ambient = lights[index].ambientColor * ambientMaterial * ambientStrength;
    vec3 diffuse = lights[index].diffuseColor * diffuseValue;
    vec3 specular = lights[index].specularColor * specularValue;

    if(isTexture)
    {
        diffuse *= vec3(texture(diffuseTexture, tex).rgb);
        specular *= vec3(texture(specularTexture, tex).rgb);
    }


    return attenuationValue*(ambient + spotLightEffect*(diffuse + specular));
}

void main()
{
    vec2 texture_coordinate;
    if(calculateUV_GPU)
    {
        float u = 0.f;
        float v = 0.f;
        float theta = 0.f;
        float phi = 0.f;
        
        vec3 calculatePosition = fragPosition;

        if(entities)
        {
            calculatePosition = normalize(outNormal);
        }
        
        if(projectionMode == 1) // cylindrical uv
        {
            theta = atan(calculatePosition.z, calculatePosition.x);
            theta += PI;
            
            u = theta/(2.f*PI);
            v = (calculatePosition.y + 1.0f) *0.5f;
        }
        else if(projectionMode == 2) // spherical uv
        {
            theta = atan(calculatePosition.z , calculatePosition.x);
            theta += PI;

            phi = acos(calculatePosition.y/ length(calculatePosition));
            
            u = theta/(2.f*PI);
            v = 1.f - (phi / PI);
        }
        else if(projectionMode == 3) // cube uv
        {
            
            float x = calculatePosition.x;
            float y = calculatePosition.y;
            float z = calculatePosition.z;

            float absX = abs(x);
            float absY = abs(y);
            float absZ = abs(z);

            int isXPositive = x > 0 ? 1 : 0;
            int isYPositive = y > 0 ? 1 :0;
            int isZPositive = z > 0 ? 1 :0;

            float maxAxis = 0.f;
            float uc = 0.f;
            float vc = 0.f;
    
            // POSITIVE X
            if (bool(isXPositive) && (absX >= absY) && (absX >= absZ))
            {
                maxAxis = absX;
                uc = -z;
                vc = y;
            }
            // NEGATIVE X
            else if (!bool(isXPositive) && absX >= absY && absX >= absZ)
            {
                maxAxis = absX;
                uc = z;
                vc = y;
            }

            // POSITIVE Y
            else if (bool(isYPositive) && absY >= absX && absY >= absZ)
            {
                maxAxis = absY;
                uc = x;
                vc = -z;
            }

            // NEGATIVE Y
            else if (!bool(isYPositive) && absY >= absX && absY >= absZ)
            {
                maxAxis = absY;
                uc = x;
                vc = z;
            }

            // POSITIVE Z
            else if (bool(isZPositive) && absZ >= absX && absZ >= absY)
            {
                maxAxis = absZ;
                uc = x;
                vc = y;
            }

            // NEGATIVE Z
            else if (!bool(isZPositive) && absZ >= absX && absZ >= absY)
            {
                maxAxis = absZ;
                uc = -x;
                vc = y;
            }

            // Convert range from -1 to 1 to 0 to 1
            u = 0.5f * (uc / maxAxis + 1.0f);
            v = 0.5f * (vc / maxAxis + 1.0f);
        }
        texture_coordinate.x = u;
        texture_coordinate.y = v;
    }
    else
    {
        texture_coordinate = texCoord;
    }
    
    vec3 result = emissive + globalAmbientColor* ambientStrength;
    vec3 normalVector = normalize(outNormal);

    vec3 viewVector = cameraPosition - fragPosition;
    float viewVectorLength = length(viewVector);

    viewVector = normalize(viewVector);

    for(int i =0; i< numbLights; ++i)
    {
        if(lights[i].lightType == 0)
        {
            result += CalculatePointLight(i, normalVector, viewVector, fragPosition,texture_coordinate);
        }
        else if(lights[i].lightType == 1)
        {
            result += CalculateDirectionalLight(i, normalVector, viewVector, texture_coordinate);
        }
        else if(lights[i].lightType == 2)
        {
            result += CalculateSpotLight(i, normalVector, viewVector, fragPosition, texture_coordinate);
        }
    }

    //fog
    float tempNear = near;
    if(near > far)
    {
        tempNear = far;
    }

    float fogValue = (far - viewVectorLength) / (far - tempNear);
    fogValue = min(fogValue, 1.f);
    fogValue = max(fogValue, 0.f);
    

    vec3 IFinal = fogValue* result + (1-fogValue) * fogColor;
    IFinal = vec3(min(IFinal.x, 1.f), min(IFinal.y, 1.f), min(IFinal.z, 1));
    
    fragColor = vec4(IFinal, 1.f);
    
}
```
</div>
</details> 

- ###### **Blinn Shading**에서는 반사 벡터를 직접 계산하는 대신, 빛 벡터와 시점 벡터를 더해 이를 normalize시킨 벡터를 사용하여 반사 효과를 처리했습니다.

```glsl
vec3 reflectDir = (lightDir + view);
reflectDir = normalize(reflectDir);
```

###### 결과적으로 **Blinn Shading**은 다른 쉐이딩과 비슷한 결과를 제공하고 다른 반사 벡터 계산하는데 비용이 비싼 다른 쉐이딩 알고리즘 보다 더 효율적인 것을 알 수 있었습니다.


<br>

### 환경 맵핑

###### 이 프로젝트에서는 Skybox와 환경맵핑(반사, 굴절)을 보여줍니다. OpenGL에서 제공하는 큐브 맵핑 함수를 쓰지 않고 직접 구현을 해보며 어떤 원리로 작동을 하는지 공부하였습니다. FBO에 6개의 다른 텍스처들을 **skyTexture**에 저장하고 CubeMapping 함수에서 저장된 텍스처들을 활용하여 물체의 표면에 반사 및 굴절 효과를 적용하고, 이를 통해 현실감 있는 장면을 연출 할 수 있었습니다.  
###### 또한 Hybrid Rendering 기법을 사용해 모든 빛들을 forward 렌더링 기법을 사용하여 버퍼에 저장하고 환경 맵핑을 받는 메인 오브젝트에 deferred 렌더링 기법을 사용해서 렌더링 처리 하였습니다.


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


### Dynamics 빛 구현

###### Point, Directional, Spot 빛을 구현하였습니다.

{% capture carousel_images %}
/assets/images/graphics_engine/point_light.JPG
/assets/images/graphics_engine/directional.JPG
/assets/images/graphics_engine/spot_light.JPG
{% endcapture %}