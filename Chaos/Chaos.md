1. MySQL修改密码

   ```shell
   # 进入mysql的bin目录
   ./mysql -u root -p
   # 输入旧密码登录mysql，在输入下面的命令
   ALTER USER 'root'@'localhost' IDENTIFIED BY 'root';
   ```

2. 终端下编译安装程序

   ```shell
   # 将程序拷贝至程序目录下
   sudo mv redis-6.0.5.tar.gz /usr/local/redis-6.0.5.tar.gz
   # 解压程序包
   sudo tar xzf redis-6.0.5.tar.gz
   # 编译测试
   sudo make test 
   # 编译安装
   sudo make install
   
   # 在编译过程中出现错误，如下命令清理了再编译： 
   sudo make distclean && sudo make && sudo make test
   ```

   