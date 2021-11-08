

# 如何使用id在JavaFx中获取元素？

我是F[XML](https://codeday.me/tag/XML)的新手,我正在尝试使用开关为所有按钮点击创建一个处理程序.但是,为了做到这一点,我需要使用和id获取元素.我尝试了以下但由于某种原因(可能是因为我在控制器类中而不是在主节点上)我得到了堆栈溢出异常.



```
public class ViewController {
public Button exitBtn;

public ViewController() throws IOException {
    Parent root = FXMLLoader.load(getClass().getResource("mainWindow.fxml"));
    Scene scene = new Scene(root);

    exitBtn = (Button) scene.lookup("#exitBtn");
}
```

}

那么我如何使用它的id作为参考获得一个元素(例如一个按钮)？

该按钮的fxml块是：

< Button fx：id =“exitBtn”contentDisplay =“CENTER”mnemonicParsing =“false”onAction =“#handleButtonClick”text =“Exit”HBox.hgrow =“NEVER”HBox.margin =“$x1”/>

解决方法:

使用控制器类,这样就不需要使用查找. FXMLLoader会将字段注入控制器.注入保证在调用initialize()方法(如果有的话)之前发生



```
public class ViewController {

    @FXML
    private Button exitBtn ;

    @FXML
    private Button openBtn ;

    public void initialize() {
        // initialization here, if needed...
    }

    @FXML
    private void handleButtonClick(ActionEvent event) {
        // I really don't recommend using a single handler like this,
        // but it will work
        if (event.getSource() == exitBtn) {
            exitBtn.getScene().getWindow().hide();
        } else if (event.getSource() == openBtn) {
            // do open action...
        }
        // etc...
    }
}
```

在FXML的根元素中指定控制器类：



```
<!-- imports etc... -->
<SomePane xmlns="..." fx:controller="my.package.ViewController">
<!-- ... -->
    <Button fx:id="exitBtn" contentDisplay="CENTER" mnemonicParsing="false" onAction="#handleButtonClick" text="Exit" HBox.hgrow="NEVER" HBox.margin="$x1" />
    <Button fx:id="openBtn" contentDisplay="CENTER" mnemonicParsing="false" onAction="#handleButtonClick" text="Open" HBox.hgrow="NEVER" HBox.margin="$x1" />
</SomePane>
```

最后,从您的控制器类以外的类加载FXML(可能,但不一定是您的Application类)



```
Parent root = FXMLLoader.load(getClass().getResource("path/to/fxml"));
Scene scene = new Scene(root);   
// etc...     
```