typhoon\_animation
================
Wenlong Liu
9/10/2018

最近世界各地频发台风(飓风)，有些台风造成了严重的人身财产损失。研究台风的最基础条件就是要摸清楚台风的路径和各阶段的气象特征。这里笔者计划用R来制作台风的路径图，包括静态图和动态图。此文是一篇可重复的博文(reproducible
blog),
感兴趣的读者可以点击[传送门](https://github.com/wenlong-liu/typhoon_animation)来查看相关数据和代码。

## 导入相关扩展包和数据

``` r
require(gganimate)
require(tidyverse)
require(lubridate)
require(ggmap)
require(mapdata)
require(showtext)
library(ggmap)
library(maps)
library(mapdata)

# 中文字体设置。
#font_files()
showtext_auto(enable = TRUE)
font_add('Songti', 'Songti.ttc')

# 导入数据
typhoon_data <- read.csv("./Data/typhoon_mangkhut.csv")
#typhoon_data <- read.csv("./Data/typhoon_doksuri.csv")
```

## 数据清理

我们需要对数据做进一步的处理，包括转换时间格式和重命名经度等。

``` r
tracks <-  typhoon_data %>% 
  # 转换时间格式
  mutate(time = mdy_hm(time)) %>% 
  # 重命名经度
  rename(long = lng) 
```

## 绘制静态地图

笔者根据[浙江水利厅](http://typhoon.zjwater.gov.cn/default.aspx)的数据绘制了2018年“山竹“超强台风的路线图。山竹是2018年太平洋台风季第22个被命名的热带气旋，9月上旬在太平洋国际换日线附近形成，然后一路向西；9月16日下午1时许登陆中国香港，当地发出最高热带气旋警告信号；9月16日下午5时许在广东省江门市台山登陆，随后消失在我国广西省。山竹超强台风造成多地“停课停工“，广东省统计有四人死亡，并有大范围的经济损失。下图可以较为清晰得展示2018山竹超强台风的路径图。

``` r
tracks_box <- make_bbox(lon = tracks$long, lat = tracks$lat, f = 0.001)
sq_map <- get_map(location = tracks_box, maptype = "satellite", source = "google", zoom = 3)

p <- ggmap(sq_map) + 
  theme(text = element_text(family = "Songti"))+
  geom_point(data = tracks, mapping = aes(x = long, y = lat, color = pressure)) +
  geom_line(data = tracks, mapping = aes(x = long, y = lat, color = pressure)) +
  #geom_path(data = tracks, mapping = aes(x = long, y = lat, color = pressure))+
  scale_color_continuous(name = "中心气压(hPa)",low = "yellow", high = "red")+
  labs(title = "2018年超强台风“山竹”路径图",
       subtitle = paste("最大风速:", max(tracks$speed),"m/s,", max(tracks$power),"级台风"),
       x = "东经", y = "北纬", 
       caption = "数据来源：浙江省水利厅")+
  NULL
p
```

![](typhoon_mangkhut_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

## 绘制动态台风路径图

在上文静态路径图的基础上，笔者继续绘制了动态台风路径图。动态图采用了R中的gganimate扩展包的功能，以台风发育时间为轴，绘制了台风路径动态图。在下图的副标题中，笔者增加了动态时间显示。需要指出的是，图中的字体还有一些问题，与静态图中字体不一致；该动态图中的中文字体需要进一步的研究。

``` r
ani <- ggmap(sq_map) + 
  theme(text = element_text(family = "STFangsong",size = 18))+
  geom_point(data = tracks, mapping = aes(x = long, y = lat, color = pressure)) +
  geom_line(data = tracks, mapping = aes(x = long, y = lat, color = pressure)) +
  #geom_path(data = tracks, mapping = aes(x = long, y = lat, color = pressure))+
  scale_color_continuous(name = "中心气压(hPa)",low = "yellow", high = "red")+
  labs(title = "2017年超强台风“山竹”路径图",
       subtitle = "时间:{frame_time}",
       x = "东经", y = "北纬", 
       caption = "数据来源：浙江省水利厅")+
   transition_reveal(time,time)+
  NULL

animate(ani, renderer = ffmpeg_renderer())
```

## 保存动态图

上图是直接内置在网页中的动态图，如果读者需要将动态图保存下来，可以使用下述命令来操作。

``` r
gif_ani = animate(ani)
anim_save("mangkhut_tracking.gif", animation = gif_ani, path = "./Materials/")
```

## 总结

R是一款非常强大的数据分析与可视化工具，有兴趣的读者可以关注公众号(wliu\_2018)的更多精彩内容。
