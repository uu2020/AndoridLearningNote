

<!DOCTYPE html>
<!--[if IEMobile 7 ]><html class="no-js iem7"><![endif]-->
<!--[if lt IE 9]><html class="no-js lte-ie8"><![endif]-->
<!--[if (gt IE 8)|(gt IEMobile 7)|!(IEMobile)|!(IE)]><!--><html class="no-js" lang="en"><!--<![endif]-->
<head>
  <meta charset="utf-8">
  <title>Android性能优化之运算篇 - 胡凯</title>
  <meta name="author" content="HuKai">

  
  <meta name="description" content="Google近期在Udacity上发布了Android性能优化的在线课程，分别从渲染，运算与内存，电量几个方面介绍了如何去优化性能，这些课程是Google之前在Youtube上发布的Android性能优化典范专题课程的细化与补充。 下面是运算篇章的学习笔记，部分内容与前面的性能优化典范有重合， &hellip;">
  

  <!-- http://t.co/dKP3o1e -->
  <meta name="HandheldFriendly" content="True">
  <meta name="MobileOptimized" content="320">
  <meta name="viewport" content="width=device-width, initial-scale=1">

  
  <link rel="canonical" href="http://hukai.me/android-performance-compute/">
  <link href="/favicon.png" rel="icon">
  <link href="/stylesheets/screen.css" media="screen, projection" rel="stylesheet" type="text/css">

  <!--
  <script src="/javascripts/modernizr-2.0.js"></script>
  <script src="/javascripts/ender.js"></script>
  <script src="/javascripts/octopress.js" type="text/javascript"></script>
  -->
  <link href="/atom.xml" rel="alternate" title="胡凯" type="application/atom+xml">
  <!--Fonts from Google"s Web font directory at http://google.com/webfonts -->
<!--Mark by @Kesen-->
<!-- <link href="http://fonts.googleapis.com/css?family=PT+Serif:regular,italic,bold,bolditalic" rel="stylesheet" type="text/css">
<link href="http://fonts.googleapis.com/css?family=PT+Sans:regular,italic,bold,bolditalic" rel="stylesheet" type="text/css"> -->

  
  <script>
    (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
    (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
    m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
    })(window,document,'script','//www.google-analytics.com/analytics.js','ga');

    ga('create', 'UA-37679268-1', 'auto');
    ga('send', 'pageview');

  </script>



</head>

<body   class="no-sidebar"  >
  <header role="banner"><hgroup>
  <h1><a href="/">胡凯</a></h1>
  
</hgroup>

</header>
  <nav role="navigation"><ul class="subscription" data-subscription="rss">
  <li><a href="/atom.xml" rel="subscribe-rss" title="subscribe via RSS">RSS</a></li>
  
</ul>
  
<form action="http://google.com/search" method="get">
  <fieldset role="search">
    <input type="hidden" name="q" value="site:hukai.me" />
    <input class="search" type="text" name="q" results="0" placeholder="Search"/>
  </fieldset>
</form>
  
<ul class="main-navigation">
  <li><a href="/">Blog</a></li>
  <li><a href="/blog/archives">Archives</a></li>
  <li><a href="/about">About</a></li>
  <li><a href="/android-training-course-in-chinese/index.html">Android Training in Chinese</a></li>
</ul>

</nav>
  <div id="main">
    <div id="content">
      <div>
<article class="hentry" role="article">
  
  <header>
    
      <h1 class="entry-title">Android性能优化之运算篇</h1>
    
    
      <p class="meta">
        








  


<time datetime="2015-04-12T13:50:00+08:00" pubdate data-updated="true">Apr 12<span>th</span>, 2015</time>
        
         | <a href="#disqus_thread">Comments</a>
        
      </p>
    
  </header>


<div class="entry-content"><p><img src="/images/android_performance_course_udacity.jpg" alt="" /></p>

<p>Google近期在Udacity上发布了<a href="https://www.udacity.com/course/ud825">Android性能优化的在线课程</a>，分别从渲染，运算与内存，电量几个方面介绍了如何去优化性能，这些课程是Google之前在Youtube上发布的<a href="http://hukai.me/android-performance-patterns/">Android性能优化典范</a>专题课程的细化与补充。</p>

<p>下面是运算篇章的学习笔记，部分内容与前面的性能优化典范有重合，欢迎大家一起学习交流！</p>

<h3>1)Intro to Compute and Memory Problems</h3>

