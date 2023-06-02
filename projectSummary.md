## 1.图片上传

背景：图片还没上传完成，用户就点了提交信息按钮，导致发送了提交信息接口，以至于后续后端无法返回正确的图片列表。

解决方法：在每次上传图片的时候，需要给用户增加一个遮罩层，并提示用户图片正在上传，防止用户在图片没上传完成之前去提交信息。
除了上述这种方法外，我们还可以去给一个时间去限定图片上传最长的时间，如果超过这个时间，就提示用户图片上传失败，让用户重新上传，提高代码的健壮性。

感悟：在项目开发中，许多场景我们都需要去做兜底，针对一些出现次数比较少的情况做一些处理，比如：在发送网络请求的时候，可能因为未知的原因，导致发送请求失败，或者响应失败，这种情况下我们最好给用户以提示。

## 2.输入框数据格式

背景：接口需要我们传给后端小数点后两位的数字的参数，在提交信息发送提交接口的时候去截取数字长度，不合理，因为用户看不到这种数据截取的现象，可能造成纠纷。

解决方法：在用户输入的时候就给用户做限制，例如当输入框内容改变的时候，通过正则的方式去对输入内容进行处理。

```javascript
const numberFixedDigit = (e: any) => {
  e.target.value = e.target.value.replace(/[^\d.]/g, ""); //清除“数字”和“.”以外的字符
  e.target.value = e.target.value.replace(/\.{2,}/g, "."); //只保留第一个. 清除多余的
  e.target.value = e.target.value
    .replace(".", "$#$")
    .replace(/\./g, "")
    .replace("$#$", ".");
  e.target.value = e.target.value.replace(/^(\\-)*(\d+)\.(\d\d).*$/, "$1$2.$3"); //只能输入两个小数
  e.target.value = e.target.value.replace(/^\./g, ""); //首位不能输入“.”
  if (e.target.value.indexOf(".") < 0 && e.target.value != "") {
    //如果没有小数点，首位不能为0，如01、02...
    e.target.value = parseFloat(e.target.value);
  }
  nextTick(() => {
    input.value = e.target.value;
  });
};
```

感悟：在对用户行为进行限制的时候，尽可能把这个限制的过程提前，例如：限制用户输入内容的长度，我们可以在输入框下方加一个字数提醒的一个文本（1/200），并且直接限制用户输入内容的长度，当用户输入内容达到 200 时，禁止用户继续输入，而不是让用户随意输入内容长度，在用户提交信息的时候再去提醒用户你输入的内容长度过长。

## 3.微信小程序 font-weight 属性问题

背景：根据设计师给的图纸字体设置 font-weight:600，安卓机上字体不加粗，ios 上字体正常加粗。

解决方法：统一使用 font-weight:bold，不过这个需要跟设计师约定一下，即可解决这个问题。

## 4.ios 下 position:fixed 固定定位失效问题

#### 背景

微信小程序中使用 fixed 固定定位封装自定义导航栏 navbar，也算是比较常见的需求了，一般都是用 position:fixed 去做一个固定定位。

#### 实现思路

    .navbar{
        position:fixed;
    }

如果没设置 top，bottom，left，right 等属性，那么会相对于原先位置进行固定，在绝大部分机型都是可以实现，没有问题。

#### 问题

机型：iphone13pro\
系统版本：ios16\
出现问题：导航栏被固定到屏幕中间

#### 解决

加上 top 属性即可解决，代码实现如下

    .navbar{
        position:fixed;
        top:0;
    }

> 记录一次兼容性问题，不过这样的兼容性问题在开发过程中我们完全可以避免，这其实就是代码质量导致的，在写一些元素定位属性的时候，我们要把位置信息写上（top，bottom，left，right）固定元素位置，这样做可以提高代码可读性，也可以避免一些奇怪的 bug

## 5.微信小程序 text 标签有上边距问题

背景：UI 还原发现没有给 text 设置上边距，却产生了上边距，与设计图有差距

代码

```javascript
              <view class="time_wrap">
                <text class="active_time"
                  >最近活跃:
                  {{
                    item.lastLoginTime
                  }}</text>
                <text class="join_time"
                  >加入时间:
                  {{
                    item.joinInTime
                  }}</text>
              </view>
```

原因:行内元素多行书写会产生空白字符，导致有上边距的产生

解决方法：
方法一：

```javascript
              <view class="time_wrap">
                <text class="active_time">最近活跃:{{item.lastLoginTime}}</text>
                <text class="join_time">加入时间:{{item.joinInTime}}</text>
              </view>
```

把行内元素的内容都写在同一行，不过这种方法局限性很大，在项目开发中，我们一般在项目中都配置了特定的格式化方法，比如：超过多少字符就换行。

方法二：

```javascript
              <view class="time_wrap">
                <text class="active_time"
                  >最近活跃:
                  {{
                    item.lastLoginTime
                  }}</text>
                <text class="join_time"
                  >加入时间:
                  {{
                    item.joinInTime
                  }}</text>
              </view>
              .time_wrap{
                display:flex;
              }
```

给父元素加上 display:flex，原理其实就是把子元素变成块级元素，这样子就没有行内元素那个特性了。

## 6.微信小程序图片定宽不定高

背景：一般我们使用 css 属性 object-fit 这个属性来改变图片的伸缩方式，试了所有属性，在微信小程序中都没有生效。

原因：微信小程序中不支持 object-fit 这个属性。

解决：使用微信小程序中内置的 image 标签中的 mode 属性，mode="widthFix"即可设置图片为定宽不定高。
