nohup ps -a > output.log 2>&1 &
 ./ChampSim/Other_PF/bin/hashed_perceptron-no-ipcp_isca2020-no-no-no-no-no-lru-lru-lru-srrip-drrip-lru
-lru-lru-1core-no -warmup_instructions 500000 -simulation_instructions 2000000 -traces traces/spec2k17/602.gcc_s-734B.champsimtrace.xz

./build_champsim.sh hashed_perceptron no ipcp_isca2020 no no no no no    lru lru lru srrip drrip lru lru lru 1 no

z1015jj*

```shell
find . -type f -name "*.txt" -exec sh -c '! grep -q "LLC" "$1" && rm "$1"' _ {} \;
#删除当前目录下txt文件中所有不含LLC字段的文件
ls -l *.txt | grep -v '^d' | wc -l
#统计txt文件数
grep -rl 'LLC' *.txt | wc -l
#统计当前txt文件中含LLC的个数
```

```
bin/champsim --warmup_instructions 20000 --simulation_instructions 80000 ../spec2k17/623.xalancbmk_s-202B.champsimtrace.xz
```
 useful分为两部分，已经在MSHR中的和prefetch hit的部分
 prefetch hit是请求由prefetch发起，发现已经在cache中的line
 还是prefetch的line回来，发现正常load hit了