<p>Android中的Java代码会需要经过编译优化再执行的过程。代码的不同写法会影响到Java编译器的优化效率。例如for循环的不同写法就会对编译器优化这段代码产生不同的效率，当程序中包含大量这种可优化的代码的时候，运算性能就会出现问题。想要知道如何优化代码的运算性能就需要知道代码在硬件层的执行差异。</p>

<h3>2)Slow Function Performance</h3>

<p>如果你写了一段代码，它的执行效率比想象中的要差很多。我们需要知道有哪些因素有可能影响到这段代码的执行效率。例如：比较两个float数值大小的执行时间是int数值的4倍左右。这是因为CPU的运算架构导致的，如下图所示：</p>

<p><img src="/images/android_perf_compute_float_int.png" alt="" /></p>

<p>虽然现代的CPU架构得到了很大的提升，也许并不存在上面所示的那么大的差异，但是这个例子说明了代码写法上的差异会对运算性能产生很大的影响。</p>

<!-- More -->


<p>通常来说有两类运行效率差的情况：第1种是相对执行时间长的方法，我们可以很轻松的找到这些方法并做一定的优化。第2种是执行时间短，但是执行频次很高的方法，因为执行次数多，累积效应下就会对性能产生很大的影响。</p>

<p>修复这些细节效率问题，需要使用Android SDK提供的工具，进行仔细的测量，然后再进行微调修复。</p>

<h3>3)Traceview Walkthrough</h3>

<p>通过Android Studio打开里面的Android Device Monitor，切换到DDMS窗口，点击左边栏上面想要跟踪的进程，再点击上面的Start Method Tracing的按钮，如下图所示：</p>

<p><img src="/images/android_perf_compute_traceview.png" alt="" /></p>

<p>启动跟踪之后，再操控app，做一些你想要跟踪的事件，例如滑动listview，点击某些视图进入另外一个页面等等。操作完之后，回到Android Device Monitor，再次点击Method Tracing的按钮停止跟踪。此时工具会为刚才的操作生成TraceView的详细视图。</p>

<p><img src="/images/android_perf_compute_traceview_2.png" alt="" /></p>

<p>关于TraceView中详细数据如何查看，这里不展开了，有很多文章介绍过。</p>

<h3>4)Batching and Caching</h3>

<p>为了提升运算性能，这里介绍2个非常重要的技术，Batching与Caching。</p>

<p><strong>Batching</strong>是在真正执行运算操作之前对数据进行批量预处理，例如你需要有这样一个方法，它的作用是查找某个值是否存在与于一堆数据中。假设一个前提，我们会先对数据做排序，然后使用二分查找法来判断值是否存在。我们先看第一种情况，下图中存在着多次重复的排序操作。</p>

<p><img src="/images/android_perf_compute_batching_1.png" alt="" /></p>

<p>在上面的那种写法下，如果数据的量级并不大的话，应该还可以接受，可是如果数据集非常大，就会有严重的效率问题。那么我们看下改进的写法，把排序的操作打包绑定只执行一次：</p>

<p><img src="/images/android_perf_compute_batching_2.png" alt="" /></p>

<p>上面就是Batching的一种示例：把重复的操作拎出来，打包只执行一次。</p>

<p><strong>Caching</strong>的理念很容易理解，在很多方面都有体现，下面举一个for循环的例子：</p>

<p><img src="/images/android_perf_compute_caching.png" alt="" /></p>

<p>上面这2种基础技巧非常实用，积极恰当的使用能够显著提升运算性能。</p>

<h3>5)Blocking the UI Thread</h3>

<p>提升代码的运算效率是改善性能的一方面，让代码执行在哪个线程也同样很重要。我们都知道Android的Main Thread也是UI Thread，它需要承担用户的触摸事件的反馈，界面视图的渲染等操作。这就意味着，我们不能在Main Thread里面做任何非轻量级的操作，类似I/O操作会花费大量时间，这很有可能会导致界面渲染发生丢帧的现象，甚至有可能导致ANR。防止这些问题的解决办法就是把那些可能有性能问题的代码移到非UI线程进行操作。</p>

<h3>6)Container Performance</h3>

<p>另外一个我们需要注意的运算性能问题是基础算法的合理选择，例如冒泡排序与快速排序的性能差异：</p>

<p><img src="/images/android_perf_compute_container.png" alt="" /></p>

