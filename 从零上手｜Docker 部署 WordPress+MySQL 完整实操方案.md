# 从零上手｜Docker 部署 WordPress+MySQL 完整实操方案
搭建 WordPress 博客，传统 LNMP 方式需要手动安装 Nginx、PHP、MySQL，还要处理版本兼容、权限问题，新手很容易卡在环境配置上。借助 Docker 容器化技术，只需要简单的配置文件，就能快速拉起 WordPress 和 MySQL，全程不用手动安装 PHP、数据库依赖，零基础也能跟着完成建站。

## 一、环境准备
### 1. 服务器要求
一台 Linux 服务器（Ubuntu / Debian / CentOS 均可），推荐 1核2G 及以上配置；
开放安全组/防火墙 **80端口**，用于网页访问。

### 2. 安装 Docker 和 Docker Compose
登录服务器终端，依次执行下面命令安装 Docker：
```bash
# 安装docker
curl -fsSL [https://get.docker.com](https://get.docker.com) | bash
# 设置开机自启
systemctl enable docker
systemctl start docker
```
安装 docker-compose：
```bash
apt install docker-compose -y
# CentOS使用 yum install docker-compose -y
```
安装完成后验证版本：
```bash
docker -v
docker-compose -v
```
输出版本号即代表安装成功。

## 二、编写 docker-compose.yml 配置文件
在服务器新建一个项目目录，存放配置与站点数据：
```bash
mkdir wp-blog && cd wp-blog
```
创建 `docker-compose.yml`
```bash
vim docker-compose.yml
```
写入以下内容：
```yaml
version: '3.8'
services:
  mysql:
    image: mysql:5.7
    container_name: wp-mysql
    volumes:
      - ./mysql_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: Root@123456
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: Wp@123456
    ports:
      - "3306:3306"

  wordpress:
    image: wordpress:latest
    container_name: wp-site
    depends_on:
      - mysql
    volumes:
      - ./wp_data:/var/www/html
    ports:
      - "80:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: mysql:3306
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: Wp@123456
      WORDPRESS_DB_NAME: wordpress
volumes:
  mysql_data:
  wp_data:
```
> ⚠️ 重要提醒：务必修改示例中的密码，使用复杂度更高的密码，防止数据库被暴力破解。

配置简单说明：
1. mysql 服务：使用 mysql5.7 镜像，数据持久化挂载到宿主机 `mysql_data` 目录，容器自动创建数据库和账号。
2. wordpress 服务：依赖 mysql 容器，网站代码挂载到 `wp_data`，映射80端口；`depends_on` 保证mysql优先启动。
3. volumes 数据卷：把数据库和网站文件保存在宿主机，**删除容器数据不会丢失**。

## 三、启动容器，安装 WordPress
在 `wp-blog` 目录执行后台启动命令：
```bash
docker-compose up -d
```
Docker 会自动拉取 mysql、wordpress 镜像并启动容器。

查看容器运行状态：
```bash
docker-compose ps
```
看到两个容器状态都是 `Up` 即为启动成功。

打开浏览器访问服务器公网IP，进入WordPress安装向导：
1. 选择站点语言；
2. 设置站点名称、管理员用户名、密码、邮箱；
3. 点击安装，完成后使用账号登录后台。

到这里，你的 WordPress 博客就已经部署完成，可以更换主题、安装插件、发布文章。

## 四、数据备份与站点迁移（重点）
容器化建站最大优势就是方便备份迁移。
站点所有数据都保存在 `mysql_data` 和 `wp_data` 文件夹。
### 备份
直接打包整个 wp-blog 目录，下载到本地或者云存储：
```bash
tar -zcvf wp-blog-backup-$(date +%Y%m%d).tar.gz wp-blog
```
### 迁移
把备份压缩包上传到新服务器，解压后进入目录，执行：
```bash
docker-compose up -d
```
网站即可恢复，文章、主题、插件全部保留。

