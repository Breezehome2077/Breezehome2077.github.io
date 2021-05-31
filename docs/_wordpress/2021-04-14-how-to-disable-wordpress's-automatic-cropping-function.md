---
title:          技巧：禁止 WordPress 的图片自动裁剪功能
date:           2021-04-14 05:26 +0800
description:    测试版本 WP 5.7.2 测试有效，复制代码到主题的 functions.php 文件即可。
order:          2102
---

<p class="post-body-mark">
    20210414 补充1：完整代码见文章末尾。
</p>
<p class="post-body-mark">
    20210414 补充2：代码中的crop项表示的是自动裁剪的方式：“0”表示按比例进行裁剪；“1”表示直接裁剪；
默认的裁剪中心为左下角，你可以通过将crop项设置为 <b>array( 'left', 'top' )</b> 的方式来进行调整。
更多介绍，请参考<a href="https://developer.wordpress.org/reference/functions/add_image_size/">官网对add_image_size()函数的说明</a>。
</p>

禁止是不能禁止的，只能说：如何不触发 WordPress 的图片自动裁剪操作。

首先，我们要明确 WordPress 对图片进行自动裁剪的原因是：上传的图片尺寸与 WordPress 内注册的图片尺寸不同; 
当上传图片尺寸大于注册的图片时， WordPress 就会对上传图片进行裁剪，生成符合注册尺寸要求的副本图片。 

因此，我们搜索的关键词就是 “WordPress abc 图片尺寸”，“abc” 可以是“删除”，可以是“移除”，也可以是其他同义词。

然后，我们就可以通过搜索引擎搜索关键词了，只要多搜搜就可以查到删除注册尺寸的方式有两种，分别是 `unset()` 和 `remove_image_size()`。

示例1：

{% highlight php %}
// unset()
function paulund_remove_default_image_sizes( $sizes) {
    unset( $sizes['thumbnail']);
    return $sizes;
}
add_filter('intermediate_image_sizes_advanced', 'paulund_remove_default_image_sizes');
{% endhighlight %}

示例2：

{% highlight php %}
// remove_image_size()
add_filter( 'init', 'wzm_disable_auto_crop_img' );
function wzm_disable_auto_crop_img() {
    remove_image_size( 'small'); // 150px
}
{% endhighlight %}

接下来，我们要检测代码是否管用，该步骤通过 WordPress 插件 <b>Regenerate thumbnail</b> 来实现。

进入后台，安装并启用插件 “Regenerate thumbnail”；在插件的管理页面可以看到当前 WordPress 内注册图片尺寸有哪些了。

然后将前面提到的两段代码复制到主题的 functions.php 文件后，我们可以看到：

1. “thumbnail” 的注册大小依旧和“设置-媒体”中的图片尺寸大小相同（这里我设置的是 996）;
2. “2048x2048” 的注册大小还是 2048x2048;
3. 已注册的图片尺寸 "small" 不见了。

所以，我们要使用的代码是 `remove_image_size()`；但由于注册的尺寸其他插件可能会用到，
例如：Woocommerce 的画廊功能就要用到 <b>woocommerce_gallery_thumbnail</b>；
因此我们就用 `add_image_size()` 来注册一个具有相同名称，但长宽为“0x0”的新的图片尺寸
（尺寸设置为0x0，就可以保证上传图片百分百大于注册的图片尺寸了）。

{% highlight php %}
add_filter( 'init', 'wzm_disable_auto_crop_img' );
function wzm_disable_auto_crop_img() {
    remove_image_size( 'small'); // 150px
    add_image_size( 'small', 0, 0, false );     // false 表示：按比例大小裁剪；true 表示：直接裁剪。
}
{% endhighlight %}

这样一通操作下来之后，大部分注册尺寸就搞定了，但还是有几个需要额外设置，他们分别是：
“thumbnail”（缩略图大小/原图）、“meidum”（中等尺寸/300px）、"medium-large"（隐藏起来的768px）和 "large"（大尺寸/1024px）。
我将它们称为上传尺寸（因为之后查到这四个要是要用函数 `update_option()` 来修改的）。

上传尺寸的修改方式为：“后台-设置-媒体”，在这个界面可以搞定“thumbnail”、“meidum” 和 “large”；
至于隐藏起来的 “medium-large” 则需要将地址栏中的 “wp-admin/options-media.php” 修改为 “wp-admin/options.php”，
然后回车打开设置页面，接下来按快捷键 <kbd>CTRL+F</kbd> 搜索 “medium_large_size_w”，找到后将数值“768”改为“0”，
最后点击页面底部的保存按钮，进行保存即可。

如果客户对这方面没有需要的话，可以使用函数 update_option() 进行设置，但需要注意的是该设置会覆盖掉上面的操作，也就是说客户不能自己在后台进行更改了。

{% highlight php %}
// 从 “wp-admin/options.php” 中可以看到长、宽、自动裁剪是分开进行设置的。
update_option( 'thumbnail_size_w', 0 );
update_option( 'thumbnail_size_h', 0 );
update_option( 'thumbnail_crop', 0 );   // 自动裁剪方式：“0”表示按比例进行裁剪；“1”表示直接裁剪。
{% endhighlight %}

至此，所有的注册尺寸我们就都搞定了。

最后，附一下完整的处理代码，有需要的直接将其复制到主题的 functions.php 文件中即可。

{% highlight php %}
//  禁用图片自动裁剪
//  添加完代码后，新上传的图片会接受此设置，
//  但之前上传的图片，需要手动删除或者使用插件 Regenerate thumbnail 进行重新生成。
add_filter( 'init', 'wzm_disable_auto_crop_img' );
function wzm_disable_auto_crop_img() {
    remove_image_size( 'small'); // 150px
    add_image_size( 'small', 0, 0, false );
    remove_image_size( '1536x1536'); // 1536px
    add_image_size( '1536x1536', 0, 0, false );
    remove_image_size( '2048x2048'); // 2048px
    add_image_size( '2048x2048', 0, 0, false );
    remove_image_size( 'woocommerce_thumbnail'); // 300px
    add_image_size( 'woocommerce_thumbnail', 0, 0, false );
    remove_image_size( 'woocommerce_single'); // 600px
    add_image_size( 'woocommerce_single', 0, 0, false );
    remove_image_size( 'woocommerce_gallery_thumbnail'); // 100px
    add_image_size( 'woocommerce_gallery_thumbnail', 0, 0, false );
    remove_image_size( 'shop_catalog'); // 300px
    add_image_size( 'shop_catalog', 0, 0, false );
    remove_image_size( 'shop_single'); // 600px
    add_image_size( 'shop_single', 0, 0, false );
    remove_image_size( 'shop_thumbnail'); // 100px
    add_image_size( 'shop_thumbnail', 0, 0, false );
}
// 注意：update_option()函数设置的值会覆盖掉“后台-设置-媒体”中的值
update_option( 'thumbnail_size_w', 0 );
update_option( 'thumbnail_size_h', 0 );
update_option( 'thumbnail_crop', 0 );   // 自动裁剪方式：“0”表示按比例进行裁剪；“1”表示直接裁剪。
update_option( 'medium_size_w', 0 );
update_option( 'medium_size_h', 0 );
update_option( 'medium_large_size_w', 0 );  // 隐藏起来的 “768px”。
update_option( 'large_size_w', 0 );
update_option( 'large_size_h', 0 );
{% endhighlight %}