<p>避免我们重复造轮子，Java提供了很多现成的容器，例如Vector，ArrayList，LinkedList，HashMap等等，在Android里面还有新增加的SparseArray等，我们需要了解这些基础容器的性能差异以及适用场景。这样才能够选择合适的容器，达到最佳的性能。</p>

<p><img src="/images/android_perf_compute_container_2.png" alt="" /></p>

<p><strong>Notes:</strong>关于更多代码优化的小技巧，请点击<a href="http://hukai.me/android-training-performance-tips/">这里</a></p>
</div>


  <footer>
    <p><img src="http://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" alt="" title="" type="image/png"><br>
    <a href="http://creativecommons.org/licenses/by-nc-sa/4.0/" target="_blank">知识共享许可协议</a>：本站作品由<a href="http://hukai.me" target="_blank">HuKai</a>创作，采用<a href="http://creativecommons.org/licenses/by-nc-sa/4.0/" target="_blank">知识共享 署名-非商业性使用-相同方式共享 4.0 国际 许可协议</a>进行许可。
    <br>
    如果你觉得这篇文章对你有帮助，请点击下面的分享链接，你还可以选择扫描二维码进行打赏，承诺所有打赏都会用于公益!
    <img src="/images/pay2me.jpg" alt=""/>
    <p class="meta">
      
  

<span class="byline author vcard">Posted by <span class="fn">HuKai</span></span>

      








  


<time datetime="2015-04-12T13:50:00+08:00" pubdate data-updated="true">Apr 12<span>th</span>, 2015</time>
      

<span class="categories">
  
    <a class='category' href='/blog/categories/android/'>Android</a>, <a class='category' href='/blog/categories/android-performance/'>Android:Performance</a>
  
</span>


    </p>
    
    
      <div class="sharing">
  
  
  
  
	<!-- JiaThis Button BEGIN -->
<div class="jiathis_style_32x32">
	<a class="jiathis_button_qzone"></a>
	<a class="jiathis_button_tsina"></a>
	<a class="jiathis_button_tqq"></a>
	<a class="jiathis_button_weixin"></a>
	<a class="jiathis_button_googleplus"></a>
	<a class="jiathis_button_renren"></a>
	<a class="jiathis_button_linkedin"></a>
	<a class="jiathis_button_douban"></a>
	<a href="http://www.jiathis.com/share?uid=1723296" class="jiathis jiathis_txt jtico jtico_jiathis" target="_blank"></a>
	<a class="jiathis_counter_style"></a>
</div>
<script type="text/javascript">
var jiathis_config = {data_track_clickback:'true'};
</script>
<script type="text/javascript" src="http://v3.jiathis.com/code/jia.js?uid=1342501457879302" charset="utf-8"></script>
<!-- JiaThis Button END -->
  
  
</div>

    
    <br></br>
    
    <p class="meta">
      
        <a class="basic-alignment left" href="/android-performance-render/" title="Previous Post: Android性能优化之渲染篇">&laquo; Android性能优化之渲染篇</a>
      
      
        <a class="basic-alignment right" href="/android-performance-memory/" title="Next Post: Android性能优化之内存篇">Android性能优化之内存篇 &raquo;</a>
      
    </p>
  </footer>
</article>

  <section>
    <h1>Comments</h1>
    <div id="disqus_thread" aria-live="polite"><noscript>Please enable JavaScript to view the <a href="http://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
</div>
  </section>


</div>


    </div>
  </div>
  <footer role="contentinfo"><p>
  Copyright &copy; 2015 - HuKai -
  <span class="credit">Powered by <a href="http://octopress.org">Octopress</a> - 本站作品采用 <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享 署名-非商业性使用-相同方式共享 4.0 国际 许可协议</a>进行许可.</span> 
  
  <script src="/javascripts/modernizr-2.0.js"></script>
  <script src="/javascripts/ender.js"></script>
  <script src="/javascripts/octopress.js" type="text/javascript"></script>
</p>

</footer>
  

<script type="text/javascript">
      var disqus_shortname = 'kesenhoo';
      
        
        // var disqus_developer = 1;
        var disqus_identifier = 'http://hukai.me/android-performance-compute/';
        var disqus_url = 'http://hukai.me/android-performance-compute/';
        var disqus_script = 'embed.js';
      
    (function () {
      var dsq = document.createElement('script'); dsq.type = 'text/javascript'; dsq.async = true;
      dsq.src = 'http://' + disqus_shortname + '.disqus.com/' + disqus_script;
      (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(dsq);
    }());
</script>











</body>
</html>
