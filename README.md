# 算法菜农种菜的地方~

## 域名
### 绑定域名
绑定域名分2种情况：带www和不带www的。<br>
- 带www的域名 : CNAME指向你的用户名.github.io;A记录填写IP,先ping一下allenmien.github.io的IP;
- 不带www的域名：A记录填写IP,先ping一下allenmien.github.io的IP;

### CNAME
绑定域名，需要在根目录下新建一个CNAME的文件。
- 如果你填写的是没有www的，比如 mygit.me，那么无论是访问 http://www.mygit.me 还是 http://mygit.me ，都会自动跳转到 http://mygit.me
- 如果你填写的是带www的，比如 www.mygit.me ，那么无论是访问 http://www.mygit.me 还是 http://mygit.me ，都会自动跳转到 http://www.mygit.me
- 如果你填写的是其它子域名，比如 abc.mygit.me，那么访问 http://abc.mygit.me 没问题，但是访问 http://mygit.me ，不会自动跳转到 http://abc.mygit.me

