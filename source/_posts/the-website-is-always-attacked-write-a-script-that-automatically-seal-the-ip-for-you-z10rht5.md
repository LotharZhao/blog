<h1>网站总被攻击？写个自动封禁 IP 的脚本给你</h1>
<hr />
<ul>
<li>网站总被攻击？写个自动封禁 IP 的脚本给你 - 微信公众平台</li>
<li><a href="https://mp.weixin.qq.com/s/7dMgSrY6F6CDqKsVor8vSQ">https://mp.weixin.qq.com/s/7dMgSrY6F6CDqKsVor8vSQ</a></li>
<li>网站总被攻击？写个自动封禁 IP 的脚本给你</li>
<li>2024-09-25 09:06:23</li>
</ul>
<hr />
<pre><code>每天傍晚18:00一起成长！

</code></pre>
<p>个人网站总被攻击？写个自动封禁IP的脚本给你！具体如下：</p>
<p>1.在ngnix的conf目录下创建一个 <code>blockip.conf</code>​ 文件</p>
<p>2.里面放需要封禁的IP，格式如下</p>
<pre><code>deny 1.2.3.4;
</code></pre>
<p>3.在ngnix的HTTP的配置中添加如下内容</p>
<pre><code>include blockips.conf;
</code></pre>
<p>​<img src="assets/640-20240925090624-abzkk8k.webp" alt="图片" />4.重启 ngnix</p>
<pre><code>/usr/local/nginx/sbin/nginx -s reload
</code></pre>
<p>然后你就会看到IP被封禁了，你会喜提403；<img src="https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPrAlwAPXXqf8M2e7C22QytZ3GE9QJ2cMMTqXiafQdSlhh4aVd7mrb9NLc9bRRuibFmbNtFt3iblU2tbQ/640?wx_fmt=png" alt="图片" />​</p>
<h2>小思考：如何实现使用ngnix自动封禁ip的功能</h2>
<ul>
<li>1.AWK统计access.log，记录每分钟访问超过60次的ip，然后配合nginx进行封禁</li>
<li>2.编写shell脚本</li>
<li>3.crontab定时跑脚本</li>
</ul>
<p>好了上面操作步骤列出来了，那我们先来实现第一个吧</p>
<p>​<img src="https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPrAlwAPXXqf8M2e7C22QytZp4NKlgFibte065FcqNLVO3CEz6tTiblxK3gI1Jv6ZYcQGp6tKly7z8ew/640?wx_fmt=png" alt="图片" />​</p>
<h6>操作一：AWK统计access.log，记录每分钟访问超过60次的ip</h6>
<pre><code>awk '{print $1}' access.log | sort | uniq -cd | awk '{if($1&gt;60)print $0}'

1. awk '{print $1}' access.log   取出access.log的第一列即为ip。
2. sort | uniq -cd  去重和排序
3. awk '{if($1&gt;60)print $0}' 判断重复的数量是否超过60个，超过60个就展示出来
</code></pre>
<h6>操作二：编写shell脚本，实现整体功能（写了注释代码）</h6>
<pre><code>#不能把别人IP一直封着吧，这里就清除掉了
echo &quot;&quot; &gt; /usr/local/nginx/conf/blockip.conf

#前面最开始编写的统计数据功能
ip_list=$(awk '{print $1}' access.log | sort | uniq -cd | awk '{if($1&gt;60)print $0}')

#判断这个变量是否为空
if test -z &quot;$ip_list&quot;
then
        #为空写入 11.log中，并重新启动ngnix
        echo &quot;为空&quot;  &gt;&gt; /usr/local/nginx/logs/11.log

        /usr/local/nginx/sbin/nginx -s reload

else
        #如果不为空 前面加上 deny格式和ip写入blockip.conf中
        echo &quot;deny&quot; $ip_list &gt; /usr/local/nginx/conf/blockip.conf

            #因为前面携带了行数，所有我们需要去除掉前面的行数，写入后在读取一次
        ip_list2=$(awk '{print $3}' /usr/local/nginx/conf/blockip.conf)

                #最后再把读取出来的值，在次写入到blockip.conf中
        echo &quot;deny&quot; $ip_list2&quot;;&quot;&gt; /usr/local/nginx/conf/blockip.conf

        #重启ngnix
        /usr/local/nginx/sbin/nginx -s reload
        #清空之前的日志，从最新的开始截取
        echo &quot;&quot; &gt; /usr/local/nginx/logs/access.log

fi
</code></pre>
<h6>操作三：使用crontab定时，来实现访问每分钟超过60的</h6>
<p>直接实操吧：</p>
<pre><code>crontab -e 
* * * * * cd /usr/local/nginx/logs/ &amp;&amp; sh ip_test.sh  每一分钟运行一次
systemctl restart crond.service 重启一下配置既可
</code></pre>
<p>​<img src="assets/640-20240925090624-2q0yv5v.webp" alt="图片" />​</p>
<p>感谢大家 上面就是全部思路，当然如果<strong>你有更好的思路或者方法，欢迎我们在评论区探讨。</strong></p>
<blockquote>
<p><em>链接：<a href="https://blog.csdn.net/qq_38925100">https://blog.csdn.net/qq_38925100</a></em></p>
<p><em>/article/details/123742463</em></p>
</blockquote>
<pre data-mpa-powered-by="yiban.io"><hr/><p><img class="rich_pages wxw-img" data-cropselx1="0" data-cropselx2="578" data-cropsely1="0" data-cropsely2="1028" data-galleryid="" data-ratio="1.7777777777777777" data-s="300,640" data-src="https://mmbiz.qpic.cn/mmbiz_jpg/IsrmVA0RIYP6EMRo42wrV1Qp4QQqPFg25vv4NgazABcaQvnIJoyThRqbxvBKsKOQOMg4LFd8OWLicKqbdibBHOicQ/640?wx_fmt=jpeg&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1" data-type="jpeg" data-w="1080" data-original-style="outline: 0px;box-sizing: border-box !important;overflow-wrap: break-word !important;visibility: visible !important;width: 578px !important;" data-index="6" src="https://mmbiz.qpic.cn/mmbiz_jpg/IsrmVA0RIYP6EMRo42wrV1Qp4QQqPFg25vv4NgazABcaQvnIJoyThRqbxvBKsKOQOMg4LFd8OWLicKqbdibBHOicQ/640?wx_fmt=jpeg&amp;wxfrom=5&amp;wx_lazy=1&amp;wx_co=1&amp;tp=webp" _width="578px" crossorigin="anonymous" alt="图片" data-fail="0"/><strong></strong></p></pre>
