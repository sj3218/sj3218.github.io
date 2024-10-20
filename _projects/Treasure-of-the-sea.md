---
layout: page
name: Treasuer of the Sea
tools: [Game, 3D, Unity]
image: "/assets/images/background/treasure.JPG"
---

# Treasuer of the Sea
<br>
{% include elements/video.html id="BgBBYVEOf8c" %}

<br>

<div style="display: flex; gap: 20px;">
  <div style="background-color: #2c2c2c; padding: 20px; border-radius: 8px; color: white; width: 50%;">
    <h2>Description</h2><br>
    <p>
    Treasure of the Sea는 활 전투를 사용하는 3D 오픈월드 액션 어드벤처 게임입니다. 플레이어는 재료를 수집하고 제작하여 적과 싸웁니다. 재료는 바위, 나무, 적에게서 얻을 수 있습니다. 이 게임의 주요 목표는 보스 섬을 파괴하는 것입니다. 보스 섬에 있는 상자에 모든 두개골을 수집하고 모든 적들을 쓰러뜨려야합니다.
    </p>
    <p>
      저는 이 프로젝트에서 주로 플레이어 컨트롤러와 적 AI를 구현했습니다. 이외에 플레이어가 상호작용 할 수 있는 나무, 돌, 상자와 같은 것들을 구현하였습니다.
    </p>
  </div>
  <div style="background-color: #2c2c2c; padding: 20px; border-radius: 8px; color: white; width: 50%;">
    <h2>Project Info</h2><br>
    <p>👨‍💻 직책: 게임플레이 프로그래머</p>
    <p>👥 팀 규모: 2</p>
    <p>⏳ 개발 기간: 2022.05 ~ 2021.08</p>
    <p>🛠️ Engine: Unity Engine</p>
    <p>⚙️ Github: <button onclick="window.location.href='https://github.com/wonju-cho/Treasure_of_the_Sea/tree/main/PlatinumChest/TreasureOfTheSea';">Github 링크</button></p>
  </div>
</div>

<br>

### 플레이어 
###### 플레이어 캐릭터는 무료 에셋을 다운 받아 사용하고 애니메이션은 직접 만들었습니다.
![alt text](
/assets/images/treasure_of_the_sea/player_animation.JPG)
###### 캐릭터의 움직임은 지정된 속도를 조절하여 이동시켰습니다. 플레이어 캐릭터는 평소 활을 등에 매고 돌아다니지만 적을 향해 화살을 당길 때만 활을 장착하게 구현했습니다. 
```cs
if(is_aiming)
{
    playerController.Aim();
    playerController.bowScript.EquipWeapon();

    if(Input.GetButtonUp(animStrings.aim_input))
    {
        playerController.bowScript.ReMoveCrossHair();
        shootingSFX.Play();
        CharacterFire();
        aiming_trigger = false;
        if(playerController.hitDetected == true)
        {
            Debug.Log("Detect");
            playerController.bowScript.Fire(playerController.hit.point);
        }
        else
        {
            Debug.Log("No Detect");
            playerController.bowScript.Fire(Camera.main.transform.position + Camera.main.transform.forward*10f);
        }
    }
}
else
{
    playerController.bowScript.UnEquipWeapon();
    playerController.bowScript.ReMoveCrossHair();
    playerController.bowScript.DisableArrow();
    playerController.bowScript.ReleaseString();
}

```

###### 화살 발사는 Raycast를 사용하여 화면에 cross hair 표시를 하고, 프리팹으로 만든 화살을 생성하여 포물선에 맞춰 발사되게 하였습니다. 화살이 없을 경우에는 발사를 하지 못하도록 하였습니다.

```cs
ray = Camera.main.ScreenPointToRay(centerPos);
        
if (Physics.Raycast(ray, out hit, 500f, aimLayers))
{
    hitDetected = true;
}
else
{
    hitDetected = false;
}
Debug.DrawLine(ray.origin, hit.point, Color.green);
bowScript.ShowCrosshair(hit.point);
```
```cs
public void Fire(Vector3 hitPoint)
{
    if(bowSettings.arrowCount >= 1)
    {
        canFire = true;
        Vector3 dir = (hitPoint - bowSettings.arrowPos.position).normalized;
        
        currentArrow = Instantiate(bowSettings.arrowObject, bowSettings.arrowPos.position, bowSettings.arrowPos.rotation) as Rigidbody;
        currentArrow.AddForce(dir * bowSettings.arrowForce, ForceMode.VelocityChange);
        
        bowSettings.arrowCount--;

        var arrowSlot = inventoryHolder.InventorySystem.GetInventorySlot("Arrow");
        arrowSlot.RemoveFromStack(1);

        UpdateUISlots();

    }
    else
    {
        canFire = false;
    }
}
```

<br>

### 적 AI 구현
###### 적 AI를 구현하였습니다. 플레이어와 같은 에셋이고 색만 바꾸었습니다. 적의 종류에는 melee타입과 range타입이 있습니다. 두 종류의 적은 애니메이션이 비슷하지만 Attack State부분만 다릅니다.
![alt text](
/assets/images/treasure_of_the_sea/enemy_animation.JPG)
###### 평소에는 idle 상태를 유지하며 주변을 순찰하도록 합니다. 플레이어가 일정 간격 안으로 접근을 한다면 Chase State로 바꾸었습니다.
```cs
timer += Time.deltaTime;
if (timer > 5f)
{
    animator.SetBool("isPatrolling", true);
}

float distance = Vector3.Distance(player.position, animator.transform.position);

if(distance < chaseRange)
{
    animator.SetBool("isChasing", true);
}
```
###### 플레이어가 도망을 가서 적과의 거리가 멀어지면 쫓는 것을 구만두게 하였고, 적이 플레이어에게 공격할 수 있는 거리에 도달하면 Attack State로 바꾸었습니다.

```cs
float distance = Vector3.Distance(player.position, animator.transform.position);

agent.SetDestination(player.position);

//stop chasing
if (distance > stopChasingRange)
{
    animator.SetBool("isChasing", false);
}

//stop and attack when the enemy close to the player
if (distance < attackRange)
{
    animator.SetBool("isAttacking", true);
}
```

###### 나무, 돌과 같이 적 AI는 죽을 때 일정 비율로 아이템을 제공하도록 구현하였습니다.
```cs
public void DropItem()
{
    bool dropSuccess = false;

    foreach (Loot loot in loots)
    {
        float spawnPercentage = Random.Range(-0.01f, 100f);

        if (spawnPercentage <= loot.dropRate)
        {
            dropSuccess = true;
            Instantiate(loot.item, transform.position, Quaternion.identity);
            break;
        }
    }

    if(dropSuccess == false)
    {
        Instantiate(loots[0].item, transform.position, Quaternion.identity);
    }

}
```
<br>

{% capture carousel_images %}
/assets/images/treasure_of_the_sea/treasure_of_sea_game_1.JPG
/assets/images/treasure_of_the_sea/treasure_of_sea_game_2.JPG
/assets/images/treasure_of_the_sea/treasure_of_sea_game_3.JPG
/assets/images/treasure_of_the_sea/treasure_of_sea_game_4.JPG
/assets/images/treasure_of_the_sea/treasure_of_sea_game_5.JPG
{% endcapture %}
{% include elements/carousel.html %}


<br>
<br>
<br>
<br>
<br>
<br>