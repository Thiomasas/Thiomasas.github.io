---
published: true
layout: single
title: "마인크래프트 플러그인 만들기 - 명령어 등록"
classes: wide
category: Java
sidebar:
    nav: "java" 
tags: 
  - java
  - minecraft
---

[저번 포스트](https://fred16157.github.io/java/java-minecraft-plugin-start/)에서는 마인크래프트 플러그인 개발을 하기 위해 프로젝트를 세팅하고 테스트까지 해보았다. 이번 포스트에서는 플러그인에 명령어를 등록하는 방법을 소개하겠다.

이어지는 포스트입니다. 처음부터 따라하면서 읽으시려면 [여기](https://fred16157.github.io/java/java-minecraft-plugin-start/) 부터 읽어주세요.

## plugin.yml에 명령어 정보 적기

저번에 생성했던 plugin.yml에 명령어의 정보를 적어야 한다.

plugin.yml에 다음과 같은 형식으로 명령어의 정보를 적으면 된다.

~~~yml
commands:
    명령어이름:
        description: 명령어 설명
        usage: 명령어 사용법
        aliases: [다른이름1, 다른이름2]
~~~

예를 들어 test라는 명령어를 적으려면,

~~~yml
commands:
    test:
        description: 테스트 명령어입니다.
        usage: /test, /테스트, /ㅌㅅㅌ 중 하나를 치면 작동합니다.
        aliases: [테스트, ㅌㅅㅌ]
~~~

이런식으로 적으면 된다.

## 명령어 처리 클래스 만들기

명령어는 있지만 만약 명령어의 기능을 담당하는 클래스가 없다면 어떻게 될까?

당연히 명령어를 쳤을때 아무런 반응도 없을 것이고, 명령어를 친 사람은 당황할 것이다.

그래서 명령어를 처리할 클래스가 따로 필요하다.

![클래스 생성](https://imgur.com/Uj9j0RB.png)

클래스를 따로 하나 만들어주자. 이 처리 클래스에서는 명령어 사용자에게 철 10개, 금 20개, 다이아몬드 30개를 주는 기능을 구현할 것이다.

클래스 파일에 다음 내용을 붙여넣으면 된다.

~~~java
package com.example.ZizonPlugin;

import org.bukkit.Material;
import org.bukkit.command.Command;
import org.bukkit.command.CommandExecutor;
import org.bukkit.command.CommandSender;
import org.bukkit.command.ConsoleCommandSender;
import org.bukkit.entity.Player;
import org.bukkit.inventory.ItemStack;

public class TestCommand implements CommandExecutor {   //명령어 처리 클래스는 CommandExecutor 인터페이스를 상속해야 한다.
    @Override
    public boolean onCommand(CommandSender sender, Command command, String label, String[] args) {  //명령어 실행 시
        if(sender instanceof Player) {  //명령어 사용자가 플레이어인 경우
            Player player = (Player)sender; //명령어 사용자 객체를 플레이어 객체로 변환할 수 있음
            player.sendMessage("zl존 개쩌는 테스트 명령어를 실행하셨습니다.");    //사용자에게 메시지 발신
            ItemStack irons = new ItemStack(Material.IRON_INGOT);   //아이템을 ItemStack 객체를 생성해 만들 수 있음
            ItemStack golds = new ItemStack(Material.GOLD_INGOT);
            ItemStack diamonds = new ItemStack(Material.DIAMOND);

            irons.setAmount(10);    //아이템 갯수를 정할 수 있음
            golds.setAmount(20);
            diamonds.setAmount(30);

            player.getInventory().addItem(irons, golds, diamonds);  //플레이어의 인벤토리를 가져와 아이템을 집어넣음
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

## 진입점에서 명령어 등록하기

plugin.yml에서 명령어 정보도 적었고, 명령어를 처리해줄 클래스도 만들었으니 이제 진입점에서 버킷에 명령어를 등록하면 끝난다.

진입점 클래스의 onEnable 메소드 안에 한줄만 추가하면 등록된다.

~~~diff
package com.example.ZizonPlugin;

import org.bukkit.plugin.java.JavaPlugin;

public class ZizonPlugin extends JavaPlugin {
    @Override
    public void onEnable() {    //플러그인 활성화시 실행
        getLogger().info("zl존 개쩌는 플러그인이 활성화되었습니다!");  //서버의 로그에 출력
+       getCommand("test").setExecutor(new TestCommand());  //test 명령어 입력시 TestCommand 클래스를 실행하게 됨
    }
    @Override
    public void onDisable() {   //플러그인 비활성화시 실행
        getLogger().info("zl존 개쩌는 플러그인이 비활성화되었습니다."); //서버의 로그에 출력
    }
}
~~~

이제 명령어가 추가되었으니 빌드하고 테스트해보자.

## 테스트

콘솔에서 명령어를 입력하면

![콘솔](https://imgur.com/07C0IKj.png)

이렇게 명령어 실행을 막는 걸 볼 수 있고,

마인크래프트를 켜서 플레이어로 들어가면,

![클라](https://imgur.com/spqghgD.png)

이런식으로 아이템이 잘 들어온다.

다음 포스트에서는 명령어를 활용해 더 고급적인 예제를 보여줄 예정이다.