---
published: true
layout: single
title: "C#으로 디스코드 봇 제작하기 - 뮤직봇 기능 추가"
classes: wide
category: .NET
tags: 
  - .net
  - discord 
---

지금까지는 채팅 채널에서만 봇이 상호작용을 했지만 이번에는 음성 채널과 상호작용하여 음악을 들려주는 기능을 추가해 볼 것이다.

이어지는 포스트입니다. 처음부터 따라하면서 읽으시려면 [여기](https://fred16157.github.io/.net/csharp-discord-bot-basic/) 부터 읽어주세요.

## 라이브러리 설치

우리는 음악을 유튜브에서 가져올 것이므로 VideoLibrary를 설치해야 하고, 영상을 음악으로 변환하기 위해 Xabe.FFmpeg와 Xabe.FFmpeg.Downloader를, 음성 채널에 재생하기 위해 NAudio를 설치해야 한다. 두 라이브러리 모두 NuGet 패키지 매니저로 설치할 수 있다.

음성 채널에 음성 스트림을 전송하려면 libsodium과 opus 라이브러리가 필요한데, 이 라이브러리는 직접 다운로드해서 실행파일과 같은 경로에 두어야 한다. [여기](https://discord.foxbot.me/binaries/)에서 다운로드 할 수 있다.

## 모듈 생성

모듈 클래스를 만드는 방법은 [저번 포스트](https://fred16157.github.io/.net/csharp-discord-bot-command-modules/)에 적혀있다.

~~~cs
using Discord;
using Discord.Audio;
using Discord.Commands;
using NAudio.Wave;
using System;
using System.IO;
using System.Threading.Tasks;
using VideoLibrary;
using Xabe.FFmpeg;
using Xabe.FFmpeg.Downloader;

namespace DiscordBot
{
    public class MusicModule : ModuleBase<SocketCommandContext>
    {
        
    }
}
~~~

## 음악 다운로드

우선 VideoLibrary로 유튜브에서 지정된 링크의 영상을 다운로드 할 것이다.

~~~diff
namespace DiscordBot
{
    public class MusicModule : ModuleBase<SocketCommandContext>
    {
+       /// <summary>
+       /// !play 입력시 실행되는 함수 - 음성 채널에 노래를 재생하는 역할을 한다.
+       /// </summary>
+       /// <param name="user"></param>
+       /// <returns></returns>
+       [Command("play", RunMode = RunMode.Async)]  //RunMode를 Async로 설정하여 영상을 다운로드하고 노래를 재생하는 동안 봇이 멈추지 않도록 한다.  
+       public async Task PlayCommand(string url = null)
+       {
+           if (url == null)    //url이 매개변수로 주어지지 않았을 경우
+           {
+               await Context.Channel.SendMessageAsync("영상의 링크를 제공해주세요.");
+               return;
+           }
+
+           if (Context.Guild.CurrentUser.VoiceChannel == null) //사용자가 음성 채널에 들어가지 않은 경우
+           {
+               await Context.Channel.SendMessageAsync("음악을 재생하려면 음성 채널에 있어야 합니다.");
+               return;
+           }
+
+           await Context.Channel.SendMessageAsync("영상 다운로드 시작");
+
+           YouTube yt = YouTube.Default;   //VideoLibrary의 유튜브 인스턴스 초기화
+           YouTubeVideo video = await yt.GetVideoAsync(url);   //링크의 영상을 변수에 저장
+
+           //영상을 저장할 폴더가 없다면 생성
+           if(!Directory.Exists(Environment.CurrentDirectory + "\\audio\\"))
+               Directory.CreateDirectory(Environment.CurrentDirectory + "\\audio\\");  
+
+           string videoPath = Environment.CurrentDirectory + "\\audio\\" + video.FullName;
+           //폴더 속에 영상을 다운로드
+           await File.WriteAllBytesAsync(videoPath, await video.GetBytesAsync());
+           await Context.Channel.SendMessageAsync("영상 다운로드 완료");
+       }
    }
}
~~~

테스트는 음성 채널에 들어간 상태에서 !play (유튜브링크) 와 같이 명령어를 입력하면 된다.

![download](https://imgur.com/BOsYKSp.png)

파일 탐색기에서도 다운로드된 것을 확인할 수 있다.

![explorer](https://imgur.com/niYluk4.png)


## 영상 파일을 음성 파일로 변환

다운로드 코드가 완성되었으면, 영상 파일을 음성 파일로 만들어야 한다.

~~~diff

            //폴더 속에 영상을 다운로드
            await File.WriteAllBytesAsync(videoPath, await video.GetBytesAsync());
+
+           //FFmpeg 경로 설정
+           FFmpeg.SetExecutablesPath(Environment.CurrentDirectory);        
+           await FFmpegDownloader.GetLatestVersion(FFmpegVersion.Official);    //FFmpeg 다운로드
+
+           //영상파일에서 음성파일로 변환
+           await Conversion.ExtractAudio(
+               videoPath,
+               Path.ChangeExtension(videoPath, ".mp3")).Start();
+
+           //영상파일 삭제
+           File.Delete(videoPath);
+
+           await Context.Channel.SendMessageAsync("음성 추출 완료");
       }
    }
}
~~~

테스트해보면 mp3 파일이 하나 생기는 것을 볼 수 있다.

![conversion](https://imgur.com/cL9WXWD.png)

![explorer2](https://imgur.com/MENSZAp.png)

## 음성 채널로 재생하기

음성 변환까지 끝냈으니 NAudio를 활용해서 디스코드 음성 채널에 데이터를 전송하는 코드를 작성해보자.

~~~diff
            await Context.Channel.SendMessageAsync("음성 추출 완료");
+
+           //유저가 있는 음성 채널로 연결
+           audioClient = await ((IGuildUser) Context.User).VoiceChannel.ConnectAsync();
+
+           //음성 파일을 스트림 형태로 읽기
+           var reader = new Mp3FileReader(Path.ChangeExtension(videoPath, ".mp3"));
+           var naudio = WaveFormatConversionStream.CreatePcmStream(reader);
+
+           //음성 채널과 연결된 음성 스트림 생성
+           audioStream = audioClient.CreatePCMStream(AudioApplication.Music);
+            
+           byte[] buffer = new byte[naudio.Length];    //음성 데이터 버퍼
+
+           int count = (int)(naudio.Length - naudio.Position);  //읽어들일 데이터의 크기
+           await naudio.ReadAsync(buffer, 0, count);    //음성 파일의 데이터를 버퍼에 저장
+           await audioStream.WriteAsync(buffer, 0, count);  //버퍼의 데이터를 음성 채널 스트림에 저장
+
+           //스트림 정리
+           await audioStream.FlushAsync();
+           await ((IGuildUser) Context.User).VoiceChannel.DisconnectAsync();
+           await audioStream.DisposeAsync();
+           audioClient.Dispose();
+           await naudio.DisposeAsync();
+           await reader.DisposeAsync();
+
+           //음성 파일 삭제
+           File.Delete(Path.ChangeExtension(videoPath, ".mp3"));
        }
    }
}

~~~ 

이제 봇을 실행시키고 play 명령어를 사용하면 음성채널에서 영상의 소리를 들을 수 있다.

![play](https://imgur.com/msBzuPq.png)

