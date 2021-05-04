---
layout: post
title:  "(Unreal) Spawn Actors"
summary: ""
author: Unreal
date: '2021-05-04 0:00:00 +0000'
category: ['Unreal']
#tags: ['Unreal Engine C++ Developer: Learn C++ and Make Video Games']
thumbnail: /assets/img/posts/thumbnail-Unreal.png
#keywords: ['C++ 글올리기', 'kw-test1']
usemathjax: false
permalink: /blog/Unreal/SpawnActors/
---

```cpp
void APawnBase::Fire()
{
	// Get ProjectileSpawnPoint Location && Rotation -> Spawn Projectile class at Location towards Rotation.
	
	if(ProjectileClass)
	{
		FVector SpawnLocation = ProjectileSpawnPoint->GetComponentLocation();
		FRotator SpawnRotation = ProjectileSpawnPoint->GetComponentRotation();

		AProjectileBase* TempProjectile = GetWorld()->SpawnActor<AProjectileBase>(ProjectileClass, SpawnLocation, SpawnRotation);
		TempProjectile->SetOwner(this);
	}
}
```