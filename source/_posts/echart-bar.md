---
title: echart-bar
date: 2020-06-19 20:05:25
tags: js
categories: 
- web前端
---

## echart 柱状图带背景且柱状条顶端显示文字的效果

![企业微信截图_20200619201143.png](http://ww1.sinaimg.cn/large/006q6S48gy1gfxv22ci91j30mf0j0aau.jpg)


###  实现方式

#### 第一种

隐藏X轴，左侧文字用Y轴右侧用label标签 由于柱状图的背景颜色新增的数据， 所以在显示和hover的时候都要处理


```
       yAxis: {
            // 左侧柱状图的Y轴
            splitLine: 'none',
            axisTick: 'none',
            axisLine: 'none',
            axisLabel: {
              verticalAlign: 'top',
              align: 'center',
              padding: [-5, 50, 10, 15],
              textStyle: {
                color: '#333',
                fontSize: '14'
              }
            },
            data: ['站三', '里斯', '王二']
            // inverse: true
          },
          // 背景数据
            // 背景
            {
              barGap: '-100%', // Make series be overlap
              name: '',
              type: 'bar',
              barWidth: 15,
              silent: true, // 关键
              itemStyle: {
                color: '#F6F5FA',
                barBorderRadius: 6
              },
              label: { // 右侧文字
                show: true,
                // 通过formatter函数来返回想要的数据
                formatter: function (params) { // 关键
                  for (let i = 0; i < data.length; i++) {
                    if (params.dataIndex === i) {
                      return data[i] + '人'
                    }
                  }
                },
                position: 'right',
                textStyle: {
                  color: '#333'
                }
              },
              data: [654, 654, 654]
            },
            ]
```


![2018041500134284.png](http://ww1.sinaimg.cn/large/006q6S48gy1gfxvn7wl00j30tu0e4mzu.jpg)