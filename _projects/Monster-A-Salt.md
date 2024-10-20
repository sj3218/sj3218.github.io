---
layout: page
name: Monster A Salt
tools: [Game, UE4, 3D]
image: "/assets/images/background/mas.JPG"
---

# Monster-A-Salt
<br>
{% include elements/video.html id="nuN-XHJQ6So" %}

<br>

<div style="display: flex; gap: 20px;">
  <div style="background-color: #2c2c2c; padding: 20px; border-radius: 8px; color: white; width: 50%;">
    <h2>Description</h2><br>
    <p>
       Monster-A-Salt는 활 전투를 사용하는 3D 사냥 및 요리 게임입니다. 플레이어는 보스를 물리치는 데 필요한 업그레이드를 제공하는 요리를 만드는 데 사용되는 재료를 수집하여 레벨을 통과해야 합니다. 이 프로젝트는 Unreal Engine 4.26을 사용하여 만들어졌습니다.
    </p>
    <p>
      저는 플레이어 무브먼트와 활을 이용한 싸움을 구현하였습니다.
    </p>
  </div>
  <div style="background-color: #2c2c2c; padding: 20px; border-radius: 8px; color: white; width: 50%;">
    <h2>Project Info</h2><br>
    <p>👨‍💻 직책: 게임플레이 프로그래머</p>
    <p>👥 팀 규모: 16</p>
    <p>⏳ 개발 기간: 2021.09 ~ 2021.12</p>
    <p>🛠️ Engine: Unreal Engine 4</p>
    <p>⚙️ Source code: <button onclick="window.location.href='https://drive.google.com/drive/folders/1jvtorcI4bEbvyeY4uyZ4TFEUVXOBgUJm';">Source Code 링크</button></p>
  </div>
</div>

<br>

### **플레이어 무브먼트와 활 무기**
###### 플레이어의 움직임을 구현하기 위해 MoveForward와 같은 좌 우 앞 뒤 점프 대쉬 등의 함수들을 작성하였습니다.
```c++
void ACharacterMovement::MoveForward(float AxisValue)
{
    if (Controller != NULL && AxisValue != 0.f)
    {
		FRotator rotation = Controller->GetControlRotation();
		rotation.Pitch = 0.f;

		FVector direction = FRotationMatrix(rotation).GetScaledAxis(EAxis::X);
		
		AddMovementInput(direction, AxisValue);
    }
}
```
###### 그런 다음 SetupPlayerInputComponent에서 input과 움직이는 함수를 바인딩 시켰습니다.
```c++
void ACharacterMovement::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
    Super::SetupPlayerInputComponent(PlayerInputComponent);

    PlayerInputComponent->BindAxis("MoveForward", this, &ACharacterMovement::MoveForward);
    PlayerInputComponent->BindAxis("MoveRight", this, &ACharacterMovement::MoveRight);
    PlayerInputComponent->BindAxis("TurnRate", this, &ACharacterMovement::TurnAtRate);
    PlayerInputComponent->BindAxis("LookUpRate", this, &ACharacterMovement::LookUpAtRate);
    PlayerInputComponent->BindAxis("Turn", this, &ACharacterMovement::AddControllerYawInput);
    PlayerInputComponent->BindAxis("LookUp", this, &ACharacterMovement::AddControllerPitchInput);

    PlayerInputComponent->BindAction("Jump", IE_Pressed, this, &ACharacterMovement::StartJump);
    PlayerInputComponent->BindAction("Jump", IE_Released, this, &ACharacterMovement::StopJump);
    PlayerInputComponent->BindAction("Dash", IE_Pressed, this, &ACharacterMovement::Dash);

	PlayerInputComponent->BindAction("Fire", IE_Pressed, this, &ACharacterMovement::StartCharge);
	PlayerInputComponent->BindAction("Fire", IE_Released, this, &ACharacterMovement::StopCharge);
}
```