## 五、常见踩坑排查
1. **访问IP打不开网站**
检查服务器安全组、防火墙是否放行80端口；`docker-compose ps` 确认容器正常运行。
2. **WordPress连接数据库失败**
核对yml文件中数据库账号密码；数据库主机地址必须写 `mysql:3306`，不能写 `127.0.0.1`。
3. **上传图片/插件提示大小限制**
默认php上传限制较小，可以自定义php.ini挂载到wordpress容器修改参数。
4. **容器重启后网站丢失数据**
忘记配置 volumes 挂载，容器删除重建数据就会清空，一定要保留volumes配置。

## 六、进阶优化建议
1. 绑定域名，配置反向代理，增加 HTTPS 证书，实现 https 访问；
2. 定期自动备份脚本，定时打包数据目录；
3. 限制容器资源，给 MySQL 和 WordPress 分配CPU、内存，防止抢占服务器资源；
4. 不要暴露3306端口到公网，生产环境建议删除mysql端口映射。

## 总结
使用 Docker Compose 部署 WordPress+MySQL，极大简化了传统建站复杂的环境部署流程。一套配置文件可以在本地、测试机、云服务器重复使用，环境保持一致。不管是个人学习、搭建独立博客，还是快速搭建演示站点，这套方案都非常实用。
只需要掌握基础的 Docker 命令，就能轻松完成站点部署、备份与迁移。



