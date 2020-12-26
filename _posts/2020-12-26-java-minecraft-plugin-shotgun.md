---
published: true
layout: single
title: "마인크래프트 플러그인 만들기 - 산탄 화살 명령어"
classes: wide
category: Java
sidebar:
    nav: "java" 
tags: 
  - java
  - minecraft
---

[저번 포스트](https://fred16157.github.io/java/java-minecraft-plugin-command/)에선 간단하게 명령어를 추가하는 플러그인을 만들어 보았다. 이번에 무슨 기능이 좋은 예제가 될 지 고민하던 중, 몬스터헌터 월드에서 활을 사용한다면 쓸 수 있는 기술인 용의 천천시를 보고 좋은 예제가 될 것이라고 생각했다.

![천천시](https://imgur.com/yS8NbXv.gif)

이번 포스트에서는 이 기술과 비슷하게 구현을 해볼 예정이다.

이어지는 포스트입니다. 처음부터 따라하면서 읽으시려면 [여기](https://fred16157.github.io/java/java-minecraft-plugin-start/) 부터 읽어주세요.

## plugin.yml 수정하기

~~~diff
commands:
  test:
    description: 테스트 명령어입니다.
    usage: /test, /테스트, /ㅌㅅㅌ 중 하나를 치면 작동합니다.
    aliases: [테스트, ㅌㅅㅌ]
+ shotgun:
+   description: 전방에 산탄 화살을 발사합니다.
+   usage: /shotgun
+   aliases: [샷건]
~~~

plugin.yml에 샷건 명령어를 추가하면 이쪽은 이번 예제에서 수정할 일이 없다.

## 명령어 처리 클래스 

명령어 처리 클래스는 화살의 정보를 설정하고 발사하면 역할이 끝난다. 특이한 점이라면 화살에 메타데이터를 붙이기 위해 플러그인 객체를 받아서 작동하는 클래스라는 점이 있는 것 같다. 플러그인 객체를 활용하는 기능이 많기 때문에 이런 형태의 명령어 처리 클래스를 많이 만들게 될 수 있다.

~~~java
package com.example.ZizonPlugin;

import org.bukkit.command.Command;
import org.bukkit.command.CommandExecutor;
import org.bukkit.command.CommandSender;
import org.bukkit.command.ConsoleCommandSender;
import org.bukkit.entity.Arrow;
import org.bukkit.entity.Player;
import org.bukkit.metadata.FixedMetadataValue;
import org.bukkit.plugin.java.JavaPlugin;
import org.bukkit.util.Vector;

import java.util.Random;

public class ShotgunCommand implements CommandExecutor {
    JavaPlugin plugin;

    public ShotgunCommand(JavaPlugin plugin) {
        this.plugin = plugin;   //메타데이터를 위해 플러그인 객체를 갖고 있어야 함
    }

    private Vector getDir(double yaw, double dirY, double angleAdd) //바라보는 방향을 벡터로 가져오는 함수
    {
        double dirX = Math.cos(Math.toRadians(yaw + 90 + angleAdd));
        double dirZ = Math.sin(Math.toRadians(yaw + 90 + angleAdd));
        return new Vector(dirX, dirY, dirZ);
    }

    @Override
    public boolean onCommand(CommandSender sender, Command command, String label, String[] args) {  //명령어 실행 시
        if(sender instanceof Player) {  //명령어 사용자가 플레이어인 경우
            Player player = (Player)sender; //명령어 사용자 객체를 플레이어 객체로 변환할 수 있음
            Random random = new Random();
            for(int i = 0; i < 15; i++) {
                //화살을 수평으로 15개 발사
                Arrow pr = player.launchProjectile(Arrow.class, getDir(player.getLocation().getYaw(), player.getLocation().getDirection().getY(), random.nextDouble() * 45 - 22.5).multiply(3));
                pr.setMetadata("shotgun", new FixedMetadataValue(plugin, true));    //생성된 화살에 명령어로 생성되었다는 것을 표시
                pr.setGravity(false);   //중력의 영향을 받지 않게 설정하여 낙차가 없게 함
                pr.setCritical(true);   //화살의 이펙트를 위해 크리티컬 판정을 설정
                pr.setDamage(10);       //화살 피격 시 10대미지를 받도록 설정 (일반 플레이어 체력의 반)
            }
            return true;    //true값을 반환하면 명령어가 성공한 것으로 간주
        }
        else if(sender instanceof ConsoleCommandSender) {   //명령어 사용자가 콘솔인 경우
            sender.sendMessage("콘솔에서는 이 명령어를 실행할 수 없습니다.");
            return false;   //false값을 반환하면 명령어가 실패한 것으로 간주
        }
        return false;   //false값을 반환하면 명령어가 실패한 것으로 간주
    }
}
~~~

## 이벤트 처리 클래스

몬스터헌터에서 나오는 용의 천천시는 착탄지점에 폭발 이펙트가 생기기 때문에 화살이 부딪힌 시점에 폭발 이펙트를 표시하기 위해서는 이벤트 처리 클래스를 만들어야 한다.

![이벤트 클래스](https://imgur.com/34Hi8OC.png)

이벤트 처리 클래스에서는 서버 내에서 일어나는 모든 이벤트를 수신할 수 있다. 이 포스트에서는 투사체가 착탄했을 때 발생되는 이벤트를 활용할 것이다.

~~~java
package com.example.ZizonPlugin;

import org.bukkit.entity.Arrow;
import org.bukkit.event.EventHandler;
import org.bukkit.event.Listener;
import org.bukkit.event.entity.ProjectileHitEvent;

public class PluginEventListener implements Listener {
    @EventHandler
    public void onProjectileHit(ProjectileHitEvent e) { //투사체 착탄시 실행
        if(e.getEntity().hasMetadata("shotgun")) {  //만약 화살이 명령어로 생성되었다면
            Arrow arrow = (Arrow)e.getEntity(); //객체를 화살로 변환
            arrow.getWorld().createExplosion(arrow.getLocation(), 1);   //화살의 착탄 위치에 폭발 생성
            arrow.remove(); //화살 삭제
        }
    }
}
~~~

## 진입점 수정

이제 명령어 처리 클래스와 이벤트 처리 클래스가 완성되었으니 진입점을 수정할 차례다.

~~~diff
package com.example.ZizonPlugin;

import org.bukkit.plugin.java.JavaPlugin;

public class ZizonPlugin extends JavaPlugin {
    @Override
    public void onEnable() {    //플러그인 활성화시 실행
        getLogger().info("zl존 개쩌는 플러그인이 활성화되었습니다!");  //서버의 로그에 출력
        getCommand("test").setExecutor(new TestCommand());
+       getCommand("shotgun").setExecutor(new ShotgunCommand(this));    //산탄 화살 명령어 등록, this 키워드로 플러그인 객체 전달
+       getServer().getPluginManager().registerEvents(new PluginEventListener(), this); //서버에 이벤트 처리 클래스 등록
    }

    @Override
    public void onDisable() {   //플러그인 비활성화시 실행
        getLogger().info("zl존 개쩌는 플러그인이 비활성화되었습니다."); //서버의 로그에 출력
    }
}

~~~

진입점도 수정되었으니 이제 빌드하고 기능을 테스트해볼 시간이다.

## 테스트

이제 클라이언트에서 제대로 작동하는지 실험해보자.

![천천시1](https://imgur.com/Xq9Gi8f.gif)

화살이 날라가는 것과 폭발은 정상인 것 같다. 그럼 데미지는 어떨까?

![천천시2](https://imgur.com/4fqZrCI.gif)

몹들이 한방컷 당하는걸 보면 데미지도 제대로 들어가는 것 같다.

## 아이템 상호작용으로 명령어 실행하기

근데 이 기술을 쓸 때마다 명령어를 치는건 불편하다. 그런 의미에서 활을 들고 클릭했을 때 명령어가 실행되도록 하면 어떨까?

~~~java
@EventHandler
public void onPlayerInteract(PlayerInteractEvent e) {   //플레이어가 상호작용을 할 때 실행
    Player player = e.getPlayer();  //상호작용한 플레이어
    Action action = e.getAction();  //플레이어가 한 상호작용
    ItemStack item = e.getItem();   //플레이어가 상호작용할 때 손에 들고 있던 아이템

    //플레이어가 좌클릭을 눌렀을 때(허공을 때리거나 블럭을 때릴 때)
    if(action.equals(Action.LEFT_CLICK_AIR) || action.equals(ActionLEFT_CLICK_BLOCK)) {
        if(item != null) {  //플레이어 손에 무언가 있으면
            switch(item.getType()) {
                case BOW:   //그 무언가가 활이라면
                    player.performCommand("zizonplugin:shotgun");   //산탄화살 발사
            }
        }
    }
}
~~~

이 이벤트 처리 메소드를 위에서 만들었던 이벤트 처리 클래스에 넣고 다시 테스트 해보자.

![클릭](https://imgur.com/pNDOL5y.gif)

활을 들고 좌클릭을 누르면 화살이 나가는 모습을 볼 수 있다.