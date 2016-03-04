最近出来的青瓷引擎，底层基于已经很成熟的h5游戏引擎phaser2进行了封装，所以性能方面可以放心使用。并且提供了类似unity3d的编辑器，制作游戏很便捷。

dart是强类型语言，并且自带虚拟机，调试起来很方便，用起来比较得心应手。而且dart2js采用了tree-shaking技术，可以保证导出的文件最小。

以下是dart与青瓷的交互。

### Js部分

    //在青瓷引擎的StartUp.js中加入对应的接口
    window.SceneStart = function(data){};
    window.DispatchEvent = function(msg, args){};
    
    window.ModelCreate = function (id, model, x, y, z) {
        //类似unity3D中的预设
        G.game.assets.load(model, 'Assets/prefab/' + model + '.bin', function(asset) {
            var node = G.game.add.clone(asset);
            node.x = x;
            node.y = y;
            node.z = z;
        });
    };
    

### Dart部分

    import 'package:js/js.dart';
    
    //需要在js文件后运行
    main() {
        //设置逻辑层API的处理函数
        SceneStart = allowInterop(SceneStartHandler);
        DispatchEvent = allowInterop(DispatchEventHandler);
    }
    
    @JS() //JS方面提供的接口，供dart调用
    external void ModelCreate(int id, String model, double x, double y, double z);

    
    @JS()//需要关联到js中的SceneStart
    external set SceneStart(Function callback);
    
    @JS()//需要关联到js中的DispatchEvent
    external set DispatchEvent(Function callback);


    void SceneStartHandler(String datastr) {
        //dart方面的实现
    }

    void DispatchEventHandler(String msg, List args) {
        //dart方面的实现
    }

在开发过程中遇到比较麻烦的问题是如何进行简捷的调试。因为没有时间对青瓷引擎进行封装，所以，dart（logic）和js（view）只能分开调试。
当然，这对团队来说只有利，分工更明确了，接口更清晰了，bug更少了：）

### dart部分的调试：

接口一般不会出什么问题，直接使用web方式，加载数据进行调试就好。<br/>
逻辑封装成lib package，使用unit test进行调试，模拟view事件就可以了。

### js部的调试：

直接在青瓷提供的编辑器中调试就好，在我的架构中，视图只需要派发少数的几个事件给logic就可以了。<br/>
view部分的事件不外乎，点击按钮，点击单位，点击地板，双击单位，键盘被按下……
    
    
### 集成

dart调试完后，导出成js，然后拷贝到青瓷项目的Scripts中，并在编辑器的"Script Order Panel"中，把dart2js的文件放在StartUp.js后面，就可以进行集成测试了。



