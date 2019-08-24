---
layout: post
title: '设计模式之观察者模式'
tags: [code]
---

观察者模式类似于消息队列中的发布订阅模式。我们用购物来打比方，假设某件商品卖得非常火爆，长期处于脱销的状态。由于供不应求，顾客会不时的去商店询问是否有货。顾客会去很多次商店但商店不一定有货，不仅仅顾客会疯，店里的老板和员工也会疯。到这里大家肯定已经想到了，与其让顾客不断的询问不如当有货的时候让商家主动通知顾客们来买。这种设计模式就是观察者模式。

## 未使用观察者模式设计

**商店类**

```java
package observer;

/**
 * @author: ZhangMengwei
 * @create: 2019-04-10 09:24
 * 商店类
 **/
public class Shop {

    // 商品
    private String product;

    // 初始商店无货
    public Shop() {
        this.product = "没有商品";
    }

    // 商店出货
    public String getProduct() {
        return product;
    }

    // 商店进货
    public void setProduct(String product) {
        this.product = product;
    }
}
```

**买家类**

```java
package observer;

/**
 * @author: ZhangMengwei
 * @create: 2019-04-10 09:28
 * 买家类
 **/

public class Buyer {

    //买家姓名
    private String name;

    //商店引用
    private Shop shop;

    public Buyer(String name, Shop shop) {
        this.name = name;
        this.shop = shop;
    }

    //购买商品
    public void buy() {
        System.out.print(name + "购买：");
        System.out.println(shop.getProduct());
    }
}
```

**模拟购物**

```java
package observer;

/**
 * @author: ZhangMengwei
 * @create: 2019-04-10 10:29
 * 模拟购物
 **/

public class TestMain {
    public static void main(String[] args) {
        Shop shop = new Shop();
        Buyer buyer1 = new Buyer("张三", shop);
        Buyer buyer2 = new Buyer("李四", shop);
        Buyer buyer3 = new Buyer("王五", shop);

        //模拟购物
        buyer1.buy();
        buyer2.buy();
        buyer3.buy();

        buyer1.buy();
        buyer1.buy();

        shop.setProduct("华为mate 20 pro");
        buyer1.buy();
    }
}
```

结果：

```
张三购买：没有商品
李四购买：没有商品
王五购买：没有商品
张三购买：没有商品
张三购买：没有商品
张三购买：华为mate 20 pro
```

这种设计，购物者得不停的轮询商店查看是否有商品，也只有不停的轮询才可能在上架商品的时候购买到商品。

## 使用观察者模式设计后

**商店类**

```java
package observer;

import java.util.ArrayList;
import java.util.List;

/**
 * @author: ZhangMengwei
 * @create: 2019-04-10 10:51
 * 新商店类
 **/

public class NewShop {
    // 商品
    private String product;

    // 想购买商品的用户
    private List<NewBuyer> buyerList;

    public NewShop() {
        this.product = "没有商品";
        this.buyerList = new ArrayList<>();
    }


    public void register(NewBuyer buyer) {
        this.buyerList.add(buyer);
    }

    public String getProduct() {
        return product;
    }

    public void setProduct(String product) {
        this.product = product;
        this.notifyBuyers();
    }

    private void notifyBuyers() {
        buyerList.stream().forEach(buyers -> buyers.inform());
    }

}
```
所有关注商品的买家都应先注册（订阅），比如告知商家手机号以便到货后可以接到通知，通知方法对所有买家进行迭代，并调用买家的inform方法进行告知。所以这里我们规定，对于Buyer买家必须要有inform方法，这是对各类形形色色买家的定制行为，故我们对买家类进行抽象如下。

**买家抽象类**

```java
package observer;

/**
 * @author: ZhangMengwei
 * @create: 2019-04-10 10:57
 * 新买家类
 **/

public abstract class NewBuyer {

    protected String name;

    protected NewShop shop;

    public NewBuyer(String name, NewShop shop) {
        this.name = name;
        this.shop = shop;
    }

    public abstract void inform();
}
```

抽象类的实现子类

```java
package observer;

/**
 * @author: ZhangMengwei
 * @create: 2019-04-10 22:10
 * 果粉
 **/

public class PhoneFans extends NewBuyer {
    public PhoneFans(String name, NewShop shop) {
        super(name, shop);
    }

    @Override
    public void inform() {
        String product = shop.getProduct();
        if (product.contains("iPhone")) {
            System.out.println(name + "购买了：" + product);
        }
    }
}
```

```java
package observer;

/**
 * @author: ZhangMengwei
 * @create: 2019-04-10 22:14
 * 米粉
 **/

public class MiFans extends NewBuyer{
    public MiFans(String name, NewShop shop) {
        super(name, shop);
    }

    @Override
    public void inform() {
        String product = shop.getProduct();
        if (product.contains("小米")) {
            System.out.println(name + "购买了：" + product);
        }
    }
}
```

模拟交易
```java
package observer;

/**
 * @author: ZhangMengwei
 * @create: 2019-04-10 10:29
 * 模拟购物
 **/

public class TestMain {
    public static void main(String[] args) {
        NewShop newShop = new NewShop();
        NewBuyer miFans = new MiFans("张三", newShop);
        NewBuyer iPhoneFans = new PhoneFans("李四", newShop);

        // 用户到商店注册
        newShop.register(miFans);
        newShop.register(iPhoneFans);

        // 商店到新货
        newShop.setProduct("小米9");
        newShop.setProduct("iPhone XR");
        newShop.setProduct("小米note8");

    }

}
```

这样购物者再也不需要每天徘徊于店门之外苦苦等待了。接下来某日商店到货，购买过程就这样神奇地结束了，总之，商家只要到货就会马上打电话给这些订阅买家告知可以购买了。

结果

```text
张三购买了：小米9
李四购买了：iPhone XR
张三购买了：小米note8
```

参考文章：[https://www.javazhiyin.com/20906.html](https://www.javazhiyin.com/20906.html)
