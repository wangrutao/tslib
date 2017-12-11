# tslib1.4移植到tiny4412

首先下载上面的tslib1.4压缩文件<br>
##1. <br>
进行解压后cd tslib<br>
然后vim plugins/input-raw.c<br>
由于tiny4412触摸屏驱动中没有使用ioctl，因此把static int check_fd(struct tslib_input * i)函数中的从if（！（（ioctl(******** 到if(bit & (1 << EV_SYN))全部进行注释，只留下i->using_syn = 1;<br>
然后修改static int ts_input_read(struct tslib_module_info * inf, struct ts_sample * samp, int nr)函数中的<br>
if（i->using_syn） {<br>
  ******************* <br>
  switch(ev.type) {<br>
  ******************* <br>
  case EV_ABS: <br>
    switch(ev.code) { <br>
      case 53:  //就是修改这里，把ABS_X 改成53， 具体为什么是53，我也不知道，友善就喜欢改驱动。然后我用printf打印出ev.code的值，得到53,54和48，下面两个case同理； <br>
        i->current_x = ev.value;<br>
        break;<br>
      case 54:<br>
        i->current_y = ev.value;<br>
        break;<br>
      case 48:<br>
        i->current_p = ev.value;<br>
        break;<br>
    }<br>
    ***************** <br>
  ******************* <br>
}<br>
OK， 修改完后就开始正常的tslib移植，具体百度都有，我在这再记一遍：<br>
##2.<br>
sudo apt-get install automake  <br>
sudo apt-get install autogen  <br>
sudo apt-get install libtool  <br>
下载这三个软件 <br>
##3.  <br>
mkdir -p /usr/local/tslib <br>
./configure --host=arm-none-linux-gnueabi --prefix=/usr/local/tslib --cache-file=arm-none-linuxgnueabi.cache ac_cv_func_malloc_0_nonnull=yes // host是编译器， prefix是安装位置 其他的不知道<br>
make <br>
make install<br>
##4.  <br>
cd 进入/usr/local/  <br>
tar -cvf tslib.tar.gz tslib //将tslib文件夹进行压缩 <br>
mv tslib.tar.gz -f /a_linux_station/rootfs/usr/local //将压缩文件移动到nfs文件系统的usr/local中<br>
tar -xvf tslib.tar.gz //解压缩文件 <br>
vim tslib/etc/ts_conf , 把 #module_raw input前的#号和空格消除，一定记得要顶格，不要有空格。保存退出<br>
##5.  <br>
vim 修改nfs文件系统下的/etc/profile <br>
  添加如下内容： <br>
  export TSLIB_ROOT=/usr/local/tslib  <br>
  export TSLIB_TSDEVICE=/dev/event0 (你的开发板dev下的触摸屏设备节点文件) —>使用cat /dev/event0 ,然后点击触摸屏查看终端中是否出现乱码来判断这个设备节点是否链接的触摸驱动 <br>
  export TSLIB_CALIBFILE=/etc/pointercal  <br>
  export TSLIB_CONFFILE=$TSLIB_ROOT/etc/ts.conf <br>
  export TSLIB_PLUGINDIR=$TSLIB_ROOT/lib/ts <br>
  export TSLIB_FBDEVICE=/dev/fb0  <br>
在串口终端source /etc/profile   将/etc/profile刷新  <br>
##6.  <br>
在nfs文件系统中cd /usr/local/tslib/bin 然后运行./ts_test  <br>
#OK  <br>
