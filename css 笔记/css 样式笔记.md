css 样式笔记

# 1.css 样式选择器

```css
*{ //表示通配符，匹配所有元素
    margin: 0, 外间距
    padding：0 内间距
  
}
用于去掉浏览器默认自带的边距

p.mycol{ //交集选择器，p表示 p标签 .mycol 表示样式名
   backgroud-color:red;
}

p#mycol{ //交集选择器，p表示 p标签 #mycol 表示id为mycol的
   backgroud-color:red;
}

<p class='mycol'></p> //应用上样式
<div class='mycol'></div> //没有应用上



```



```css
/*哪个元素在前面，就要将样式写在上面*/
ol li {
    list-style: none; /*去掉默认的小黑点*/
}

/*后代选择器 层次，可以不写完整，可以跳过一些元素，只要能找到即可，一般我们只写3层及以下 */
ol li strong {
    background-color: blueviolet;
}

<ol>
    <li>
        <strong>我是杨新强</strong>
    </li>
</ol>
```



























