---
layout: page
name: Freeze Tag
tools: [Game, VR, 3D, Unity]
image: "/assets/images/background/freeze.JPG"
---

# Freeze Tag
<br>
{% include elements/video.html id="WP4t_LqYApI" %}

<br>

<div style="display: flex; gap: 20px;">
  <div style="background-color: #2c2c2c; padding: 20px; border-radius: 8px; color: white; width: 50%;">
    <h2>Description</h2><br>
    <p>
       Freeze Tag게임은VR액션싱글게임입니다.맵에 깔려있는 보석들을 모두 모은 후 적들을 피해 NPC들을 풀어주면 이기는 게임입니다. 적 AI는플레이어가 처다보지 않을 때 빨간색을 띄고 쫓아옵니다. 플레이어의 일정 반경 안에 들어오고 플레이어가 처다 볼시 적 AI들은 파란색을 띄고 멈춥니다.
    </p>
    <p>
      저는 Unity에서 AI를 구현하고 플레이어가 볼 수 있도록 폰에 UI 작업을 했습니다.
    </p>
  </div>
  <div style="background-color: #2c2c2c; padding: 20px; border-radius: 8px; color: white; width: 50%;">
    <h2>Project Info</h2><br>
    <p>👨‍💻 직책: 게임플레이 프로그래머</p>
    <p>👥 팀 규모: 2</p>
    <p>⏳ 개발 기간: 2022.09 ~ 2022.12</p>
    <p>🛠️ Engine: Unity Engine</p>
    <p>⚙️ Github: <button onclick="window.location.href='https://github.com/wonju-cho/FreezeTag';">Github 링크</button></p>
    <p>⚙️ Source code: <button onclick="window.location.href='https://drive.google.com/drive/folders/14vhGjgF-oV2Kzc6WvLDmn9Sh06RBYybf';">Source Code 링크</button></p>
  </div>
</div>

<br>

### **AI**

###### Unity에서 navmesh와 A* 알고리즘을 이용해 게임 ai를 만들 수 있도록 제공하는 것은 알고있습니다. 그래도 Unity를 사용해서 A* 알고리즘을 직접 개발하여 게임에 적용시키고 싶어 시작했습니다. 
###### FindPathUsingAstar 함수를 만들어 적의 위치에서 플레이어 위치를 찾을 수 있다면 true를 리턴시키도록 하였습니다. 
```c#
public bool FindPathUsingAStar(Vector3 playerPos, Vector3 enemyPos)
{
    Node start_node = grid.GetNodePosFromWorldPosition(playerPos);
    Node goal_node = grid.GetNodePosFromWorldPosition(enemyPos);

    Heap<Node> openList = new Heap<Node>(grid.MaxSize);
    HashSet<Node> closeList = new HashSet<Node>();

    //Push Start node onto the Open List.
    openList.Add(start_node);

    Node parent_node;

    //Loop
    while (openList.Count > 0)
    {
        parent_node = openList.RemoveFirst();

        //if node == goal -> path finding.
        if (parent_node == goal_node)
        {
            stepPath = goal_node.parent.worldPos;
            step_goal_node = goal_node.parent;

            return true;
        }

        int currCost;

        foreach (Node child in grid.GetNeighborNodes(parent_node))
        {
            if (closeList.Contains(child) || !child.canWalk)
            {
                continue;
            }

            currCost = parent_node.gCost + GetDistance(parent_node, child);

            if (currCost < child.gCost || !openList.Contains(child))
            {
                child.gCost = currCost;
                child.hCost = GetDistance(child, goal_node);
                child.parent = parent_node;

                if (!openList.Contains(child))
                {
                    openList.Add(child);
                }
            }
        }

        closeList.Add(parent_node);
    }
    return false;
}
```

<br>

### **플레이어 시야**

###### 플레이어가 적을 바라볼 때 보이는 적들의 움직임을 멈추도록 구현하였습니다. 
###### EnemyInRadius 함수를 만들어 적이 플레이어 반경 안에 들어오는지 확인을 했습니다. 시야 각에 들어오더라도 멀리 떨어진 적들은 움직일 수 있도록 하기 위해서 였습니다.
```c#
private bool EnemyInRadius(Vector3 enemyPos)
{
    float distance;
    Vector3 playerPos = transform.position;
    float x = Mathf.Pow((enemyPos.x - playerPos.x), 2);
    float y = Mathf.Pow((enemyPos.z - playerPos.z), 2);

    distance = Mathf.Pow(x + y, 0.5f);

    if(distance <= viewRadius)
    {
        return true;
    }
    return false;
}

```
###### 반경 안에 들어온 적들 중 플레이어가 볼 수 있는 적을 확인하고 그 적의 isVisibleFromPlayer를 true로 바꾸어 멈출 수 있도록 했습니다.
```c#
void FindVisibleEnemy()
{
    Transform enemy;
    Vector3 dir;
    float distance;

    for (int i = 0; i < enemies.Length; ++i)
    {
        enemy = enemies[i];
        dir = (enemy.position - transform.position).normalized;

        if(EnemyInRadius(enemy.position))
        {
            if (Vector3.Angle(transform.forward, dir) < viewAngle / 2)
            {
                distance = Vector3.Distance(transform.position, enemy.position);

                if (!Physics.Raycast(transform.position, dir, distance, wallMask))
                {
                    //visible
                    enemy.GetComponent<EnemyAI>().isVisibleFromPlayer = true;
                    continue;
                }
            }
        }

        enemy.GetComponent<EnemyAI>().isVisibleFromPlayer = false;
    }
}
```
<br>

### **Cellphone UI**
![alt text](
  /assets/images/freeze_tag/FreezeTag_CellPhoneUI.png)
  
###### 플레이어가 게임 중 미니맵, 탈출을 시킨 NPC 수, Health 수, 모은 보석 수에 대해서 확인 할 수 있도록 휴대폰 UI를 구현하였습니다.
###### 모든 조건들이 맞아 떨어지는지 DoesGameEnd()함수를 불러 확인하고 게임을 종료할 수 있게 하였습니다.
```c#
public void DoesGameEnd()
{

    if (count_released_npc == npc_images.Length)
    {
        if (gem_current_count == gem_total_count)
        {
            is_end_the_game = true;
        }
    }
}
```

<br>

## **배운 점**
###### 이 프로젝트에서 A* 알고리즘을 직접 구현하여 AI를 적용시켜 보았습니다. Unity에서 제공하는 NavMesh 대신 격자 기반의 경로 탐색을 선택하였는데, 성능은 Unity의 AI 함수보다 떨어졌지만 이 과정을 통해 A* 알고리즘의 작동 원리를 더 깊이 이해하게 되었습니다.
###### 처음으로 VR을 활용한 게임을 제작하며 VR 컨트롤러와 상호작용하는 방법을 익혔습니다. 개발 중 디버깅 과정에서는 항상 VR 기기를 착용할 수 없었기 때문에, 키보드를 이용한 테스트 컨트롤러를 별도로 구현했습니다. 이로 인해 디버깅 및 테스트 단계에서 개발 효율성을 높일 수 있었지만, 이후 실제 VR 환경으로 다시 전환하는 과정이 복잡해지는 문제도 겪었습니다. 이를 통해 VR 개발의 효율성을 높이는 방법과 전환 과정의 복잡성을 줄이기 위한 전략의 필요성을 깨달았습니다.


<br>
<br>
<br>
<br>
<br>
<br>