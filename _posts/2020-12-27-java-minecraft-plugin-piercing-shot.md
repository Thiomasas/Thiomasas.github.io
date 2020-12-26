---
published: true
layout: single
title: "마인크래프트 플러그인 만들기 - 관통 화살"
classes: wide
category: Java
sidebar:
    nav: "java" 
tags: 
  - java
  - minecraft
---

[저번 포스트](https://fred16157.github.io/java/java-minecraft-plugin-shotgun/)에선 몬스터헌터 활처럼 퍼져나가는 화살을 발사하는 명령어를 만들어 보았다. 이번엔 명령어가 아니라, 활을 발사할 때 발생하는 이벤트를 활용해 만들어 볼 것이다.

![용화살](https://imgur.com/t1wvJSv.gif)

이번엔 이 움짤 속 기술인 몬스터헌터의 용화살처럼 관통하는 화살을 만들어볼 것이다.

이어지는 포스트입니다. 처음부터 따라하면서 읽으시려면 [여기](https://fred16157.github.io/java/java-minecraft-plugin-start/) 부터 읽어주세요.

## 이벤트 처리 클래스 수정하기

이번 포스트에서는 이 클래스에서 수정할 게 많다. 우리가 사용할 이벤트는 활이 발사될 때, 그리고 투사체가 탄착했을 때 이렇게 두가지다.

그런데 그 전에, 화살에 메타데이터를 부여해야 하기 때문에 플러그인 객체를 받아와야 한다.

이벤트 처리 클래스에서 생성자와 플러그인 객체를 다음과 같이 만든다.
~~~java
public class PluginEventListener implements Listener {
    JavaPlugin plugin;

    public PluginEventListener(JavaPlugin plugin) {
            this.plugin = plugin;
    }
    ...
~~~

그리고 진입점에서 플러그인을 넘겨주어야 한다. 그냥 플러그인 객체인 this만 인자로 넘겨주면 된다.

~~~diff
package com.example.ZizonPlugin;

import org.bukkit.plugin.java.JavaPlugin;

public class ZizonPlugin extends JavaPlugin {
    @Override
    public void onEnable() {    //플러그인 활성화시 실행
        getLogger().info("zl존 개쩌는 플러그인이 활성화되었습니다!");  //서버의 로그에 출력
        getCommand("test").setExecutor(new TestCommand());
        getCommand("shotgun").setExecutor(new ShotgunCommand(this));
-       getServer().getPluginManager().registerEvents(new PluginEventListener(), this);
+       getServer().getPluginManager().registerEvents(new PluginEventListener(this), this);
    }

    @Override
    public void onDisable() {   //플러그인 비활성화시 실행
        getLogger().info("zl존 개쩌는 플러그인이 비활성화되었습니다."); //서버의 로그에 출력
    }
}

~~~

이제 본격적으로 이벤트를 다뤄볼 시간이다. 먼저 활이 발사될 때 처리할 이벤트 처리기를 만들어야 한다.

~~~java
@EventHandler
public void onEntityShootBow(EntityShootBowEvent e) {   //활을 누군가 발사했을 때
    if(e.getEntity() instanceof Player) {   //발사한 사람이 플레이어면서
        Arrow arrow = (Arrow)e.getProjectile();
        if(arrow.isCritical()) {    //활시위를 끝까지 당기고 발사했다면
            arrow.setGravity(false);    //중력의 영향 없음
            arrow.setDamage(10);        //피격시 10데미지
            arrow.setPierceLevel(4);    //관통 레벨

            //메타데이터 설정
            arrow.setMetadata("piercing_shot", new FixedMetadataValue(plugin, true));
        }
    }
}
~~~

그리고 저번에 만들었던 투사체 착탄 이벤트 처리기인 onProjectileHit을 수정해야 한다.

~~~diff
@EventHandler
public void onProjectileHit(ProjectileHitEvent e) { //투사체 착탄시 실행
    if (e.getEntity().hasMetadata("shotgun")) {  //만약 화살이 명령어로 생성되었다면
        Arrow arrow = (Arrow) e.getEntity(); //객체를 화살로 변환
        arrow.getWorld().createExplosion(arrow.getLocation(), 1);   //화살의 착탄 위치에 폭발 생성
        arrow.remove(); //화살 삭제
    }
+   else if (e.getEntity().hasMetadata("piercing_shot") && e.getHitEntity() instanceof LivingEntity) {
+           //만약 화살이 관통 화살이면서 맞은 객체가 생물이라면
+           Arrow arrow = (Arrow) e.getEntity(); //객체를 화살로 변환
+           arrow.getWorld().createExplosion(arrow.getLocation(), 1);   //화살의 착탄 위치에 폭발 생성
+       }
}
~~~

참고로 산탄 화살도 그렇고 맞은 곳에 폭발을 일으키는 이유는 타격감 때문이다. 이펙트가 심심해서..

어쨌든 이걸로 코드 수정은 끝났다.

## 테스트

이제 플러그인을 적용하고 활을 쏴서 확인해보자.

![용화살 테스트](https://imgur.com/noetjra.gif)
