# 前言：

我将实验全都写在一个工程内，所以启动类的名字有所区别，请注意到`AndroidManifest.xm`中修改正确的启动类。

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

### 3.运行结果

![image-20230328182438744](./assets/index/image-20230328182438744.png)

# 实验二

<font color='blue'>提示：我是直接在实验一的基础上做的，运行时将`AndroidMainfest.xml`中的如下代码替换，就可以改变启动时运行的Activity了</font>

```xml
<activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
```

替换为

```xml
<!--MainActivity2是我之后新建的类-->
<activity android:name=".MainActivity2"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>
```

## 实验要求

见实验指导书

## 实验代码和过程

### 1.准备

准备了6张图片，放在了`drawable`目录下

![image-20230328182719726](./assets/index/image-20230328182719726.png)

### 2.布局文件

layout目录下新建布局文件`activity_main2.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <ListView

        android:id="@+id/main2.list"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="horizontal">

    </ListView>
</androidx.constraintlayout.widget.ConstraintLayout>
```

### 3.“组件”

在layout目录下新建`list_array.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <LinearLayout
        android:id="@+id/list_item"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="horizontal">
        <ImageView
            android:id="@+id/header"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:adjustViewBounds="true"
            android:maxHeight="80dp"
            android:maxWidth="80dp"
            android:padding="10dp"/>
        <TextView
            android:id="@+id/name"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textSize="20dp"
            android:padding="10dp"/>
    </LinearLayout>
</LinearLayout>
```

### 4.`MainActivity2.java`

```java
package site.dbin.application1;

import android.os.Bundle;
import android.widget.ArrayAdapter;
import android.widget.ListView;
import android.widget.SimpleAdapter;
import android.widget.TextView;

import androidx.annotation.Nullable;
import androidx.appcompat.app.AlertDialog;
import androidx.appcompat.app.AppCompatActivity;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class MainActivity2 extends AppCompatActivity {
    // 图片的信息
    private String[] data = {
            "Apple","Banana","Orange","Watermelon","Pear"
    };
    // 图片的id
    private int[] header = {
            R.drawable.img0,R.drawable.img1,R.drawable.img2,R.drawable.img3,
            R.drawable.img4,//R.drawable.img5,R.drawable.img6
    };
    // AlerDialog
    private AlertDialog alert = null;
    private AlertDialog.Builder builder = null;
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        this.setContentView(R.layout.activity_main2);
//      实验指导书让写的，没什么用
//        ArrayAdapter<String> adapter = new ArrayAdapter<>(
//                MainActivity2.this,R.layout.list_array,data);
        ListView listView = findViewById(R.id.main2_list);
        // 添加元素点击的监听器
        listView.setOnItemClickListener((adapterView, view, i, l) -> {
            // 获取子元素的name,最终对应的就是前面定义的data变量的数据
            TextView textView = view.findViewById(R.id.name);
            alert = null;
            builder = new AlertDialog.Builder(MainActivity2.this);
            alert = builder
                    .setTitle("提示：")
                    .setMessage("你点击了"+textView.getText())
                    .setPositiveButton("确定", (dialog, which) -> {})
                    .create();
            alert.show();
        });
        //listView.setAdapter(adapter);
        // 存放数据的list
        List<Map<String,Object>> list = new ArrayList<>();
        for(int i=0;i<data.length;i++){
            Map<String,Object> map = new HashMap<>();
            map.put("header",header[i]);
            map.put("data",data[i]);
            list.add(map);
        }
        // 创建一个适配器，适配器的数据是list,布局是list_array.xml
        // 子元素中数据是header对应header,name对应data.
        SimpleAdapter simpleAdapter = new SimpleAdapter(
                MainActivity2.this,
                list,
                R.layout.list_array,
                new String[]{"header","data"},
                new int[]{R.id.header,R.id.name}
                );
        listView.setAdapter(simpleAdapter);
    }
}
```

### 5.运行效果

![image-20230328184133050](./assets/index/image-20230328184133050.png)

![image-20230328184209684](./assets/index/image-20230328184209684.png)

# 实验三

