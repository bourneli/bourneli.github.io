---
layout: post
title: Go语言实现Template Pattern
categories: [go]
---



[Template Pattern](https://en.wikipedia.org/wiki/Template_method_pattern)是设计模式中的经典模式，它定义一个操作中算法的骨架，而将一些步骤延迟到子类中，模板方法使得子类可以不改变算法的结构即可重定义该算法的某些特定步骤。其UML示意图如下，

![](https://upload.wikimedia.org/wikipedia/commons/2/2a/W3sDesign_Template_Method_Design_Pattern_UML.jpg)

通俗点的理解就是 ：完成一件事情，有固定的数个步骤，但是每个步骤根据对象的不同，而实现细节不同；就可以在父类中定义一个完成该事情的总方法，按照完成事件需要的步骤去调用其每个步骤的实现方法。每个步骤的具体实现，由子类完成。、

最近在用go语言开发中，发现有很多地方可以用上述模式来开发，这样可以使得代码更简洁，易于阅读，降低维护成成本。但是，go语言对面向对象编程支持不像java，scala，C++那样彻底。go原生的interface无法实现abstract方法，所以无法直接实现上述模式。因此，需要一些特殊的技巧来实现子类的方法调用。



首先，定出模板类，以及模板方法，

{% highlight go %}
// 定义模板类，使用struct，而不是interface
type DBLoader struct {
   Sql    func() string
   Assign func(row *[]map[string]string) error
}

// 实现模版方法
func (l *DBLoader)Load(db_conf *mysql.DbConfig) error {
   sql := l.Sql() // 拼接sql
   err = l.Assign(&rst)   // 解码数据
   // ... 仅用于演示，其他逻辑这里忽略
   return nil
}
{% endhighlight %}



派生子类，实现特定逻辑，
{% highlight go %}

// go继承
type MyLoader struct {
   DBLoader
   my_value string
}

// 实现sql逻辑
func (m *MyLoader) Sql() string {
   return "select * ..."
}


// 实现赋值逻辑
func (m *MyLoader) Assign(row *[]map[string]string) error {
   m.my_value = "bourneli"
   return nil
}

// 将子类特定函数赋值给父类，以便在模板函数中调用
func NewMyLoader() *MyLoader {
   m := MyLoader{}
   m.DBLoader.Sql = m.Sql // 非常重要
   m.DBLoader.Assign = m.Assign // 非常重要
   return &m
}
{% endhighlight %}



最后，Mock MySQL查询操作，测试子类逻辑是否正确执行，
{% highlight go %}

func TestDBLoader_Load(t *testing.T) {
   monkey.Patch(mysql.Query, func(d *mysql.DbConfig, query string, args ...interface{}) ([]map[string]string, error) {
      return []map[string]string{}, nil
   }
   defer monkey.UnpatchAll()

   Convey("",t, func(){
      m := NewMyLoader()
      So(m.my_value, ShouldEqual, "")
      err := m.Load(nil)
      So(err, ShouldBeNil)
      So(m.my_value, ShouldEqual, "bourneli")
   })
}

func TestDBLoader_QueryError(t *testing.T) {
   monkey.Patch(mysql.Query, func(d *mysql.DbConfig, query string, args ...interface{}) ([]map[string]string, error) {
      return []map[string]string{}, fmt.Errorf("some error")
   })
   defer monkey.UnpatchAll()

   Convey("",t, func(){
      m := NewMyLoader()
      So(m.my_value, ShouldEqual, "")
      err := m.Load(nil)
      So(err, ShouldNotBeNil)
   })
}
{% endhighlight %}



希望上面的技巧对读者有所帮助。笔者的主要参考资料为[Elegant way to implement template method pattern in Golang](https://stackoverflow.com/a/40072856/1114397)









