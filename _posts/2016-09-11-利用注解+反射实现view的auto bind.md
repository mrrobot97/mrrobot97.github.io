---
layout: post
author: mrrobot
title: 利用注解+反射实现view的auto bind
---
>相信都用过JakeWharton大神的[ButterKnife](https://github.com/JakeWharton/butterknife)框架，有了ButterKnife我们就可以不用再反复地写繁琐的findViewById()了，ButterKnife内部利用的是注解+编译时生成java字节码实现的，没有利用到反射，故在实际使用时不会对应用的性能产生影响。今天我们就简单的利用注解+反射实现一个简单的view的auto bind，当然只是为了了解一下注解和反射的简单用法。

首先我们定义一个注解接口：

```
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface BindView {
    public int id();
}

```

很简单，只有一个id，因为我们为view绑定控件时只需要一个int类型的id即可。

然后是我们的注解处理类：

```
public class Binder {
    public static void bind(Object object, View view){
        Field[] fields=object.getClass().getDeclaredFields();
        for (Field field:fields){
            if (field.isAnnotationPresent(BindView.class)){
                BindView bindView=field.getAnnotation(BindView.class);
                if (bindView!=null){
                    field.setAccessible(true);
                    try {
                        field.set(object,view.findViewById(bindView.id()));
                    } catch (IllegalAccessException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    }
}
```

我们需要将注解所在的object以及需要绑定到的view作为参数传进去。

然后就可以使用啦！

```
public class MainActivity extends AppCompatActivity {

    @BindView(id=R.id.text_view)
    private TextView mTextView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Binder.bind(this,this.findViewById(android.R.id.content));
        mTextView.setText("This is not a hello world");
    }
}
```

这里我们只需要@BindView(id=...)，然后在setContentView()后调用Binder.bind()即可使用我们的mTextView了。

当然利用反射是会对应用性能产生影响的，实际使用中我们还是应该使用ButterKnife。