https://vr.justeasy.cn/view/n1793884p1ujjk50-1793884162.html
https://vr.justeasy.cn/view/1m78938uk884u343-1793884355.html
https://vr.justeasy.cn/view/117938j843fzr8x7-1793884399.html
https://vr.justeasy.cn/view/178q9390h10z1006-1793900121.html
https://vr.justeasy.cn/view/by81793d19b00152-1793900164.html
https://vr.justeasy.cn/view/16a79jm39d0019t6-1793900208.html
https://vr.justeasy.cn/view/1793r980rx02j140-1793900251.html
https://vr.justeasy.cn/view/179379505q021h83-1793900295.html
https://vr.justeasy.cn/view/817cj93950c50327-1793900338.html
https://vr.justeasy.cn/view/i17937490s03x7j0-1793900382.html
https://vr.justeasy.cn/view/1p79390m0t4ggc14-1793900425.html
https://vr.justeasy.cn/view/16179n39h0w00457-1793900469.html
https://vr.justeasy.cn/view/117i9390e60c7501-1793900513.html
https://vr.justeasy.cn/view/17930m90h05540j5-1793900556.html
https://vr.justeasy.cn/view/19s5rw7939900588-1793900600.html
https://vr.justeasy.cn/view/179or390mk06b3b2-1793900644.html
https://vr.justeasy.cn/view/h1793k8fb9006p76-1793900687.html
https://vr.justeasy.cn/view/1793990el075b619-1793900731.html
https://vr.justeasy.cn/view/w817939005w7e064-1793900775.html
https://vr.justeasy.cn/view/17b939050v88v207-1793900819.html
https://vr.justeasy.cn/view/179y39t00i85yky1-1793900863.html
https://vr.justeasy.cn/view/17s939pgc20f0895-1793900907.html
https://vr.justeasy.cn/view/v51579390096xc38-1793900950.html
https://vr.justeasy.cn/view/81a793090vj09i82-1793900994.html
https://vr.justeasy.cn/view/b179d31h90101m26-1793901038.html
https://vr.justeasy.cn/view/17o09390g1yd0p69-1793901081.html
https://vr.justeasy.cn/view/17939l01j015tw13-1793901125.html
https://vr.justeasy.cn/view/s17i9pk3901h0158-1793901169.html
https://vr.justeasy.cn/view/ti1779q3901jt245-1793901257.html
https://vr.justeasy.cn/view/17939i0v1p2o88e8-1793901300.html
https://vr.justeasy.cn/view/1t7x9q390r13f333-1793901344.html
https://vr.justeasy.cn/view/1l73l9os39013796-1793901388.html
https://vr.justeasy.cn/view/1v679tw390154o20-1793901432.html
https://vr.justeasy.cn/view/1qd79390w1p94654-1793901475.html
https://vr.justeasy.cn/view/x17hc939c0150kq6-1793901518.html
https://vr.justeasy.cn/view/1793b9k30f16c549-1793901561.html
https://vr.justeasy.cn/view/17b9390pg51c59r3-1793901608.html
https://vr.justeasy.cn/view/17943sh901gg64f0-1793901652.html
https://vr.justeasy.cn/view/h1zd79n39z016g84-1793901696.html
https://vr.justeasy.cn/view/1793n99am701n727-1793901739.html
https://vr.justeasy.cn/view/fyt31793901h7171-1793901783.html
https://vr.justeasy.cn/view/oj1e79e3940181k5-1793901827.html
https://vr.justeasy.cn/view/1pq72ez939018i59-1793901871.html
https://vr.justeasy.cn/view/tll179390y1df903-1793901915.html
https://vr.justeasy.cn/view/17x6ze9390179947-1793901958.html
https://vr.justeasy.cn/view/1z7b79k39k0199b0-1793902002.html
https://vr.justeasy.cn/view/1v1il79390i2c034-1793902046.html
https://vr.justeasy.cn/view/z1m7e9390z2n3077-1793902089.html
https://vr.justeasy.cn/view/1v793p9o0g2g2121-1793902132.html
https://vr.justeasy.cn/view/41179z3v90k21165-1793902176.html
https://vr.justeasy.cn/view/17w793902h202rb8-1793902220.html
https://vr.justeasy.cn/view/q18793u9i02q2h52-1793902264.html
https://vr.justeasy.cn/view/b1b7939802if2t95-1793902307.html
https://vr.justeasy.cn/view/179q3os90ut23399-1793902351.html
https://vr.justeasy.cn/view/rx17293l90238n82-1793902394.html
https://vr.justeasy.cn/view/1793090s2s42y3o5-1793902437.html
https://vr.justeasy.cn/view/n1l793902z0496v9-1793902481.html
https://vr.justeasy.cn/view/1790mpj39d0y2513-1793902525.html
https://vr.justeasy.cn/view/1c7coq9390225527-1793902569.html
https://vr.justeasy.cn/view/t75179390gab2838-1793902850.html
https://vr.justeasy.cn/view/17983lt9ge02w882-1793902893.html
https://vr.justeasy.cn/view/176pg9sv39029l25-1793902937.html
https://vr.justeasy.cn/view/o61q793mv9029a68-1793902981.html
https://vr.justeasy.cn/view/u179396f03n01ox3-1793903025.html
https://vr.justeasy.cn/view/10p7m93903z70547-1793903068.html
https://vr.justeasy.cn/view/g17939g0931vtv00-1793903112.html
https://vr.justeasy.cn/view/1793fxi29031ox43-1793903155.html
https://vr.justeasy.cn/view/pm17k9i390y3h187-1793903198.html
https://vr.justeasy.cn/view/17e9390j3o2o3y90-1793903241.html
https://vr.justeasy.cn/view/917t9q3903zy29x2-1793903304.html
https://vr.justeasy.cn/view/1b7uv939t0393385-1793903347.html
https://vr.justeasy.cn/view/179i3p90633794g9-1793903391.html
https://vr.justeasy.cn/view/1xr7e923d9003426-1793903437.html
https://vr.justeasy.cn/view/b13ky79390gp3468-1793903480.html
https://vr.justeasy.cn/view/1w172939037y51e2-1793903524.html
https://vr.justeasy.cn/view/17593903i25p5w75-1793903567.html
https://vr.justeasy.cn/view/7179m339035y3p99-1793903611.html
https://vr.justeasy.cn/view/g1e79w3903nr6402-1793903654.html
https://vr.justeasy.cn/view/1793b90x3yv628e6-1793903698.html
https://vr.justeasy.cn/view/xevb17939103n730-1793903742.html
https://vr.justeasy.cn/view/17939aneru03a773-1793903785.html
https://vr.justeasy.cn/view/17939gd0t13871q6-1793903828.html
https://vr.justeasy.cn/view/1p7h9gj3903886u0-1793903872.html
https://vr.justeasy.cn/view/pe17j9390309p0p3-1793903915.html
https://vr.justeasy.cn/view/1y3c79n39nf03947-1793903959.html
https://vr.justeasy.cn/view/9q17t93590398u90-1793904002.html
https://vr.justeasy.cn/view/1793ml904x03s164-1793904046.html
https://vr.justeasy.cn/view/117x939ho04r0z77-1793904089.html
https://vr.justeasy.cn/view/179139m0041u72o0-1793904132.html
https://vr.justeasy.cn/view/1ci79390j9b24164-1793904175.html
https://vr.justeasy.cn/view/1799b390u842oz07-1793904219.html
https://vr.justeasy.cn/view/gb179390il42s5a0-1793904262.html
https://vr.justeasy.cn/view/g1u793lcim904293-1793904305.html
https://vr.justeasy.cn/view/127e98379043cx37-1793904348.html
https://vr.justeasy.cn/view/1i793904x038s7v9-1793904391.html
https://vr.justeasy.cn/view/1a7k93c9i0442uz3-1793904434.html
https://vr.justeasy.cn/view/71p7893cg9f04466-1793904477.html
https://vr.justeasy.cn/view/rk17939f0425e0v9-1793904521.html
https://vr.justeasy.cn/view/127939on90324553-1793904564.html
https://vr.justeasy.cn/view/1785934904i59mt6-1793904608.html
https://vr.justeasy.cn/view/1799t390m4ile639-1793904651.html
https://vr.justeasy.cn/view/17939la0ktzy4682-1793904695.html
https://vr.justeasy.cn/view/k17y939094m7o2c6-1793904738.html
https://vr.justeasy.cn/view/1793d90476cqbe89-1793904782.html
https://vr.justeasy.cn/view/1d7ys939i0a481e3-1793904825.html
https://vr.justeasy.cn/view/17f93900d1488517-1793904869.html
https://vr.justeasy.cn/view/1x7ov930b90l4900-1793904912.html
https://vr.justeasy.cn/view/wy1l792390mz4944-1793904956.html
https://vr.justeasy.cn/view/w1ds7939zz04i988-1793904999.html
https://vr.justeasy.cn/view/i179w390meu503g1-1793905043.html
https://vr.justeasy.cn/view/1li79g39e0b50f75-1793905087.html
https://vr.justeasy.cn/view/8117939g0w5711r8-1793905130.html
https://vr.justeasy.cn/view/g1793hr97105c162-1793905173.html
https://vr.justeasy.cn/view/17u999396052wc09-1793905221.html
https://vr.justeasy.cn/view/1793o9hw0d5zz252-1793905264.html
https://vr.justeasy.cn/view/2d1h7a9o3905m295-1793905307.html
https://vr.justeasy.cn/view/k17e9y39i05h3n39-1793905350.html
https://vr.justeasy.cn/view/1tm07939m05bl382-1793905394.html
https://vr.justeasy.cn/view/179ld3906vbh17n1-1793906183.html
https://vr.justeasy.cn/view/l1egfw793906i259-1793906271.html
https://vr.justeasy.cn/view/1ym79390bka64302-1793906314.html
https://vr.justeasy.cn/view/l9179az3906h34e5-1793906357.html
https://vr.justeasy.cn/view/1i87e8e9w3906389-1793906401.html
https://vr.justeasy.cn/view/1kc7v9390vi643p3-1793906444.html
https://vr.justeasy.cn/view/179g390t464547l6-1793906488.html
https://vr.justeasy.cn/view/f4e179390u06e520-1793906532.html
https://vr.justeasy.cn/view/1793920lk6ou56u4-1793906576.html
https://vr.justeasy.cn/view/179nvi3906b60ez8-1793906620.html
https://vr.justeasy.cn/view/18e7g9k3y90j6652-1793906664.html
https://vr.justeasy.cn/view/d17u93190sj669s5-1793906707.html
https://vr.justeasy.cn/view/17k93m9q90f67338-1793906750.html
https://vr.justeasy.cn/view/41z7933p906x7c82-1793906794.html
https://vr.justeasy.cn/view/17d9z35i90568b25-1793906837.html
https://vr.justeasy.cn/view/17h7r9391068si68-1793906880.html
https://vr.justeasy.cn/view/179f39pe069zt312-1793906924.html
https://vr.justeasy.cn/view/e179391iz4076955-1793906967.html
https://vr.justeasy.cn/view/71f7et939069l9n8-1793907014.html
https://vr.justeasy.cn/view/v179390j70mm4pb6-1793907057.html
https://vr.justeasy.cn/view/t179s3o907a080a9-1793907101.html
https://vr.justeasy.cn/view/17n939097169e3u2-1793907144.html
https://vr.justeasy.cn/view/3z0179h39j07d176-1793907187.html
https://vr.justeasy.cn/view/h71797c3904721z9-1793907231.html
https://vr.justeasy.cn/view/179q369o072lk662-1793907274.html
https://vr.justeasy.cn/view/q179339w0t7730n6-1793907318.html
https://vr.justeasy.cn/view/1v79cm39r02734m9-1793907361.html
https://vr.justeasy.cn/view/17d9639073ef9972-1793907404.html
https://vr.justeasy.cn/view/1719mc390743kie6-1793907448.html
https://vr.justeasy.cn/view/ch17j939g07m47f9-1793907491.html
https://vr.justeasy.cn/view/1ul791z3907kc522-1793907534.html
https://vr.justeasy.cn/view/d0817939075a7b66-1793907577.html
https://vr.justeasy.cn/view/1nn791x398076r09-1793907621.html
https://vr.justeasy.cn/view/1wej0793907g6052-1793907664.html
https://vr.justeasy.cn/view/f1739390hzl76906-1793907707.html
https://vr.justeasy.cn/view/17a7nv9390q773d9-1793907751.html
https://vr.justeasy.cn/view/7179q3m9kt0n7782-1793907794.html
https://vr.justeasy.cn/view/1wv793wx907v82l6-1793907838.html
https://vr.justeasy.cn/view/1anr7r93a907d869-1793907881.html
https://vr.justeasy.cn/view/1q17sa939079hq13-1793907924.html
https://vr.justeasy.cn/view/1r7x93t39rz07955-1793907967.html
https://vr.justeasy.cn/view/18s7ps939x079d99-1793908010.html
https://vr.justeasy.cn/view/179d13l940q80d42-1793908054.html
https://vr.justeasy.cn/view/179a3it9y0m80825-1793908097.html
https://vr.justeasy.cn/view/1q7e9390jy5o8129-1793908140.html
https://vr.justeasy.cn/view/71i3979i3m908172-1793908183.html
https://vr.justeasy.cn/view/17hz9j390l8s21p4-1793908226.html
https://vr.justeasy.cn/view/17939088f2d5p0r8-1793908269.html
https://vr.justeasy.cn/view/1793ds9008300sf1-1793908312.html
https://vr.justeasy.cn/view/127l93d901v834a3-1793908355.html
https://vr.justeasy.cn/view/k1709390c83c8qt6-1793908400.html
https://vr.justeasy.cn/view/17lj92u39088d432-1793908443.html
https://vr.justeasy.cn/view/1793490g378p47e5-1793908486.html
https://vr.justeasy.cn/view/17939l0uwd85m1s8-1793908530.html
https://vr.justeasy.cn/view/1l7939l0m3785l61-1793908572.html
https://vr.justeasy.cn/view/z1cp793q9i08c604-1793908615.html
https://vr.justeasy.cn/view/1s7bi93908is6497-1793908658.html
https://vr.justeasy.cn/view/147939086framw89-1793908701.html
https://vr.justeasy.cn/view/1k67l9pg39e08733-1793908745.html
https://vr.justeasy.cn/view/se179390877o1jw6-1793908788.html
https://vr.justeasy.cn/view/k1xuh7939097p0g8-1793909720.html
https://vr.justeasy.cn/view/4173v9c390997852-1793909763.html
https://vr.justeasy.cn/view/p1v7wc29390979f5-1793909806.html
https://vr.justeasy.cn/view/91793930o7983ft7-1793909849.html
https://vr.justeasy.cn/view/17ojfu939n079881-1793909892.html
https://vr.justeasy.cn/view/at179l3m90s99m24-1793909935.html
https://vr.justeasy.cn/view/179b390r4rsp9967-1793909978.html
https://vr.justeasy.cn/view/17939229714s0010-1793910021.html
https://vr.justeasy.cn/view/17pg9s9391j0s053-1793910064.html
https://vr.justeasy.cn/view/1oka793914009u36-1793910107.html
https://vr.justeasy.cn/view/179z3mo91801f3c9-1793910150.html
https://vr.justeasy.cn/view/1je1793910178iq2-1793910194.html
https://vr.justeasy.cn/view/q17939z1h029e2q6-1793910237.html
https://vr.justeasy.cn/view/s17393dki9102z69-1793910281.html
https://vr.justeasy.cn/view/c1n7v93f9p103102-1793910324.html
https://vr.justeasy.cn/view/179391y03zgo3575-1793910366.html
https://vr.justeasy.cn/view/179x3h9108r35r97-1793910409.html
https://vr.justeasy.cn/view/iooqp17939104v40-1793910451.html
https://vr.justeasy.cn/view/1n79391oe0848k53-1793910495.html
https://vr.justeasy.cn/view/4wfd1g7v93910526-1793910538.html
https://vr.justeasy.cn/view/1s77939heh10g569-1793910580.html
https://vr.justeasy.cn/view/8q91e7939106p146-1793910627.html
https://vr.justeasy.cn/view/1793z910e0x6fz58-1793910670.html
https://vr.justeasy.cn/view/fzj7179391300702-1793910713.html
https://vr.justeasy.cn/view/17g93o9d1p100745-1793910756.html
https://vr.justeasy.cn/view/6017907391o078k8-1793910799.html
https://vr.justeasy.cn/view/c179391k0838fja0-1793910842.html
https://vr.justeasy.cn/view/17y0h93y9l103874-1793910885.html
https://vr.justeasy.cn/view/17p9j3s09109th17-1793910928.html
https://vr.justeasy.cn/view/j179u39s10r9x6m0-1793910971.html
https://vr.justeasy.cn/view/1793e6l9l110l0d3-1793911015.html
https://vr.justeasy.cn/view/xwf1s2793911y047-1793911058.html
https://vr.justeasy.cn/view/b1b7n939110l9389-1793911101.html
https://vr.justeasy.cn/view/17u913gp9e141132-1793911144.html
https://vr.justeasy.cn/view/147g939d11127nb5-1793911187.html
https://vr.justeasy.cn/view/179m39gp11ju3218-1793911230.html
https://vr.justeasy.cn/view/1l7939111t26bn01-1793911273.html
https://vr.justeasy.cn/view/17939y2q11730534-1793911316.html
https://vr.justeasy.cn/view/1u47e193911342e7-1793911359.html
https://vr.justeasy.cn/view/1793imhv91431390-1793911402.html
https://vr.justeasy.cn/view/1793f0w9uk1143b4-1793911445.html
https://vr.justeasy.cn/view/oqa17s939o101477-1793911488.html
https://vr.justeasy.cn/view/0n127939by1b1520-1793911531.html
https://vr.justeasy.cn/view/y01bh79391s15c63-1793911574.html
https://vr.justeasy.cn/view/k17933910i1yx606-1793911617.html
https://vr.justeasy.cn/view/1n7h9939116v34z9-1793911660.html
https://vr.justeasy.cn/view/179393194v16ot92-1793911704.html
https://vr.justeasy.cn/view/17ad9s3911hsq735-1793911746.html
https://vr.justeasy.cn/view/1v79392ca1177et8-1793911789.html
https://vr.justeasy.cn/view/1m793921cq918521-1793911832.html
https://vr.justeasy.cn/view/e1r7e939nvd11864-1793911879.html
https://vr.justeasy.cn/view/171e93jk9119w1q0-1793911922.html
https://vr.justeasy.cn/view/z17293yu91195l43-1793911964.html
https://vr.justeasy.cn/view/1vy7t9391b1999a6-1793912007.html
https://vr.justeasy.cn/view/1790a3x911203bd9-1793912050.html
https://vr.justeasy.cn/view/117939u12r08frk2-1793912094.html
https://vr.justeasy.cn/view/a179vm3q9b1212y6-1793912137.html
https://vr.justeasy.cn/view/hmdz1p79f3912169-1793912181.html
