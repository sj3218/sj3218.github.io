---
layout: page
name: Liquid Monsters
tools: [Game, Puzzle, 2D, Custom Engine]
image: "/assets/images/background/liquid.JPG"
---

# Liquid Monsters
<br>
{% include elements/video.html id="NQsKBhQuyy4" %}

<br>

<div style="display: flex; gap: 20px;">
  <div style="background-color: #2c2c2c; padding: 20px; border-radius: 8px; color: white; width: 50%;">
    <h2>Description</h2><br>
    <p>
      Liquid Monster는 2D 플랫폼 퍼즐 게임입니다. 두명의 플레이어가 서로를 먹어서 합쳐지거나 아이템을 사용해서 맵을 풀어나가는 게임입니다.
    </p>
    <p>
      저는 게임에 필요한 아이템들을 구현하고 ImGui를 사용해 직관적인 사용자 인터페이스를 제공하여 디자이너들이 맵 내의 요소들을 배치하고 수정할 수 있도록 했습니다. 이 외에도 FMOD를 이용해 게임에 적용될 소리들을 추가하며 게임의 몰입감을 높이는 데 기여했습니다.
    </p>
  </div>
  <div style="background-color: #2c2c2c; padding: 20px; border-radius: 8px; color: white; width: 50%;">
    <h2>Project Info</h2><br>
    <p>👨‍💻 직책: 플레이어 프로그래머, 툴 프로그래머</p>
    <p>👥 팀 규모: 5</p>
    <p>⏳ 개발 기간: 2019.09 ~ 2020.04</p>
    <p>🛠️ Engine: 자체 엔진(C++, OpenGL, SDL, FMOD, ImGui, Json)</p>
    <p>⚙️ Files 다운로드: <button onclick="window.location.href='https://drive.google.com/drive/folders/1idm_-ZEHRmeOC7ruRiTdck5SZRJdtbHb';">소스파일 다운로드</button></p>
  </div>
</div>

<br>

<br>

### **툴 에디터 구현**

{% include elements/video.html id="_AvoJwB_uuQ" %}

###### 위의 영상은 Debugging 모드 영상입니다.
<br>

###### 디자이너들이 요청하는 사항에 따라 에디터 툴을 설계하고 구현했습니다. 
###### **Level Editor** 부분은 오브젝트들을 생성하거나 삭제를 할 수 있도록 하였습니다. 생성된 오브젝트들은 마우스를 통해 맵에 원하는 위치에 이동 시킬 수 있고 텍스처와 애니메이션을 적용할 수 있도록 하였습니다. 
###### **Tile Editor**에서는 그래픽스 타일과 물리 타일을 마우스로 드래그 하여 맵을 디자인 할 수 있도록 하였고 흰색으로 테두리로 표시된 타일은 물리 타일로 플레이어가 올라 갈 수 있는 타일입니다. 
###### **Player Information**에서는 플레이어의 위치나 속도를 볼 수 있도록 인터페이스를 제공했습니다.


<br>

### **데이터 관리 시스템 구현**

