---
published: true
layout: single
title: "C#으로 디스코드 봇 제작하기 - 기초"
classes: wide
category: .NET
sidebar:
    nav: "dotnet" 
tags: 
  - .net
  - discord
---

디스코드는 게임 커뮤니티부터 팀 프로젝트 단톡방까지 활용도가 매우 높은 메신저이다. 이번 포스트에서는 C#의 라이브러리인 Discord.NET을 활용해 디스코드 봇을 제작하는 법에 대해 다뤄볼 예정이다.

## 프로젝트 준비

Discord.NET은 디스코드의 API를 쉽게 활용하기 위해 사용하는 라이브러리이다. 이 라이브러리는 .NET Core 기반으로 프로젝트를 생성하는 것을 권장하므로 .NET Core로 프로젝트를 준비해보겠다.

프로젝트 생성은 Visual Studio 2019 기준으로 설명할 것이다.

![new_project](https://imgur.com/L3qxbN5.png)

.NET Core 프로젝트를 선택해서 생성한다.

![nuget](https://imgur.com/THHrk05.png)

솔루션 탐색기에서 프로젝트를 우클릭하고 NuGet 패키지 관리를 선택한다.

![install](https://imgur.com/dubEmgz.png)

Discord.Net 패키지를 검색해 설치버튼을 누른다.

이렇게 되면 디스코드 봇 프로젝트는 준비가 되었고, 디스코드에서 앱을 만들어야 한다.

[이 링크](https://discord.com/developers/applications)로 접속해서 우측 상단의 New Application 버튼을 클릭하고 봇의 이름을 정한 뒤 Create 버튼을 누른다.

Create 버튼을 누르면 왼쪽의 메뉴에 Bot 탭이 있는데, 탭에 들어가보면 Add Bot이라는 버튼이 있다. 버튼을 누르면 아래 사진처럼 봇의 상세정보가 생성된다. 여기서 Token 값은 개발하면서 봇의 아이디와 비밀번호 역할을 하니까, 잘 관리해야 한다.

![bot_page](https://imgur.com/cqWOowc.png)

## 기본적인 봇 만들기

프로젝트를 생성하면서 생긴 Program.cs에 봇의 코드를 작성할 것이다.

~~~cs
using Discord;
using Discord.Commands;
using Discord.WebSocket;
using System;
using System.Threading.Tasks;

namespace DiscordBot
{
    class Program
    {
        DiscordSocketClient client; //봇 클라이언트
        CommandService commands;    //명령어 수신 클라이언트
        /// <summary>
        /// 프로그램의 진입점
        /// </summary>
        /// <param name="args"></param>
        static void Main(string[] args)
        {
            new Program().BotMain().GetAwaiter().GetResult();   //봇의 진입점 실행
        }

        /// <summary>
        /// 봇의 진입점, 봇의 거의 모든 작업이 비동기로 작동되기 때문에 비동기 함수로 생성해야 함
        /// </summary>
        /// <returns></returns>
        public async Task BotMain()
        {
            client = new DiscordSocketClient(new DiscordSocketConfig() {    //디스코드 봇 초기화
                LogLevel = LogSeverity.Verbose                              //봇의 로그 레벨 설정 
            });
            commands = new CommandService(new CommandServiceConfig()        //명령어 수신 클라이언트 초기화
            {
                LogLevel = LogSeverity.Verbose                              //봇의 로그 레벨 설정
            });

            //로그 수신 시 로그 출력 함수에서 출력되도록 설정
            client.Log += OnClientLogReceived;    
            commands.Log += OnClientLogReceived;

            await client.LoginAsync(TokenType.Bot, "봇 토큰"); //봇의 토큰을 사용해 서버에 로그인
            await client.StartAsync();                         //봇이 이벤트를 수신하기 시작

            client.MessageReceived += OnClientMessage;         //봇이 메시지를 수신할 때 처리하도록 설정

            await Task.Delay(-1);   //봇이 종료되지 않도록 블로킹
        }

        private async Task OnClientMessage(SocketMessage arg)
        {
            //수신한 메시지가 사용자가 보낸 게 아닐 때 취소
            var message = arg as SocketUserMessage;
            if (message == null) return;

            int pos = 0;

            //메시지 앞에 !이 달려있지 않고, 자신이 호출된게 아니거나 다른 봇이 호출했다면 취소
            if (!(message.HasCharPrefix('!', ref pos) ||
             message.HasMentionPrefix(client.CurrentUser, ref pos)) ||
              message.Author.IsBot)
                return;

            var context = new SocketCommandContext(client, message);                    //수신된 메시지에 대한 컨텍스트 생성   

            await context.Channel.SendMessageAsync("명령어 수신됨 - " + message.Content); //수신된 명령어를 다시 보낸다.
        }

        /// <summary>
        /// 봇의 로그를 출력하는 함수
        /// </summary>
        /// <param name="msg">봇의 클라이언트에서 수신된 로그</param>
        /// <returns></returns>
        private Task OnClientLogReceived(LogMessage msg)
        {
            Console.WriteLine(msg.ToString());  //로그 출력
            return Task.CompletedTask;
        }
    }
}
~~~

## 봇 추가해서 테스트해보기

코드를 모두 작성했다면 위에서 만들었던 디스코드 앱 관리의 OAuth2 탭에 들어간다.

![oauth](https://imgur.com/BubDlc0.png)

Scopes에서 bot을 선택한 뒤 밑에서 생성된 링크를 가지고 서버에 봇을 추가해 테스트해볼 수 있다.

![join](https://imgur.com/qsNKAQR.png)

봇을 실행한 상태에서 명령어를 입력하면 정상적으로 답변이 오는 것을 볼 수 있다.

![test](https://imgur.com/dPZMLMR.png)