###### 저희 게임에서는 활을 무기로 사용하는 게임이었기에 캐릭터가 활 시위를 당기는 함수가 필요하였습니다. StartCharge()에서 당기는 시간을 업데이트 시키고 canFire 변수를 true로 바꾸어 주었습니다.
```c++
void ACharacterMovement::StartCharge()
{
	UWorld* world = GetWorld();

	if (world)
	{
		FVector cameraLookVector = pCamera->GetForwardVector();
		FRotator cameraRotator = cameraLookVector.Rotation();
		FRotator ownerRotator = this->GetActorRotation();
		ownerRotator.Yaw = cameraRotator.Yaw;

		this->SetActorRotation(ownerRotator);
		chargeTime = world->GetTimeSeconds();
		canFire = true;
		checkingCharge = true;
	}
}
```
###### 이후 플레이어가 마우스 버튼을 놓았을 때 StopCharge()를 불러와서 화살이 발사 조건에 충족을 시키면 Fire()를 호출하여 화살을 발사시킵니다.
```c++
void ACharacterMovement::StopCharge()
{
	if (canFire)
	{
		UWorld* world = GetWorld();
		if (world)// calculate charge time
		{
			float worldTime = world->GetTimeSeconds();
			chargeTime = worldTime - chargeTime;
			//UE_LOG(LogTemp, Warning, TEXT("charge time : %f"), chargeTime);
			//UE_LOG(LogTemp, Warning, TEXT("rate of arrow : %f"), rateOfArrow);
		}

		if (chargeTime < minChargeTime)
		{
			//do not fire
			chargeTime = world->GetTimeSeconds();
			checkingCharge = false;
		}
		else
		{
			if (chargeTime > maxChargeTime)
			{
				chargeTime = maxChargeTime;
				arrowVelocity = maxArrowVelocity;
				pPlayerStats->SetCurrentArrowVelocity(arrowVelocity);
				//UE_LOG(LogTemp, Warning, TEXT("Arrow Velocity : %f"), arrowVelocity);
				//pPlayerStats->SetAttackDamage(damage);
				Fire();
			}
			else
			{
				float time = chargeTime - minChargeTime;
				arrowVelocity += rateOfArrow * time;
				pPlayerStats->SetCurrentArrowVelocity(arrowVelocity);
				//UE_LOG(LogTemp, Warning, TEXT("Arrow Velocity : %f"), arrowVelocity);
				//pPlayerStats->SetAttackDamage(damageOfPowerShot);
				Fire();
			}
		}
	}
}
```
###### Fire함수에서는 AArrowProjectile라는 화살을 생성하고 속도를 업데이트 시켜줍니다. 화살에는 CollisionComponent를 가지고 있어 적에게 닿았을 시 적의 hp를 닳게 하였습니다.
```c++
void ACharacterMovement::Fire()
{
	if (projectileClass)
	{
		FVector eyeLocation;
		FRotator eyeRotation;
		GetActorEyesViewPoint(eyeLocation, eyeRotation);
		
		FVector worldLocation;
		FVector worldDirection;
		FRotator newRot;
		GetWorld()->GetGameViewport()->GetViewportSize(viewPort);
		APlayerController* controller = GetWorld()->GetFirstPlayerController();

		if (controller->DeprojectScreenPositionToWorld(viewPort.X/2.f, viewPort.Y/2.f, worldLocation, worldDirection))
		{
			//UE_LOG(LogTemp, Warning, TEXT("detect viewport position"));
			//UE_LOG(LogTemp, Warning, TEXT("viewport position : %f, %f"), viewPort.X/2, viewPort.Y/2);
			newRot = UKismetMathLibrary::FindLookAtRotation(eyeLocation, -worldLocation);
		}

		FVector muzzleLocation = eyeLocation + FTransform(eyeRotation).TransformVector(MuzzleOffset);
		FRotator muzzleRotation = eyeRotation;
		muzzleRotation.Pitch += pPlayerStats->GetArrowPitch();

		UWorld* world = GetWorld();
		if (world)
		{
			FActorSpawnParameters spawnParam;
			spawnParam.Owner = this;
			spawnParam.Instigator = GetInstigator();

			AArrowProjectile* projectile = world->SpawnActor<AArrowProjectile>(projectileClass, muzzleLocation, muzzleRotation, spawnParam);
			
			if (projectile)
			{
				FVector launchDir = muzzleRotation.Vector();
				projectile->SetArrowSpeed(arrowVelocity);
				projectile->FireDirection(launchDir);
				projectile->arrowDamage = pPlayerStats->GetAttackDamage();
			}

			chargeTime = world->GetTimeSeconds(); 
			arrowVelocity = minArrowVelocity;
			pPlayerStats->SetCurrentArrowVelocity(arrowVelocity);
		}

		ResetArrow();
		//GetWorldTimerManager().SetTimer(UnusedHandle, this, &ACharacterMovement::ResetArrow, 0.0, false);
	}
}
```

<br>

## **사진**

{% capture carousel_images %}
/assets/images/monster_a_salt/monster_menu.JPG
/assets/images/monster_a_salt/monster_game.JPG
/assets/images/monster_a_salt/monster_game_2.JPG
/assets/images/monster_a_salt/monster_cook.JPG
/assets/images/monster_a_salt/monster_craft.JPG
/assets/images/monster_a_salt/monster_boss.JPG
{% endcapture %}
{% include elements/carousel.html %}

<br>

## **배운 점**
######  게임플레이 프로그래머로서 플레이어 컨트롤러와 전투 시스템을 구현하며, 엔진의 다양한 기능을 활용하는 방법을 익혔습니다.
###### 특히, 언리얼 엔진의 블루프린트 시스템을 사용해 시각적으로 스크립트를 작성하는 과정에서, 프로그래밍과 비주얼 디자인의 경계를 허물며 게임 로직을 이해하는 데 큰 도움이 되었습니다. 또한, 레벨 디자인 및 아트 자산 통합 과정을 통해 게임의 비주얼과 플레이 경험이 어떻게 조화를 이루는지를 배울 수 있었습니다.


<br>
<br>
<br>
<br>
<br>
<br>