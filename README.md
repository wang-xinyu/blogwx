# BLOGWX

基于Pelican搭建的个人博客

## Install

```
sudo pip install pelican markdown
git clone https://github.com/wang-xinyu/wang-xinyu.github.io.git output
git clone https://github.com/getpelican/pelican-themes
cd pelican-themes/nest
git clone https://github.com/wang-xinyu/nest.git .
sudo pelican-themes -U nest
```

## 更新文章

```
make html
cd output
git add/commit/push
```

## 常见问题

1. ValueError: unknown locale: UTF-8
	
	```
	export LC_ALL=en_US.UTF-8
	export LANG=en_US.UTF-8
    ```

2. favicon不显示

    - 打开https://wang-xinyu.github.io/favicon.ico
    - 刷新
    - 重启浏览器

3. 更新主题

   在pelican-themes目录下：
		sudo pelican-themes -U themename
