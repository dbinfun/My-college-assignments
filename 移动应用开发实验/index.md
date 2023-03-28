# 实验一

## 要求

1. 利用线性布局，实现以下效果

   ![image-20230328175314692](./assets/index/image-20230328175314692.png)

   2. 在MainActivity中实现该计算器程序的功能，能够进行简单的4则运算，如2*8=16,4.6-8=-3.4等。

   3. “清除”按钮点击后清除整个算式。

   4. 对于不正确的输入，通过Toast对象进行错误提示，如零不能为除数、操作数位数太长等。

## 实验代码

### 1.`activity_main.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<!--线性布局-->
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <EditText
        android:id="@+id/main.text"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="0"
        />
    <Button
        android:id="@+id/main.clear_text"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="清除"/>
    <LinearLayout
        android:orientation="vertical"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
        <LinearLayout
            android:orientation="horizontal"
            android:layout_width="match_parent"
            android:layout_height="wrap_content">
        <Button
            android:id="@+id/main.num.1"
            android:layout_weight="1"
            android:text="1"
            android:layout_margin="5dp"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content" />
        <Button
            android:id="@+id/main.num.2"
            android:layout_weight="1"
            android:text="2"
            android:layout_margin="5dp"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content" />
        <Button
            android:id="@+id/main.num.3"
            android:text="3"
            android:layout_weight="1"
            android:layout_margin="5dp"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content" />
        <Button
            android:id="@+id/main.jia"
            android:text="+"
            android:layout_weight="1"
            android:layout_margin="5dp"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content" />
        </LinearLayout>
        <LinearLayout
            android:orientation="horizontal"
            android:layout_width="match_parent"
            android:layout_height="wrap_content">
            <Button
                android:id="@+id/main.num.4"
                android:text="4"
                android:layout_weight="1"
                android:layout_margin="5dp"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content" />
            <Button
                android:id="@+id/main.num.5"
                android:text="5"
                android:layout_weight="1"
                android:layout_margin="5dp"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content" />
            <Button
                android:id="@+id/main.num.6"
                android:text="6"
                android:layout_weight="1"
                android:layout_margin="5dp"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content" />
            <Button
                android:id="@+id/main.jian"
                android:text="-"
                android:layout_weight="1"
                android:layout_margin="5dp"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content" />
        </LinearLayout>
        <LinearLayout
            android:orientation="horizontal"
            android:layout_width="match_parent"
            android:layout_height="wrap_content">
            <Button
                android:id="@+id/main.num.7"
                android:text="7"
                android:layout_weight="1"
                android:layout_margin="5dp"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content" />
            <Button
                android:id="@+id/main.num.8"
                android:text="8"
                android:layout_weight="1"
                android:layout_margin="5dp"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content" />
            <Button
                android:id="@+id/main.num.9"
                android:text="9"
                android:layout_weight="1"
                android:layout_margin="5dp"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content" />
            <Button
                android:id="@+id/main.cheng"
                android:text="*"
                android:layout_weight="1"
                android:layout_margin="5dp"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content" />
        </LinearLayout>
        <LinearLayout
            android:orientation="horizontal"
            android:layout_width="match_parent"
            android:layout_height="wrap_content">
            <Button
                android:id="@+id/main.dian"
                android:text="."
                android:layout_weight="1"
                android:layout_margin="5dp"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content" />
            <Button
                android:id="@+id/main.num.0"
                android:text="0"
                android:layout_weight="1"
                android:layout_margin="5dp"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content" />
            <Button
                android:id="@+id/main.deng"
                android:text="="
                android:layout_weight="1"
                android:layout_margin="5dp"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content" />
            <Button
                android:id="@+id/main.chu"
                android:text="/"
                android:layout_weight="1"
                android:layout_margin="5dp"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content" />
        </LinearLayout>


    </LinearLayout>

</LinearLayout>
```

### 2.`MainActivity.java`

```java
package site.dbin.application1;

import androidx.appcompat.app.AppCompatActivity;

import android.os.Bundle;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;

import java.util.Objects;

public class MainActivity extends AppCompatActivity {
    // 程序将会将用户输入全部转换为字符串，然后解析为计算式。
    String evalString = "";
    // 定义所有的数字按钮
    static final int ids[] = {
            R.id.main_num_0,
            R.id.main_num_1, R.id.main_num_2,R.id.main_num_3,
            R.id.main_num_4,R.id.main_num_5,R.id.main_num_6,
            R.id.main_num_7,R.id.main_num_8,R.id.main_num_9};
    static final int fuhao[] = {
            R.id.main_jia,R.id.main_jian,R.id.main_cheng,R.id.main_chu,R.id.main_dian
    };
    // 定义所有的符号
    static final char fuhaoc[] = {'+','-','*','/','.'};
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        final EditText editText = findViewById(R.id.main_text);
        editText.setFocusableInTouchMode(false); // 不可以编辑
        editText.setKeyListener(null); //不可以粘贴
        editText.setClickable(false);// 不可以点击
        // 添加数字监听
        for(int i=0;i<=9;i++){
            Button button = findViewById(ids[i]);
            int finalI = i;
            button.setOnClickListener((v)-> {
                evalString = evalString+ finalI;
                editText.setText(evalString);
            });
        }
        // 添加符号监听
        for(int i=0;i<fuhao.length;i++){
            Button button = findViewById(fuhao[i]);
            int finalI = i;
            button.setOnClickListener((v)->{
                evalString = evalString+ fuhaoc[finalI];
                editText.setText(evalString);
            });
        }
        // 添加等于监听
        Button clean = findViewById(R.id.main_clear_text);// 清空按钮
        Button dengyu = findViewById(R.id.main_deng);//等于按钮
        dengyu.setOnClickListener((v)-> {
            Double ans = getResult(evalString);
            if(ans==null){
                Toast.makeText(MainActivity.this,"输入非法",Toast.LENGTH_LONG).show();
            }else{
                // 先显示，后清空算式，不用清空就可以直接输入下一个算式了。
                editText.setText(String.format("%s=%f",this.evalString,ans));
                this.evalString = "";
            }
        });
        // 清空

        clean.setOnClickListener((v)-> {
            this.evalString = "";
            editText.setText(evalString);
        });
    }

    public static Double getResult(String str) {
        boolean isNumber = true;
        try {
            Double.valueOf(str);
        }catch (Exception e){
            isNumber = false;
        }
        try {
            // 递归头
            if (str.isEmpty() || isNumber) {
                return str.isEmpty() ? 0 : Double.parseDouble(str);
            }
            if (str.contains("+")) {
                int index = str.lastIndexOf("+");
                return getResult(str.substring(0, index)) + getResult(str.substring(index + 1));
            }
            if (str.contains("-")) {
                int index = str.lastIndexOf("-");
                return getResult(str.substring(0, index)) - getResult(str.substring(index + 1));
            }
            if (str.contains("*")) {
                int index = str.lastIndexOf("*");
                return getResult(str.substring(0, index)) * getResult(str.substring(index + 1));
            }
            if (str.contains("/")) {
                int index = str.lastIndexOf("/");
                Double d = getResult(str.substring(index + 1));
                if(Objects.equals(d,0.0)){
                    return null;
                }
                return getResult(str.substring(0, index)) / d;
            }
        }catch (Exception e) {
            return null;
        }
        // 出错
        return null;
    }
}
```

