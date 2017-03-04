# 设计模式的思考

- 场景一

    创业潮来了，我们也不能落后。现在我们承包了一个工厂，它能用于生产各种各样的产品，那么我们最原始的生产模式就是，购置机器，生产产品，进行售卖。

    ```
    //定义我们需要的机器
    class machine {
        // 这个机器能生产A商品
        function make(){}    
    }

    // 购买机器
    var machine = new machine();

    //生产商品
    var A = machine->make();

    //售卖商品
    A->sale();
    ```

- 场景二

    很幸运，我们的产品A成功了！但一个产品不可能一直热销下去，我们得多元化发展，为此我们需要生产另一个商品 B 来摊薄运营风险。

    那么，问题来了，原来买的机器很好，但它们只能用于生产产品A，不能用于生产产品B，怎么办？采购新的机器啊。好！说干就干。

    ```
    //定义我们需要的新机器
    class newMachine {
        function makeB(){}
    }
    ```

    嗯，新机器看起来还不错，也能生产产品 B。。。但是 Are you killing me？新机器跟旧机器的操作居然不一样！，那我们还得再进行一次培训才能生产产品 B ？不不不，这不是我们想要的。

    怎么办，能不能所有的机器都只需要相同的操作？
    
    能！但首先我们需要定义一下机器们的接口。

    > 概念
    >  
    > 所谓接口，即最低生产标准。如我们规定我们的机器必须要有以下属性，只要不同时具备以下属性的机器都不是我们想要的机器
    > - 能生产，且生产操作必须一致，都为 make
    > - 有明显的标识，能操作人一眼就知道这个机器是用来生产什么的

    那么我们的接口看起来应该是这样的：

    ```
    interface machine_interface {
            //有明显的产品类型标识
            var productionType;
            //有统一的生产操作 make
            function make(){}
    }
    ```

    好了接口已经定义好了，那么我们重新采购一批新的机器吧

    ```
    class newMachine implement machine_interface {
        var productionType = 'B';
        function make(){}
    }

    var machine = new newMachine();

    machine->make();
    ```

    好了！现在经过简单的培训，所有人都能使用旧机器来生产 A，也能使用新机器来生产 B 了。

- 场景三

    产品 B 一推出就大获好评，供不应求，为了获得更多的产能，我们不能停留在小作坊的模式了，必须得规范化我们的生产流程，正式进入工厂模式。

    > 概念
    >
    > 所谓工厂模式，就是引入自动化的生产机器，只需要我们规定好产品 B 具体的生产流程，机器就会按我们规定的流程去生产，而操作人不需要知道具体的生产流程。

    这样的机器看起来应该是这样的：

    ```
    class autoMachine() {
            //每个机器都有自己的生产标准
            const standrad = 'B';
            //机器会按照生产标准去生产
            function make(){
                return makWithStandrad(this.standrad);
            }
    }

    var machine = new autoMachine();

    machine->make();

    ```

- 场景四

    结合用户的反馈，我们已经成功研发出了新一代的产品 C！
    
    但是问题又来了，原来的机器因为出厂的时候写好了产品 B 的生产标准，因此只能用于生产产品 B。如果想要生产产品 C，必要要重新采购一批写好产品 C 生产流程的机器。

    这不符合基本法啊！为了环（sheng）保（qian），我们需要一种新的机器，它应该由我来设定生产标准，而不是厂家。

    这样的机器看起来应该是这样的：

    ```
    class newAutoMachine(){
            //每个机器都有一个存放生产标准的地方
            var standrad;
            //每个机器生产之前都必须先定义好生产标准
            function _construct(){
                this.standrad = new Standrad();
            }
            //机器会按生产标准去生产
            funtion make(){
                return makeWithStandrad(this.standrad);
            }
    }
    ```

    而我们要做的就是根绝厂家给的「生产标准接口」去定义一个生产标准就可以让机器按我们的要求去生产了。

    ```
    // 厂家定义的生产标准接口
    interface standrad_interface{}
    // 我们定义的符合标准的生产标准
    class Standrad implement standrad_interface{}

    var machine = new newAutoMachine();

    machine->make();
    ```

    好了，现在我们能自定义机器的生产标准了。

- 场景五
    
    新的机器看起来已经很完美了，但是还有点美中不足，每次我需要生产不同的产品，我都要去改生产标准，这很不科学啊。

    它能不能根据给定的不同的生产标准，去生产不同的产品呢！这也就是就是所谓的「依赖注入」模式。

    符合我们需求的机器看起来应该是这样的：

    ```
    class newestAutoMachine(){
            //每个机器都有一个存放生产标准的地方
            var standrad;
            //每个机器生产之前都必须传入生产标准
            function _construct(Standrad standrad){
                this.standrad = standrad;
            }
            //机器会按生产标准去生产
            funtion make(){
                return makeWithStandrad(this.standrad);
            }
    }
    ```

    新的机器有什么不同呢？在生产标准的传入方式上！

    ```
    // 我们定义的符合标准的产品B的生产标准
    class StandradB implement standrad_interface{}
    // 我们定义的符合标准的产品C的生产标准
    class StandradC implement standrad_interface{}

    //创建生产标准
    var standradB = new StandradB();
    //启动机器并传入产品B的生产标准
    new machineB = new newestAutoMachine(standradB);
    //因为传入了产品B的生产标准，因而生产出的产品就是B
    machineB->make();

    //创建生产标准
    var standradC = new StandradC();
    //启动机器并传入产品C的生产标准
    new machineC = new newestAutoMachine(standradC);
    //因为传入了产品C的生产标准，因而生产出的产品就是C
    machineC->make();
    ```

    好了，现在这个机器会根据我们传入的不同标准，而去生产不同的产品了。

- 总结

    「依赖注入」跟「控制反转」经常是成对的出现，它们是什么关系呢？结论是，「控制反转」是一种思想，而「依赖注入」则是「控制反转」的一种实现。
    从上面的例子来说，原来控制产出的是机器的定义，而我们则希望实现，由生产标准控制机器定义，从而控制产出，这就是所谓的「控制反转」思想。
    而「依赖注入」使得机器的生产模式，由传入的生产标准所决定！所以说「依赖注入」则是「控制反转」的一种实现。

- PS

    「依赖注入」不是「控制反转」的唯一实现。
