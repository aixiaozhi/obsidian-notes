DDR频率是3200Mhz，就是每秒3200M次，DDR是上升沿下降沿都传输，所以是6400MTPS
![[Pasted image 20240717185816.png]]
champsim只有在load or 下层prefetch的时候激活预取

![[Pasted image 20240717185926.png|400]]
改成no_instr，要不会报错

**统计没有prefetcher的信息先dotxt.py后txt_care.py**

找到此文件夹下.txt文件中不含"DEADLOCK!"字样的.txt文件，并保存到一个820_954.txt文件中
```shell
grep -L "DEADLOCK!" *.txt > 820_954.txt
```
删除 `all_file.txt` 中出现在 `a44_file.txt` 中的条目，并将剩余的条目保存到 `left_file.txt` 的命令：
```
grep -Fxv -f a44_file.txt all_file.txt > left_file.txt
grep -rl "DEADLOCK!" *.txt | wc -l 
```

统计当前文件夹下以...为后缀个数
```shell
ls -1 *.champsimtrace_output.txt | wc -l
```

linux下打印当前文件夹下.txt文件中不含"ChampSim completed all CPUs"字样的文件
```shell
find . -type f -name "*.txt" -exec grep -L "ChampSim completed all CPUs" {} +
```
delete_match.sh里面存的是删除maki<2的benchmark的脚本
filter<2.sh存的是do_txt.py之后找到<2的脚本