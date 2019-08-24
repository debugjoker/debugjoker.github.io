---
layout: post
title: '定时任务之Timer'
tags: [code]
---

## 1.Timer简介

有且仅有一个后台线程对多个业务线程进行定时定频率的调用，该类位于java.util包下。通过阅读该类源码可以看到有两个变量一个是TaskQueue一个是TimerThread。

![Timer类结构图](https://static.oschina.net/uploads/space/2017/1021/173200_bAJV_3700425.png)

## 2.简单使用

### 2.1简单的定时任务

   ```java
   package timer;
   
   import java.util.TimerTask;
   
   /**
    * 定时任务类
    **/
   
   public class MyTimerTask extends TimerTask {
   
       private String name;
   
       public MyTimerTask(String inputName){
           name=inputName;
       }
   
   
       @Override
       public void run() {
           //打印当前name的内容
           System.out.println("Current exec name is"+name);
       }
   
       public String getName() {
           return name;
       }
   
       public void setName(String name) {
           this.name = name;
       }
   }
   ```

   启动定时任务

   ```java
   package timer;
   
   import java.util.Timer;
   
   /**
    * 启动定时任务
    **/
   
   public class MyTimer {
       public  static void  main(String[] args){
           //1.创建一个timer实例
           Timer timer=new Timer();
           //2.创建一个MyTimerTask实例
           MyTimerTask myTimerTask=new MyTimerTask("No.1");
           //3.通过timer定时频率调用myTimerTask的业务逻辑
           // 既第一次执行是在当前时间的两秒之后，之后每隔一秒钟执行一次
           timer.schedule(myTimerTask,2000L,1000L);
       }
   }
   ```

### 2.2Timer的定时调度函数

#### 2.2.1schedule函数的四种用法

1. schedule(task,time)

   作用：在时间等于或超过time的时候执行且**仅执行一次**task，如果传入的time为过去的时间则立即执行，否则等到指定的时间才开始执行

2. schedule(task,time,period)

   参数：task-所要安排的任务、time-首次执行任务的时间、period-执行一次task的时间间隔，**单位是毫秒**

   作用：在时间等于或超过time的时候执行且一次task，之后每隔period毫秒再次执行task

3. schedule(task,delay)

   参数：task-所要安排的任务、delay-执行任务前的延迟时间，单位是毫秒

   作用：**等待delay毫秒**后执行且**仅执行一次**task

4. schedule(task,delay,period)

   参数：task-所要安排的任务、delay-执行任务前的延迟时间，单位是毫秒、period-执行一次task的时间间隔，单位是毫秒

   作用：等待delay毫秒后执行且仅执行一次task，之后**每隔period毫秒重复执行**一次task

#### 2.2.2 其他重要函数

1. TimerTask的cancel()函数

   作用：取消**当前**TimerTask里的任务

   ```java
   @Override
   public void run() {
       if (count<3){
           //打印当前name的内容
           System.out.println("Current exec name is "+name);
           //以yyyy-MM-dd HH:mm:ss的格式打印当前执行时间
           Calendar calendar=Calendar.getInstance();
           SimpleDateFormat sf=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
           System.out.println("Current exec time is"+sf.format(calendar.getTime()));
           count++;
       }else {
           cancel();
           System.out.println("Task cancel");
       }
   }
   ```

   

2. TimerTask的scheduledExecutionTime()

   作用：返回此任务最近实际执行的已安排执行的时间

   返回值：最近发生此任务执行安排的时间为Long类型

   ```java
   SimpleDateFormat sf=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
   System.out.println(sf.format(myTimerTask.scheduledExecutionTime()));
   ```

3. Timer的cancel()函数

   作用:终止此计时器，丢弃**所有**当前已安排的任务

   ```java
   public static void main(String args[]) {
       //创建Timer实例
       Timer timer = new Timer();
       //创建TimerTask实例
       MyTimerTask task1 = new MyTimerTask("task1");
       MyTimerTask task2 = new MyTimerTask("task2");
       //获取当前的执行时间并打印
       Date startTime = new Date();
       SimpleDateFormat sf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
       System.out.printin("start time is:" + sf.format(startTime));
       //task1首次执行是距离现在时间3秒后执行，之后每隔2秒执行一次
       //task2首次执行是距离现在时间1秒后执行，之后每隔2秒执行一次
       timer.schedule(task1,3000,2000);
       timer.schedule(task2,1000,2000);
       //休眠5秒
       Thread.sleep(5000);
       //获取当前的执行时间并打印
       Date cancelTime = new Date();
       System.out printin("cancel time is:" + sf.format(cancelTime));
       //取消所有任务
       timer.cancel();
       System.out.printin("Tasks all canceled！");
   }
   ```

4. Timer的purge()函数

   作用：从此计时器的任务队列中移除所要已取消的任务

   返回值：从队列中移除的任务数

#### 2.2.3 schedule和scheduleAtFixeRate的区别

**首次计划执行的时间早于当前的时间**

如果当前时间为2018年08月08日 00点00分06秒，计划任务执行时间为2018年08月08日 00点00分00秒开始执行，每两秒执行一次任务

1. schedule方法

   第一次执行时间为当前时间也就是2018年08月08日 00点00分06秒，第二次执行时间为2018年08月08日 00点00分08秒

2. scheduleAtFixeRate方法

   第一次执行时间为当前时间也就是2018年08月08日 00点00分00秒，但根据计算当前任务执行次数比计划次数少了几次，于是会补上几次，也就是00、02、04、06都会执行，接下去是08

**任务执行所需时间超出任务的执行周期间隔**

如果当前时间和计划执行时间均为2018年08月08日 00点00分00秒，当前任务执行周期间隔为每两秒执行一次，而单次执行完成任务需要五秒

1. schedule方法

   相对于上一次任务结束的时间，举例：2018年08月08日 00点00分00秒开始执行，第二次执行时间为2018年08月08日 00点00分05秒，依次类推

2. scheduleAtFixeRate方法

   相对于上一次任务开始的时间，举例：2018年08月08日 00点00分00秒开始执行，第二次执行时间为2018年08月08日 00点00分02秒，依次类推

## 3.Timer的缺陷

+ 如果TimerTask抛出RuntimeException,Timer会停止所有任务的执行
+ 管理并发任务的缺陷