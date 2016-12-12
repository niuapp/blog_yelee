---
layout: post
title: Android对话框和通知模板代码
date: 2016-12-12 20:27:36 
comments: true
categories:
	- Android
tags: 
	- 学习笔记
---

—— 关于Android常用对话框和通知的代码

<!-- more -->

<br/>
## 概述
<br/>

都是基本代码，仅作记录。

> 对话框样式在不同的api版本可能不一样，所以实际使用要确定一个样式，或者自定义对话框View。

<br/>
## 一般对话框
<br/>

### 通知对话框
<br/>

```java

	Builder builder = new Builder(this);
	
	//禁止不可撤销
	builder.setCancelable(false);
	//设置标题
	builder.setTitle("对话框");
	//设置提示信息
	builder.setMessage("提示信息");
	//设置确认 取消
	builder.setPositiveButton("确认", new OnClickListener() {
		
		@Override
		public void onClick(DialogInterface dialog, int which) {
			//这是确定按钮
			Toast.makeText(MainActivity.this, "这是确定按钮", 0).show();
		}
	});
	
	builder.setNegativeButton("确认", new OnClickListener() {
		
		@Override
		public void onClick(DialogInterface dialog, int which) {
			Toast.makeText(MainActivity.this, "这是取消按钮", 0).show();
		}
	});
	
	//
	builder.show();

```

### 列表对话框
<br/>

```java

	Builder builder = new Builder(this);
			
	//设置标题
	builder.setTitle("列表对话框");
	//提示信息 --- 列表对话框 不能设置这个！！
	//builder.setMessage("选择条目");
	//条目
	final String[] values = new String[]{"条目1", "条目2", "条目3", "条目4"};
	builder.setItems(values, new OnClickListener() {
		
		@Override
		public void onClick(DialogInterface dialog, int which) {
			//点击监听
			Toast.makeText(MainActivity.this, "选择了--条目"+(which+1), 0).show();
		}
	});
	
	//显示
	builder.show();

```

### 单选对话框
<br/>

```java

	Builder builder = new Builder(this);
	
	//设置标题
	builder.setTitle("单选对话框");
	//设置单选条目 	参数，默认选择，监听
	builder.setSingleChoiceItems(new String[]{"条目1", "条目2", "条目3", "条目4"}, 2, new OnClickListener() {
		
		@Override
		public void onClick(DialogInterface dialog, int which) {
			//可以在这里 进行操作
			Toast.makeText(MainActivity.this, "选择了--条目"+(which+1), 0).show();
		}
	});
	
	//设置确定按钮
	builder.setPositiveButton("确认选择", null);
	
	builder.show();

```

### 多选对话框
<br/>

```java

	Builder builder = new Builder(this);
	//设置不可撤销
	builder.setCancelable(false);
	
	//设置标题
	builder.setTitle("多选对话框");
	
	//设置多选条目	参数，默认选择，监听
	builder.setMultiChoiceItems(new String[]{"条目1", "条目2", "条目3", "条目4"}, new boolean[]{false, true, false, true}, new OnMultiChoiceClickListener() {
		
		@Override
		public void onClick(DialogInterface dialog, int which, boolean isChecked) {
			if (isChecked) {
				Toast.makeText(MainActivity.this, "选择了--条目"+(which+1), 0).show();
			}else {
				Toast.makeText(MainActivity.this, "取消选择了--条目"+(which+1), 0).show();
			}
		}
	});
	
	//确认按钮
	builder.setPositiveButton("确认", null);
	
	builder.show();

```

### 进度对话框
<br/>

```java

	final ProgressDialog pDialog = new ProgressDialog(this);
		
	//
	pDialog.setTitle("进度对话框");
	//可以设置提示消息了
	pDialog.setMessage("进度...的消息");
	
	//设置样式，水平/旋式
	pDialog.setProgressStyle(ProgressDialog.STYLE_HORIZONTAL);
	//设置最大值
	pDialog.setMax(100);
	
	//开一个线程，测试
	new Thread(){
		public void run() {
			
			int p = 0;
			
			while (true){
				pDialog.setProgress(p++);
				
				try {
					Thread.sleep(20L);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				
				if (pDialog.getProgress() == 100) {
					//关闭进度对话框
					pDialog.dismiss();
					//Toast.makeText(MainActivity.this, "over", 0).show();
					return;
				}
			}
		}
		
	}.start();
	
	
	//显示
	pDialog.show();

```

### 自定义对话框
<br/>

```java

	//获得view对象
	View view = View.inflate(this, R.layout.dialog_test, null);
	
	//创建对话框(设置边框都为0)，指定view对象
	Builder builder = new Builder(this);
	final AlertDialog dialog = builder.create();
	dialog.setView(view, 0, 0, 0, 0);
	//builder.setView(view);
	
	//设置标题
	dialog.setTitle("自定义对话框");
	
	//试试能设置 信息吗  能
	dialog.setMessage("提示信息");
	
	//初始化 控件
	final EditText pwd = (EditText) view.findViewById(R.id.pwd);
	Button save = (Button) view.findViewById(R.id.save);
	
	//给save设置点击事件
	save.setOnClickListener(new View.OnClickListener() {
		
		@Override
		public void onClick(View v) {
			//获取密码 
			System.out.println(pwd.getText().toString().trim());
			Toast.makeText(MainActivity.this, pwd.getText().toString().trim(), 0).show();
			
			//点击按钮后 关闭对话框
			dialog.dismiss();
		}
	});
	
	dialog.show();

```

### 对话框位置等属性的修改
<br/>

```java

	AlertDialog.Builder builder = new AlertDialog.Builder(SettingActivity.this);
	builder.setMessage("确认退出吗？");
	builder.setPositiveButton("确认", new DialogInterface.OnClickListener() {
	    @Override
	    public void onClick(DialogInterface dialog, int which) {
	        //确认退出
	        UIUtils.showToastSafe("退出登陆");
	        startActivity(new Intent(SettingActivity.this, MainActivity.class));
	    }
	});
	builder.setNegativeButton("取消", null);
	AlertDialog dialog = builder.create();
	Window dialogWindow = dialog.getWindow();
	// 获取dialog中的view 设置字体颜色
	//View dialogView = dialogWindow.getDecorView();
	//setViewFontColor(dialogView, 0xffff0000);
	//显示在下边
	WindowManager.LayoutParams lp = dialogWindow.getAttributes();
	lp.y = UIUtils.dip2Px(150); // 新位置Y坐标
	// dialog.onWindowAttributesChanged(lp);
	//(当Window的Attributes改变时系统会调用此函数)
	dialogWindow .setAttributes(lp);
	//dialogWindow.setGravity(Gravity.BOTTOM);
	dialogWindow.setBackgroundDrawable(new ColorDrawable(0xaa666666));

	builder.show();

```

## 发送一个通知

```java

	//获取通知管理器
	NotificationManager nManager = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
	
	//用低版本的 方法 -- 直接new
	Notification notification = new Notification(R.drawable.ic_launcher, "标题", System.currentTimeMillis());
	
	PendingIntent contentIntent = PendingIntent.getActivity(this, 123, new Intent(this, Test_a.class), PendingIntent.FLAG_ONE_SHOT);
	notification.setLatestEventInfo(this, "内容标题？", "内容", contentIntent);
	
	//设置点击后消失
	notification.flags = Notification.FLAG_AUTO_CANCEL;
	
	//通过 通知管理器 唤醒通知
	nManager.notify(1, notification);

```

<br/>
完
<br/>

> 总结：
> 对话框比较常用，这里的修改它一些属性需要看一下。