###### 개발 중 레벨이 이동할 때마다 텍스처 로딩 시간이 길어져 게임 플레이의 몰입감을 저하시키는 문제가 발생했습니다. 이를 해결하기 위해 데이터 관리 시스템을 설계 및 구현했습니다. 텍스처의 이름과 파일 경로를 **.dat** 파일에 저장하고 게임 시작 시 모든 텍스처를 한 번에 초기화하여 미리 불러오는 방식으로 최적화하였습니다. 
###### 먼저 아래의 형식처럼 텍스처의 이름과 파일 경로를 All_Sprite.dat 파일에 저장했습니다.
```
OBJECT_WIND	texture/Object/wind.png
OBJECT_PRESSURE	texture/Object/temp_pressure.png
BACKGROUND	texture/Level/background_chapter_1.png
TEST_BACKGROUND	texture/Level/test_background.png
TILE_EXAMPLE_1	texture/Tile/Tile_example_3.png
TILE_EXAMPLE_2	texture/Tile/Tile_example_2.png
```
###### 텍스쳐를 초기화하는 함수에서는 input파일을 열어 데이터를 읽어 받도록 하였습니다.
```c++
void Data::Initialize_Texture()
{
    std::ifstream sprite_dat(mTextureFileName.c_str());
    std::string   line;
    if (sprite_dat.is_open())
    {
        while (std::getline(sprite_dat, line))
        {
            size_t index = line.find(delimiter);
            eTexture sprite_name = static_cast<eTexture>(mTextureEnum);

            std::string sprite_path = line.substr(index + 1, -1);
            mLoadQueue.push({sprite_name, sprite_path});
            ++mTextureEnum;
        }
    }
    sprite_dat.close();

    workerThread.push_back(std::thread{&Data::LoaderThread, this});
}
```
###### 이후 교수님께서 멀티쓰레딩을 사용하면 이러한 문제를 더욱 효과적으로 해결할 수 있다고 조언해 주셨고, 이를 바탕으로 학부 마지막 학기에 Parallel Programming 수업을 수강하며 관련 지식을 보충했습니다.
<br>

### **아이템 구현**

###### 게임 내에서 플레이어가 활용할 수 있는 다양한 아이템을 구현했습니다. 예를 들어, 배터리를 사용하여 무빙 박스를 이동시키거나, 버튼을 눌러 플레이어를 띄우는 팬을 작동시키는 등의 아이템들을 구현하였습니다. 
###### 아이템 구현에는 Component 클래스를 기반으로 하여 각 아이템의 기능을 정의했으며 아이템이 작동할 때마다 그래픽스 타일들을 업데이트하여 사용자들이 변화된 상태를 확인 할 수 있도록 하였습니다.
```C++
void Battery::UpdateWireGraphics(bool isBattery)
{
    if (isBattery) // true = when the battery is in. BLUE
    {
        for (auto wire : mWireTiles)
        {
            switch (wire.second)
            {
            case eTexture::TILE_WIRE_RED_1:
                TILE_MAP->ChangeGraphicTileTexture(wire.first, eTexture::TILE_WIRE_BLUE_1);
                break;
            case eTexture::TILE_WIRE_RED_2:
                TILE_MAP->ChangeGraphicTileTexture(wire.first, eTexture::TILE_WIRE_BLUE_2);
                break;
            }
        }
    }
    else // false = when the battery is not in. RED
    {
        for (auto wire : mWireTiles)
        {
            switch (wire.second)
            {
            case eTexture::TILE_WIRE_BLUE_1:
                TILE_MAP->ChangeGraphicTileTexture(wire.first, eTexture::TILE_WIRE_RED_1);
                break;
            case eTexture::TILE_WIRE_BLUE_2:
                TILE_MAP->ChangeGraphicTileTexture(wire.first, eTexture::TILE_WIRE_RED_2);
                break;
            }
        }
    }
}
```

<br>

## **사진**
{% capture carousel_images %}
/assets/images/liquid_monster/liquidmonster_debug.JPG
/assets/images/liquid_monster/liquidmonster_debug_2.JPG
/assets/images/liquid_monster/liquidmonster_game_mode.JPG
{% endcapture %}
{% include elements/carousel.html %}


<br>

## **배운 점**
###### 이 게임 프로젝트는 제가 처음 팀원들과 제작한 게임 프로젝트였습니다. 1년 동안 팀원들과 협력하여 게임 엔진을 개발하며 다양한 어려움에 직면했습니다. 예를 들어, 텍스처 로딩 시간이 길어져 레벨 로딩이 느려지는 문제나, 물리 엔진에서 캐릭터가 박스 모서리에 끼는 문제가 발생했습니다. 이러한 문제들을 해결하기 위해 팀원들과 끊임없이 소통하고 해결책을 모색하며, 문제 해결 능력뿐만 아니라 협력의 중요성을 배웠습니다.

<br><br><br><br><br>
<br>