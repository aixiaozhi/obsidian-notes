 wget -A.xz -r https://dpc3.compas.cs.stonybrook.edu/champsim-traces/speccpu
 一定要-r是递归下载，所有的都下载
 -A是文件名字

rsync -avz --info=progress2 -e "ssh -p 22" /home/ainz/a_huawei_924_cstz/ys_test_no0ns_bw_withoutptw/ ainz@166.111.78.26:/home/ainz/a_huawei_924_cstz/ys_test_no0ns_bw_withoutptw
两个服务器传输文件夹

rsync -avz --info=progress2 -e "ssh -p 22" /home/ainz/a_huawei/speccpu/ ainz@166.111.78.26:/home/ainz/a_huawei/speccpu/