## 实验要求：

设计如下布局

![image-20230401125717543](./assets/index/image-20230401125717543.png)

在MainActivity中通过SharedPreferences保存验证通过的用户名和密码。

当用户在“记住用户名和密码”处打钩，则在下一次进入该界面时，自动读取SharedPreferences中的内容，并填充到指定的文本框中。

在Module中新建一个Activity，取名为“DownloadFile”。

设计DownloadFile的布局文件，主要包括两个文本框（et1和et2）和一个按钮（btn1）。

在DownloadFile中编写相关的逻辑代码，点击按钮btn1后，通过HttpURLConnection实现将et1中指定网络地址的文件进行下载，并保存到本地，保存的文件名来自et2中的输入。

下载完成后，使用Toast给出提示。

通过Device File Explorer检查文件是否生成，能否正确打开。

## 实验代码(第一部分)

### `activity_main3.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity3">
<LinearLayout
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <LinearLayout
        android:orientation="horizontal"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="用户名:"
            android:textSize="30dp"
            android:width="50pt"/>
        <EditText
            android:id="@+id/main3.username"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text=""
            android:textSize="30dp"/>
    </LinearLayout>
    <LinearLayout
        android:orientation="horizontal"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="密码:"
            android:textSize="30dp"
            android:width="50pt"/>
        <EditText
            android:id="@+id/main3.password"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text=""
            android:inputType="textPassword"
            android:textSize="30dp"
            android:width="100pt"/>
        <ToggleButton
            android:id="@+id/main3.show"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:width="50pt"
            android:textAllCaps="false"
            android:textOn="hide"
            android:textOff="show"
            />
    </LinearLayout>
    <LinearLayout
        android:orientation="vertical"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
        <RadioButton
            android:id="@+id/main3.remember"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="记住用户名和密码"
            android:textSize="20dp"/>
        <Button
            android:id="@+id/main3.submit"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="登录"
            android:textSize="30dp" />
    </LinearLayout>
</LinearLayout>
</androidx.constraintlayout.widget.ConstraintLayout>
```

### `MainActivity3.java`

```java
package site.dbin.application1;

import androidx.appcompat.app.AppCompatActivity;

import android.content.SharedPreferences;
import android.os.Bundle;
import android.text.InputType;
import android.widget.Button;
import android.widget.EditText;
import android.widget.RadioButton;
import android.widget.Toast;
import android.widget.ToggleButton;

public class MainActivity3 extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main3);
        SharedPreferences sp = getSharedPreferences("info",MODE_PRIVATE);
        SharedPreferences.Editor editor = sp.edit();
        EditText usernameT = findViewById(R.id.main3_username);
        EditText passwordT = findViewById(R.id.main3_password);
        ToggleButton show = findViewById(R.id.main3_show);// 显示密码的控件
        show.setOnCheckedChangeListener((v,is)->{
            if(is){
                passwordT.setInputType(InputType.TYPE_CLASS_TEXT | InputType.TYPE_TEXT_VARIATION_VISIBLE_PASSWORD);
            } else {
                passwordT.setInputType(InputType.TYPE_CLASS_TEXT | InputType.TYPE_TEXT_VARIATION_PASSWORD);
            }
        });
        String username = sp.getString("username",null);
        String password = sp.getString("password",null);
        if(username!=null){
            usernameT.setText(username);
        }
        if(password!=null){
            passwordT.setText(password);
        }
        Button submit = findViewById(R.id.main3_submit);
        RadioButton remember = findViewById(R.id.main3_remember);
        submit.setOnClickListener(v -> {
            String u = usernameT.getText().toString();
            String p = passwordT.getText().toString();
            if(isBlank(u)||isBlank(p)){
                Toast.makeText(MainActivity3.this,"用户名和密码不能为空", Toast.LENGTH_SHORT).show();
                return;
            }
            if(remember.isChecked()) {
                editor.putString("username", u);
                editor.putString("password", p);
                editor.commit();// 提交数据
            }
            Toast.makeText(MainActivity3.this,"成功", Toast.LENGTH_SHORT).show();
        });
    }

    private static boolean isBlank(String s){
        if(s==null)return true;
        boolean is = true;
        for(int i=0;i<s.length();i++){
            if(s.charAt(i)!=' '){
                is = false;
                break;
            }
        }
        return is;
    }
}
```

## 运行效果(第一部分)

![image-20230401130753245](./assets/index/image-20230401130753245.png)

## 实验代码(第二部分)

### 修改AndroidManifest.xml

在二级标签中添加下面类容，申请网络权限

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

### `activity_download_file.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".DownloadFile">
    <LinearLayout
        android:orientation="vertical"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        <EditText
            android:id="@+id/et1"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="请输入下载地址"/>
        <EditText
            android:id="@+id/et2"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="请输入文件名"/>
        <Button
            android:id="@+id/btn1"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="下载"/>
    </LinearLayout>
