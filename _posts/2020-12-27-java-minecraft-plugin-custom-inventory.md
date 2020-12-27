---
published: true
layout: single
title: "마인크래프트 플러그인 만들기 - 커스텀 인벤토리"
classes: wide
category: Java
sidebar:
    nav: "java" 
tags: 
  - java
  - minecraft
---

마인크래프트로 웬만한 대형 서버를 들어가보면 인벤토리 창에 아이템을 놓아 여러 기능을 구현하는 경우가 많다. 예를 들어 유명한 서버들 중 하나인 하이픽셀에서는 플레이하고 싶은 미니게임을 인벤토리 창에서 골라서 접속할 수 있다. 이번 포스트에서는 인벤토리 창을 활용해서 개사기템을 뽑을 수 있는 기능을 구현할 것이다. 

![하이픽셀](https://imgur.com/dbB9TvQ.png)
*이 화면은 하이픽셀의 미니게임 선택 창이다.*

이어지는 포스트입니다. 처음부터 따라하면서 읽으시려면 [여기](https://fred16157.github.io/java/java-minecraft-plugin-start/) 부터 읽어주세요.

## plugin.yml 수정하기

이번엔 인벤토리 창을 명령어로 열어볼 것이기 때문에 plugin.yml을 수정해야 한다.
다음 코드를 plugin.yml의 commands 아래에 넣어주자.

~~~yml
opitem:
    description: 개사기템을 뽑을 수 있는 인벤토리 창을 엽니다.
    usage: /opitem
~~~

## 인벤토리 클래스 만들기

우리가 인벤토리 창을 구현하기 위해서는 인벤토리 객체와 그 인벤토리의 이벤트 처리기들이 필요하다. 그래서 객체와 이벤트 처리기를 클래스로 묶어서 사용할 예정이다.

![인벤클래스 생성](https://imgur.com/4wb0Us1.png)

클래스를 하나 만들어주고, 아래처럼 인벤토리 코드를 짜보자.

~~~java
package com.example.ZizonPlugin;

import org.bukkit.Bukkit;
import org.bukkit.Material;
import org.bukkit.enchantments.Enchantment;
import org.bukkit.entity.Player;
import org.bukkit.event.Event;
import org.bukkit.event.EventHandler;
import org.bukkit.event.HandlerList;
import org.bukkit.event.Listener;
import org.bukkit.event.inventory.InventoryClickEvent;
import org.bukkit.event.inventory.InventoryDragEvent;
import org.bukkit.inventory.Inventory;
import org.bukkit.inventory.ItemStack;
import org.bukkit.inventory.meta.ItemMeta;

import java.util.Arrays;
import java.util.HashMap;
import java.util.Map;


public class OpItemInventory implements Listener {
    private final Inventory inv;    //보여줄 인벤토리 창
    private final Map<Enchantment, Integer> enchantments;   //아이템에 붙을 인챈트 목록
    public OpItemInventory() {
        inv = Bukkit.createInventory(null, 9, "템 목록");  //인벤토리 초기화
        enchantments = new HashMap<>();
        for(Enchantment e : Enchantment.values()) { //모든 인챈트를 255레벨로 부여
            enchantments.put(e, 255);
        }

        ItemStack sword = new ItemStack(Material.DIAMOND_SWORD, 1);   //다이아몬드 검 생성
        sword.addUnsafeEnchantments(enchantments);  //255레벨은 일반적인 수치가 아니니 addUnsafeEnchantments로 추가해야 한다.
        ItemMeta swordMeta = sword.getItemMeta();   //검의 메타데이터
        swordMeta.setDisplayName("개쩌는 칼");      //검의 이름 설정
        swordMeta.setLore(Arrays.asList("말 그대로 개쩌는 칼이다.")); //검의 설명 설정
        sword.setItemMeta(swordMeta);   //메타데이터 저장
        inv.addItem(sword); //인벤토리에 검 추가

        ItemStack bow = new ItemStack(Material.BOW, 1); //활 생성
        bow.addUnsafeEnchantments(enchantments);    //255레벨은 일반적인 수치가 아니니 addUnsafeEnchantments로 추가해야 한다.
        ItemMeta bowMeta = bow.getItemMeta();   //활의 메타데이터
        bowMeta.setDisplayName("개쩌는 활");    //활의 이름 설정
        bowMeta.setLore(Arrays.asList("말 그대로 개쩌는 활이다."));   //활의 설명 설정
        bow.setItemMeta(bowMeta); //메타데이터 저장
        inv.addItem(bow);   //인벤토리에 활 추가
    }

    @EventHandler
    public void onOpenOpItemInventory(OpenOpItemInventoryEvent e) {
        e.player.openInventory(inv);
    }

    @EventHandler
    public void onInventoryClick(InventoryClickEvent e) {   //인벤토리 클릭 시
        if (e.getInventory() != inv) return;    //이 인벤토리를 클릭한게 아니라면 취소
        e.setCancelled(true);   //위치 변경 취소
        ItemStack clickedItem = e.getCurrentItem(); //클릭된 아이템
        //만약 클릭된 아이템이 없다면 취소
        if (clickedItem == null || clickedItem.getType() == Material.AIR) return;

        Player player = (Player) e.getWhoClicked(); //클릭한 사람에게
        player.getInventory().addItem(clickedItem); //아이템 지급
        player.closeInventory();    //인벤토리 닫기
    }

    @EventHandler
    public void onInventoryDrag(InventoryDragEvent e) { //인벤토리 드래그 시
        if (e.getInventory() == inv) {  //만약 드래그된 인벤토리가 이 인벤토리라면
            e.setCancelled(true);   //위치 변경 취소
        }
    }
}
~~~

근데, 위의 코드에서 빨간줄이 뜨는 곳이 있을 것이다.

~~~java
@EventHandler
public void onOpenOpItemInventory(OpenOpItemInventoryEvent e) {
    e.player.openInventory(inv);
}
~~~

이부분에서 OpenOpItemInventoryEvent이 없다고 뜰건데, 정상이다. 어디에도 없는 클래스라서 그렇다.

그래서 이제 우리가 만들어줘야 한다.

## 이벤트 클래스 생성

위에서 언급된 클래스는 무엇이냐 하면, 이벤트 클래스이다. 이벤트 클래스는 이벤트 처리기가 받아오는 객체의 원본이다. 이 이벤트 클래스가 원본인 객체를 가지고 플러그인 관리자에 이벤트를 호출해달라고 요청하면 원본이 맞는 객체를 수신하고 있는 이벤트 처리기가 실행된다.

이런식으로 지금까지 활을 쏠 때, 아이템과 상호작용할 때, 투사체가 부딪힐 때마다 구현되어있던 이벤트 처리기가 발동되던 것이다.

그래서 이번에는 이벤트 클래스를 하나 만들어볼 것이다. 같은 파일에 만들기 위해 public을 떼버리고 default 접근자로 선언했지만, 파일을 하나 더 만들어서 public 클래스로 만들어도 상관은 없다.

~~~java
//이벤트 클래스는 Event를 상속해야 한다.
class OpenOpItemInventoryEvent extends Event {  
    //이벤트를 수신하는 처리기들의 목록
    private static final HandlerList HANDLERS = new HandlerList();
    //이벤트를 일으킨 플레이어
    public Player player;
    public OpenOpItemInventoryEvent(Player player) {
        this.player = player;
    }

    public HandlerList getHandlers() {
        return HANDLERS;
    }

    public static HandlerList getHandlerList() {
        return HANDLERS;
    }
}
~~~

이러면 이벤트 클래스 선언도 끝이 난다.

## 명령어 클래스 생성

명령어 클래스로 쓸 파일을 하나 생성하자.

![명령어 클래스](https://imgur.com/pgMNU58.png)

명령어 클래스에서는 이벤트 클래스를 선언하고 플러그인 관리자에 이벤트 호출 요청만 해주면 된다.

~~~java
package com.example.ZizonPlugin;

import org.bukkit.Bukkit;
import org.bukkit.command.Command;
import org.bukkit.command.CommandExecutor;
import org.bukkit.command.CommandSender;
import org.bukkit.command.ConsoleCommandSender;
import org.bukkit.entity.Player;

public class OpItemCommand implements CommandExecutor {
    @Override
    public boolean onCommand(CommandSender sender, Command command, String label, String[] args) {  //명령어 실행 시
        if(sender instanceof Player) {  //명령어 사용자가 플레이어인 경우
            //이벤트 객체 생성
            OpenOpItemInventoryEvent event = new OpenOpItemInventoryEvent((Player)sender);  
            //플러그인 관리자에 이벤트 호출 요청
            Bukkit.getPluginManager().callEvent(event);
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

## 진입점 수정

이번에도 어김없이 진입점 코드를 수정을 해주어야 한다. 이번 포스트에서는 딱 두줄만 추가해주면 된다.

~~~diff
package com.example.ZizonPlugin;

import org.bukkit.plugin.java.JavaPlugin;

public class ZizonPlugin extends JavaPlugin {
    @Override
    public void onEnable() {    //플러그인 활성화시 실행
        getLogger().info("zl존 개쩌는 플러그인이 활성화되었습니다!");  //서버의 로그에 출력
        getCommand("test").setExecutor(new TestCommand());
        getCommand("shotgun").setExecutor(new ShotgunCommand(this));
+       getCommand("opitem").setExecutor(new OpItemCommand());
        getServer().getPluginManager().registerEvents(new PluginEventListener(this), this);
+       getServer().getPluginManager().registerEvents(new OpItemInventory(), this);
    }

    @Override
    public void onDisable() {   //플러그인 비활성화시 실행
        getLogger().info("zl존 개쩌는 플러그인이 비활성화되었습니다."); //서버의 로그에 출력
    }
}

~~~

## 테스트

이제 코드 짜는 건 끝났고 테스트만 남았다. 빌드하고 인벤토리를 여는 명령어를 입력해 테스트해보자.

![테스트](https://imgur.com/Mc1qn1o.gif)