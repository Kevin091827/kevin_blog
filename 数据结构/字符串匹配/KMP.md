

# 1.字符串匹配问题

**何为字符串匹配问题？**


字符串匹配问题则是，给出一个模式串和一个文本串，需要找出模式串是文本串的子串且模式串在文本串开始的位置

# 2.暴力解法

```java
    /**
     * 暴力解法
     * O(MN)
     * @param txt
     * @param pat
     * @return
     */
    public static int forceSearch(String txt,String pat){
        int txtLen = txt.length();
        int patLen = pat.length();
        int i = 0;
        int j = 0;
        while(i < txtLen && j < patLen){
            //逐一比对
            if(txt.charAt(i) == pat.charAt(j)){
                //比对成功，继续向后
                i++;
                j++;
            }else{
                //比对失败，j回到模式串开始，i回到上一次比对位置的下一个位置
                j = 0;
                i = i - j + 1;
            }
        }
        if(j == patLen){
            return i - j;
        }else{
            return -1;
        }
    }
```
# 3.kmp模式匹配算法

## 求next数组

### 1.模式串标上序号

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190809140932929.png)

### 2.把模式串的所有子串列举出来

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190809141034537.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

### 3.把模式串中相等的前缀和后缀的长度求出来

前缀：除了最后一个字符外，一个字符串所有头部字符的集合

后缀：除了第一个字符外，一个字符串所有的尾部字符的集合

例如：最大相等前缀和后缀长度为2

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190809141406764.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)


![在这里插入图片描述](https://img-blog.csdnimg.cn/20190809141627830.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

这样我们求出了一个相等最长前缀和后缀的长度集合maxl

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190809141826210.png)

next数组就是相当于最大长度集合集体向右移动一位，数组第一位赋值为-1

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190809142004725.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

### 4.代码实现

```java
    /**
     * 寻找前缀后缀最长公共元素长度
     * 获取next数组，告知下一次应该跳到模式串的哪一个位置
     * @param patStr
     * @param next
     */
    public static void next(String patStr,int[] next){
        int length = patStr.length();
        next[0] = -1;
        int k = -1;
        int j = 0;
        while(j < length - 1){
            if(k == -1 || patStr.charAt(k) == patStr.charAt(j)){
                ++k;
                ++j;
                next[j] = k;
            }else{
                k = next[k];
            }
        }
        for(int i = 0;i<next.length;i++) {
            System.out.println("---->" +next[i]);
        }
    }
```


## 模式匹配

接下来有了next数组就可以进行模式匹配了

### 1.模式串和文本串从第一个字符开始比较，相等则，指针移到模式串和文本串的下一个字符

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190809142555907.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

### 2.当不匹配时，将字串中索引为0的元素移到不匹配元素的位置，其他元素移动相同的距离

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190809142909709.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190809143015773.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

### 3.当出现这个前面连续几个比对成功，突然出现一个匹配失败时，则将模式串移到以匹配不成功元素对应next数组的元素为索引所在的位置，因为前面的相等前缀，无需重复比较

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190809143131289.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190809143231315.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

### 4.直到匹配成功

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190809143705948.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

### 5.完整代码实现
```java
    /**
     * KMP
     * O(m+n) m:字符串长度，n：next数组长度
     * @param txt
     * @param pat
     * @param next
     * @return
     */
    public static int kmpSearch(String txt,String pat,int[] next){
        int txtLen = txt.length();
        int patLen = pat.length();
        int i = 0;
        int j = 0;
        while(i < txtLen && j < patLen){
            if(j == -1 || txt.charAt(i) == pat.charAt(j)){
                j++;
                i++;
            }else{
                j = next[j];
            }
        }
        if(j == patLen){
            return i - j;
        }else {
            return -1;
        }
    }



    /**
     * 寻找前缀后缀最长公共元素长度
     * 获取next数组，告知下一次应该跳到模式串的哪一个位置
     * @param patStr
     * @param next
     */
    public static void next(String patStr,int[] next){
        int length = patStr.length();
        next[0] = -1;
        int k = -1;
        int j = 0;
        while(j < length - 1){
            if(k == -1 || patStr.charAt(k) == patStr.charAt(j)){
                ++k;
                ++j;
                next[j] = k;
            }else{
                k = next[k];
            }
        }
        for(int i = 0;i<next.length;i++) {
            System.out.println("---->" +next[i]);
        }
    }
```