</androidx.constraintlayout.widget.ConstraintLayout>
```

### `DownloadFile.java`

```java
package site.dbin.application1;

import static androidx.constraintlayout.helper.widget.MotionEffect.TAG;

import androidx.appcompat.app.AppCompatActivity;

import android.os.Bundle;
import android.os.FileUtils;
import android.util.Log;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;

import java.io.File;
import java.io.FileOutputStream;
import java.io.InputStream;
import java.net.HttpURLConnection;
import java.net.URL;

public class DownloadFile extends AppCompatActivity {
    private EditText et1, et2;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_download_file);
        et1 = findViewById(R.id.et1);
        et2 = findViewById(R.id.et2);
        Button btn1 = findViewById(R.id.btn1);
        btn1.setOnClickListener((v)->{
            String url = et1.getText().toString();
            // 测试连接
            if(isBlank(url))url = "https://raw.githubusercontent.com/dbinfun/My-college-assignments/main/README.md";
            String fileName = et2.getText().toString();
            if(isBlank(url)||isBlank(fileName)){
                Toast.makeText(this,"连接和文件名不能为空",Toast.LENGTH_SHORT).show();
                return;
            }
            String pd = this.getFilesDir().getAbsolutePath();
            Log.d(TAG,pd);// 输出路径，方便查找
            downLoad(url,pd+"/"+fileName);
        });
    }
    public static void downLoad(final String path, final String FileName) {
        //开启新的线程进行文件下载
        new Thread(() -> {
            try {
                URL url = new URL(path);
                HttpURLConnection con = (HttpURLConnection) url.openConnection();
                con.setReadTimeout(5000);
                con.setConnectTimeout(5000);
                con.setRequestProperty("Charset", "UTF-8");
                con.setRequestMethod("GET");
                InputStream is = con.getInputStream();//获取输入流
                FileOutputStream fileOutputStream = null;//文件输出流
                if (is != null) {
                    File file = new File(FileName);
                    boolean isCreate = file.createNewFile();
                    if(!isCreate){
                        is.close();
                        Log.e(TAG, "downLoad: 文件访问错误");
                        return;
                    }

                    fileOutputStream = new FileOutputStream(file);//指定文件保存路径，
                    byte[] buf = new byte[1024];
                    int ch;
                    while ((ch = is.read(buf)) != -1) {
                        fileOutputStream.write(buf, 0, ch);//将获取到的流写入文件中
                    }
                }
                if (fileOutputStream != null) {
                    fileOutputStream.flush();
                    fileOutputStream.close();
                    is.close();
                }

            } catch (Exception e) {
                e.printStackTrace();
            }
        }).start();//新线程的启动
    }
    private static boolean isBlank(String s){
        if(s==null)return true;
        boolean is = true;
        for(int i=0;i<s.length();i++){
            if(s.charAt(i)!=' '){
                is = false;
                break;
            }
        }
        return is;
    }
}
```

## 运行效果(第二部分)

![image-20230401131039719](./assets/index/image-20230401131039719.png)

到`/data/data/你的程序包名+的工程名/file/`寻找下载的文件(有时需要点击Synchronize刷新)

![image-20230401131157826](./assets/index/image-20230401131157826.png)

![image-20230401131244677](./assets/index/image-20230401131244677.png)
