```C++
总而言之，bingo加的性能信息是区分PC+Offset PC+Address分别发出了多少预取，分别cover了多少miss，分别over_predict了多少的，over_predict的逻辑是没被访问干掉，就被踢出了。
ROI和final分不清楚啥意思暂时
```