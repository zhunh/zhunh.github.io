---
layout:     post
date:       2019-07-17
categories:	后端
tags:
    - SpringBoot
    - 后端
---

>
> 2019.7.17

### 方法一

```java
	//上传文件方法
	public void uploadUtil(MultipartFile,String filepath,String filename) throws IOException {
		File uploadDir = new File(filepath);
		if(!uploadDir.exists()) {
			uploadDir.mkdirs();
		}	
		File servfile = new File(filepath+filename);
		file.transferTo(servfile);
	}
transferTo(servfile);
	}
```

### 方法二

```java
//上传文件方法
	public void uploadUtil(byte[] file,String filepath,String filename) throws IOException {
		File uploadDir = new File(filepath);
		if(!uploadDir.exists()) {
			uploadDir.mkdirs();
		}
		FileOutputStream outputStream = new FileOutputStream(filepath+filename);
		outputStream.write(file);
		outputStream.flush();
		outputStream.close();
	}
```

> 列表

+ 水果
  1. 苹果
  2. 西瓜
  3. 香蕉
+ 蔬菜
  1. 白菜
  2. 黄瓜

> 图片

![alt 博客图片](https://zhunianhong.github.io/img/goicon.png)




