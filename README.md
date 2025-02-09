# Docker SubFinder 自动刮削字幕器 - 文件监听版

![image](https://img.shields.io/docker/pulls/freeznet/subfinder) ![GitHub last commit](https://img.shields.io/github/last-commit/rxrw/docker-subfinder-notify) ![GitHub stars](https://img.shields.io/github/stars/rxrw/docker-subfinder-notify) ![GitHub forks](https://img.shields.io/github/forks/rxrw/docker-subfinder-notify)

Docker Hub：<https://hub.docker.com/r/freeznet/subfinder>

Github: <https://github.com/rxrw/docker-subfinder-notify>

Inspired By:

<https://github.com/ausaki/subfinder>

<https://github.com/SuperNG6/docker-subfinder>

前天在 github 上偶然发现 [SuperNG6/docker-subfinder](https://github.com/SuperNG6/docker-subfinder) Docker 版本的字幕搜刮器，十分好用，美中不足的就是不支持实时监听文件改动。而我盘上几千集的柯南让我有点头疼，于是在此基础上修改了一版。

## 特点

+ 根据文件变动选择是否下载字幕
+ 根据FFMpeg分析视频文件，如果存在内嵌中文字幕，就不会去下载字幕了

## 效果图

![image](https://user-images.githubusercontent.com/9566402/131652996-7584c10a-ab98-47f7-98b2-7ca61fc0adce.png)

## 使用方法

docker run freeznet/subfinder:latest

~

### 配置

Docker 镜像中内置了当前最新可用的配置文件，忽略已存在的字幕 && 取消了标记号码，支持大部分视频格式，有需要可以将 `/config` 映射出来，编辑 `subfinder.json` 即可。

```json
{
  "languages": ["zh_en","zh", "en", "zh_chs"],
  "exts": ["ass", "srt"],
  "method": ["shooter", "zimuku"],
  "video_exts": [".mp4", ".mkv", ".iso", ".rmvb", ".avi", ".flv", ".m2ts", ".ts"],
  "exclude": [],
  "api_urls": {
    "zimuku": "https://zmk.pw/search",
    "zimuzu": "http://www.zmz2019.com/search",
    "zimuzu_api_subtitle_download": "/api/v1/static/subtitle/detail",
    "subhd": "https://subhd.tv/search",
    "subhd_api_subtitle_download": "/ajax/down_ajax",
    "subhd_api_subtitle_preview": "/ajax/file_ajax"
  },
  "ignore": true,
  "no_order_marker": true
}
```

### 环境变量

| 环境变量 | 说明 | Example |
| --- | --- | --- |
| EXCLUDE | 要排除的文件，正则表达式 | Detective* |
| FULLSCAN | 在第一次运行时首先进行全量扫描，默认不会进行 | false |
| TZ | 时区 | Asia/Shanghai |
| PUID | 用户权限 | 1027 |
| PGID | 用户组权限 | 100 |
| INTERNAL_SUB_SKIP | 内嵌字幕跳过语言。如果内嵌字幕中存在此语言的字幕，就不会下载。如果不想使用此功能，将值设置为乱码即可 | chi |

> chi 是中文，无论繁体还是简体。
> eng 是英文，好像都是语言的前三个字母小写吧

## docker-compose.yml

```yml
version: '3'

services:
  subfinder:
    image: freeznet/subfinder
    privileged: True
    restart: unless-stopped
    volumes:
      - ./subfinder:/config
      - ${VIDEO_DIR}:/media
      - /etc/timezone:/etc/timezone
      - /etc/localtime:/etc/localtime
    environment:
      EXCLUDE: "Detective"
    env_file:
      - .env
```

## 说明

1. 初次启动时如果视频库结构比较复杂，会有较长时间进行初次索引，这是 `inotifywait` 使然
2. 目前的逻辑是所有的文件变动都会调用 `subfinder`，由 `subfinder` 的过滤器负责过滤
3. 本项目也会根据主项目的新版本发布自动编译新的镜像。
