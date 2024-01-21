场景 
在做直播软件的时候，需要在用户打开App后，先后弹出签到，活动，提示等一系列弹窗。每个弹窗都要在前一个弹窗消失后弹出。于是就面临一个弹窗顺序问题，那时候对设计模式很陌生，不知道怎么更好的解决弹窗顺序问题，都在下前一个弹窗取消或关闭时去加载后面一个弹窗。这样做虽然也能解决问题，但是实现并不优雅，如果在弹窗中间再添加一个其他类型的弹窗改动代价就变得很大，特别是当你是后来接手代码的新人，稍有不慎，就要背锅。怎么能简单而又优雅的解决这个问题呢？
思路
开发者必读的23种设计模式，对于日常开发问题的解决提供了很好的思路，可以说几乎所有的优秀架构都离不开设计模式，这也是面试必问问题之一。23种设计模式中有一个责任链模式，为弹窗问题提供了解决方案，这也是我从okhttp源码中学习到的，读过okhttp的同学都知道，okhttp网络请求中的五大拦截器基于链式请求，使用起来简单高效。本篇文章同样也是基于责任链的思路来解决弹窗顺序问题。
代码
1. 首页我们定义一个接口DialogIntercept，同时提供两个方法 intercept和show。
interface  DialogIntercept {
    fun intercept(dialogIntercept: DialogChain)
    fun show():Boolean
}
所有的弹窗都需要实现DialogIntercept中的这两个方法。
2. 自定义弹窗实现DailogIntercept接口。
   ● 弹窗

class FirstDialog(val context: Context) :DialogIntercept{

    override fun intercept(dialogIntercept: DialogChain) {
        
    }

    override fun show():Boolean{
        return true
    }
}
这里默认show()方法默认返回true，可根据业务逻辑决定弹窗是否显示。
3. 提供一个弹窗管理类DialogChain，通过建造者模式创建管理类。根据弹窗添加的顺序弹出。
class DialogChain(private val builder: Builder) {
     private var index = 0
     fun proceed(){
      ............
      ...省略部分代码.....
      .............
     }
    class Builder(){
        var chainList:MutableList<DialogIntercept> = mutableListOf()
        fun addIntercept(dialogIntercept: DialogIntercept):Builder{
           .....省略部分代码.....
            return this
        }
        fun build():DialogChain{
            return DialogChain(this)
        }
    }

}
效果
为了测试效果，分别定义三个弹窗，FirstDialog，SecondDialog，ThirdDialog。按照显示顺序依次添加到DialogChain弹窗管理类中。
1. 定义弹窗。
  由于三个弹窗代码基本相同，下面只提供FirstDialog代码。
class FirstDialog(val context: Context) :DialogIntercept{

   
    override fun intercept(dialogIntercept: DialogChain) {
         show(dialogIntercept)
    }

    override fun show():Boolean{
        return true
    }

    private fun show(dialogIntercept: DialogChain){
        AlertDialog.Builder(context).setTitle("FirstDialog")
            .setPositiveButton("确定"
        ) { _, _ ->
            dialogIntercept.proceed()
        }.setNegativeButton("取消"
        ) { _, _ ->
            dialogIntercept.proceed()
        }.create().show()
    }
}
2 . 分别将三个弹窗按照显示顺序添加到管理器中。
 DialogChain.Builder()
   .addIntercept(FirstDialog(this))
   .addIntercept(SecondDialog(this))
   .addIntercept(ThirdDialog(this))
    .build().proceed()
3. 实现效果如下：


总结
再优秀的架构，都离不开设计模式和设计原则。很多时候我们觉得架构师遥不可及，其实更多的时候是我们缺少一个想要进步的心。新的一年，新的起点，新的开始。

