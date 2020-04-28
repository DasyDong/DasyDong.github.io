---
title: "使用Github Pages & Jekyll 建独立博客"
layout: post
date: 2017-01-24 22:48
image: /img/images/markdown.jpg
headerImage: false
tag:
- github
- python
category: blog
author: dong
description: Markdown summary with different options
# jemoji: '<img class="emoji" title=":ramen:" alt=":ramen:" src="https://assets.github.com/images/icons/emoji/unicode/1f35c.png" height="20" width="20" align="absmiddle">'
---


### 目录
- [配置和使用Github](#menuIndex1)
- [使用GitHub Pages建立博客](#menuIndex9)
- [Jekyll模板系统](#menuIndex12)
- [搭建本地jekyll环境](#menuIndex18)

<div class="entry">
        <p>文章出处：<a href="http://beiyuu.com/github-pages/" target="_blank" class="external">http://beiyuu.com/github-pages/</a></p>

<p><a href="http://github.com" title="Github" target="_blank" class="external">Github</a>很好的将代码和社区联系在了一起，于是发生了很多有趣的事情，世界也因为他美好了一点点。Github作为现在最流行的代码仓库，已经得到很多大公司和项目的青睐，比如<a href="https://github.com/jquery/jquery" title="jQuery@github" target="_blank" class="external">jQuery</a>、<a href="https://github.com/twitter/bootstrap" title="Twitter@github" target="_blank" class="external">Twitter</a>等。为使项目更方便的被人理解，介绍页面少不了，甚至会需要完整的文档站，Github替你想到了这一点，他提供了<a href="http://pages.github.com/" title="Github Pages" target="_blank" class="external">Github Pages</a>的服务，不仅可以方便的为项目建立介绍站点，也可以用来建立个人博客。</p>

<p>Github Pages有以下几个优点：</p>

<ul>
    <li>轻量级的博客系统，没有麻烦的配置</li>
    <li>使用标记语言，比如<a href="http://markdown.tw" target="_blank" class="external">Markdown</a></li>
    <li>无需自己搭建服务器</li>
    <li>根据Github的限制，对应的每个站有300MB空间</li>
    <li>可以绑定自己的域名</li>
</ul>


<p>当然他也有缺点：</p>

<ul>
<li>使用<a href="https://github.com/mojombo/jekyll" title="Jekyll" target="_blank" class="external">Jekyll</a>模板系统，相当于静态页发布，适合博客，文档介绍等。</li>
<li>动态程序的部分相当局限，比如没有评论，不过还好我们有解决方案。</li>
<li>基于Git，很多东西需要动手，不像Wordpress有强大的后台</li>
</ul>


<p>大致介绍到此，作为个人博客来说，简洁清爽的表达自己的工作、心得，就已达目标，所以Github Pages是我认为此需求最完美的解决方案了。</p>

<h2 id="menuIndex1">配置和使用Github</h2>

<p>Git是版本管理的未来，他的优点我不再赘述，相关资料很多。推荐这本<a href="http://progit.org/book/zh/" title="Pro Git中文版" target="_blank" class="external">Git中文教程</a>。</p>

<p>要使用Git，需要安装它的客户端，推荐在Linux下使用Git，会比较方便。Windows版的下载地址在这里：<a href="http://code.google.com/p/msysgit/downloads/list" title="Windows版Git客户端" target="_blank" class="external">http://code.google.com/p/msysgit/downloads/list</a>。其他系统的安装也可以参考官方的<a href="http://help.github.com/mac-set-up-git/" title="Mac下Git安装" target="_blank" class="external">安装教程</a>。</p>

<p>下载安装客户端之后，各个系统的配置就类似了，我们使用windows作为例子，Linux和Mac与此类似。</p>

<p>在Windows下，打开Git Bash，其他系统下面则打开终端（Terminal）：

<h3 id="menuIndex2">1、检查SSH keys的设置</h3>

<p>首先我们需要检查你电脑上现有的ssh key：</p>

<pre class="prettyprint linenums"><ol class="linenums"><li class="L0"><code><span class="pln">$ cd </span><span class="pun">~/.</span><span class="pln">ssh</span></code></li></ol></pre>

<p>如果显示“No such file or directory”，跳到第三步，否则继续。</p>

<h3 id="menuIndex3">2、备份和移除原来的ssh key设置：</h3>

<p>因为已经存在key文件，所以需要备份旧的数据并删除：</p>

<pre class="prettyprint linenums"><ol class="linenums"><li class="L0"><code><span class="pln">$ ls</span></code></li><li class="L1"><code><span class="pln">config  id_rsa  id_rsa</span><span class="pun">.</span><span class="pln">pub  known_hosts</span></code></li><li class="L2"><code><span class="pln">$ mkdir key_backup</span></code></li><li class="L3"><code><span class="pln">$ cp id_rsa</span><span class="pun">*</span><span class="pln"> key_backup</span></code></li><li class="L4"><code><span class="pln">$ rm id_rsa</span><span class="pun">*</span></code></li></ol></pre>

<h3 id="menuIndex4">3、生成新的SSH Key：</h3>

<p>输入下面的代码，就可以生成新的key文件，我们只需要默认设置就好，所以当需要输入文件名的时候，回车就好。</p>

<pre class="prettyprint linenums"><ol class="linenums"><li class="L0"><code><span class="pln">$ ssh</span><span class="pun">-</span><span class="pln">keygen </span><span class="pun">-</span><span class="pln">t rsa </span><span class="pun">-</span><span class="pln">C </span><span class="str">"邮件地址@youremail.com"</span></code></li><li class="L1"><code><span class="typ">Generating</span><span class="pln"> </span><span class="kwd">public</span><span class="pun">/</span><span class="kwd">private</span><span class="pln"> rsa key pair</span><span class="pun">.</span></code></li><li class="L2"><code><span class="typ">Enter</span><span class="pln"> file </span><span class="kwd">in</span><span class="pln"> which to save the key </span><span class="pun">(</span><span class="str">/Users/</span><span class="pln">your_user_directory</span><span class="pun">/.</span><span class="pln">ssh</span><span class="pun">/</span><span class="pln">id_rsa</span><span class="pun">):&lt;回车就好&gt;</span></code></li></ol></pre>

<p>然后系统会要你输入加密串（<a href="http://help.github.com/ssh-key-passphrases/" target="_blank" class="external">Passphrase</a>）：</p>

<pre class="prettyprint linenums"><ol class="linenums"><li class="L0"><code><span class="typ">Enter</span><span class="pln"> passphrase </span><span class="pun">(</span><span class="pln">empty </span><span class="kwd">for</span><span class="pln"> </span><span class="kwd">no</span><span class="pln"> passphrase</span><span class="pun">):&lt;输入加密串&gt;</span></code></li><li class="L1"><code><span class="typ">Enter</span><span class="pln"> same passphrase again</span><span class="pun">:&lt;再次输入加密串&gt;</span></code></li></ol></pre>

<p>最后看到这样的界面，就成功设置ssh key了：
<img src="/img/images/githubpages/ssh-key-set.png" alt="ssh key success"></p>

<h3 id="menuIndex5">4、添加SSH Key到GitHub：</h3>

<p>在本机设置SSH Key之后，需要添加到GitHub上，以完成SSH链接的设置。</p>

<p>用文本编辑工具打开id_rsa.pub文件，如果看不到这个文件，你需要设置显示隐藏文件。准确的复制这个文件的内容，才能保证设置的成功。</p>

<p>在GitHub的主页上点击设置按钮：
<img src="/img/images/githubpages/github-account-setting.png" alt="github account setting"></p>

<p>选择SSH Keys项，把复制的内容粘贴进去，然后点击Add Key按钮即可：
<img src="/img/images/githubpages/bootcamp_1_ssh.png" alt="set ssh keys"></p>

<p>PS：如果需要配置多个GitHub账号，可以参看这个<a href="http://omiga.org/blog/archives/2269" target="_blank" class="external">多个github帐号的SSH key切换</a>，不过需要提醒一下的是，如果你只是通过这篇文章中所述配置了Host，那么你多个账号下面的提交用户会是一个人，所以需要通过命令<code>git config --global --unset user.email</code>删除用户账户设置，在每一个repo下面使用<code>git config --local user.email '你的github邮箱@mail.com'</code> 命令单独设置用户账户信息</p>

<h3 id="menuIndex6">5、测试一下</h3>

<p>可以输入下面的命令，看看设置是否成功，<code>git@github.com</code>的部分不要修改：</p>

<pre class="prettyprint linenums"><ol class="linenums"><li class="L0"><code><span class="pln">$ ssh </span><span class="pun">-</span><span class="pln">T git@github</span><span class="pun">.</span><span class="pln">com</span></code></li></ol></pre>

<p>如果是下面的反应：</p>

<pre class="prettyprint linenums"><ol class="linenums"><li class="L0"><code><span class="typ">The</span><span class="pln"> authenticity of host </span><span class="str">'github.com (207.97.227.239)'</span><span class="pln"> can</span><span class="str">'t be established.</span></code></li><li class="L1"><code><span class="str">RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.</span></code></li><li class="L2"><code><span class="str">Are you sure you want to continue connecting (yes/no)?</span></code></li></ol></pre>

<p>不要紧张，输入<code>yes</code>就好，然后会看到：</p>

<pre class="prettyprint linenums"><ol class="linenums"><li class="L0"><code><span class="typ">Hi</span><span class="pln"> </span><span class="str">&lt;em&gt;</span><span class="pln">username</span><span class="pun">&lt;/</span><span class="pln">em</span><span class="pun">&gt;!</span><span class="pln"> </span><span class="typ">You</span><span class="str">'ve successfully authenticated, but GitHub does not provide shell access.</span></code></li></ol></pre>

<h3 id="menuIndex7">6、设置你的账号信息</h3>

<p>现在你已经可以通过SSH链接到GitHub了，还有一些个人信息需要完善的。</p>

<p>Git会根据用户的名字和邮箱来记录提交。GitHub也是用这些信息来做权限的处理，输入下面的代码进行个人信息的设置，把名称和邮箱替换成你自己的，名字必须是你的真名，而不是GitHub的昵称。</p>

<pre class="prettyprint linenums"><ol class="linenums"><li class="L0"><code><span class="pln">$ git config </span><span class="pun">--</span><span class="kwd">global</span><span class="pln"> user</span><span class="pun">.</span><span class="pln">name </span><span class="str">"你的名字"</span></code></li><li class="L1"><code><span class="pln">$ git config </span><span class="pun">--</span><span class="kwd">global</span><span class="pln"> user</span><span class="pun">.</span><span class="pln">email </span><span class="str">"your_email@youremail.com"</span></code></li></ol></pre>

<h2 id="menuIndex9">使用GitHub Pages建立博客</h2>

<p>与GitHub建立好链接之后，就可以方便的使用它提供的Pages服务，GitHub Pages分两种，一种是你的GitHub用户名建立的<code>username.github.io</code>这样的用户&amp;组织页（站），另一种是依附项目的pages。</p>

<p>创建好<code>username.github.io</code>项目之后，注意这一定要是这个格式，要不然不能访问的<img src="/img/images/githubpages/repositoryname.png" alt="repositoryname">


提交一个<code>index.html</code>文件，然后<code>push</code>到GitHub的<code>master</code>分支（也就是普通意义上的主干）。第一次页面生效需要一些时间，大概10分钟左右。</p>
<p>发布到github, 选择分支 <img src="/img/images/githubpages/github.png" alt="github"></p> </p>
<p>生效之后，访问<code>username.github.io</code>就可以看到你上传的页面了，<a href="http://beiyuu.github.com" target="_blank" class="external">beiyuu.github.com</a>就是一个例子。</p>


<h2 id="menuIndex12">Jekyll模板系统</h2>

<p>GitHub Pages为了提供对HTML内容的支持，选择了<a href="https://github.com/mojombo/jekyll" title="Jekyll" target="_blank" class="external">Jekyll</a>作为模板系统，Jekyll是一个强大的静态模板系统，作为个人博客使用，基本上可以满足要求，也能保持管理的方便，你可以查看<a href="https://github.com/mojombo/jekyll/blob/master/README.textile" target="_blank" class="external">Jekyll官方文档</a>。</p>

<p>你可以直接fork<a href="https://github.com/beiyuu/beiyuu.github.com" target="_blank" class="external">我的项目</a>，然后改名，就有了你自己的满足Jekyll要求的文档了，当然你也可以按照下面的介绍自己创建。</p>

<h3 id="menuIndex13">Jekyll基本结构</h3>

<p>Jekyll的核心其实就是一个文本的转换引擎，用你最喜欢的标记语言写文档，可以是Markdown、Textile或者HTML等等，再通过<code>layout</code>将文档拼装起来，根据你设置的URL规则来展现，这些都是通过严格的配置文件来定义，最终的产出就是web页面。</p>

<p>基本的Jekyll结构如下：</p>

<pre class="prettyprint linenums"><ol class="linenums"><li class="L0"><code><span class="pun">|--</span><span class="pln"> _config</span><span class="pun">.</span><span class="pln">yml</span></code></li><li class="L1"><code><span class="pun">|--</span><span class="pln"> _includes</span></code></li><li class="L2"><code><span class="pun">|--</span><span class="pln"> _layouts</span></code></li><li class="L3"><code><span class="pun">|</span><span class="pln">   </span><span class="pun">|--</span><span class="pln"> </span><span class="kwd">default</span><span class="pun">.</span><span class="pln">html</span></code></li><li class="L4"><code><span class="pun">|</span><span class="pln">   </span><span class="str">`-- post.html</span></code></li><li class="L5"><code><span class="str">|-- _posts</span></code></li><li class="L6"><code><span class="str">|   |-- 2007-10-29-why-every-programmer-should-play-nethack.textile</span></code></li><li class="L7"><code><span class="str">|   `</span><span class="pun">--</span><span class="pln"> </span><span class="lit">2009</span><span class="pun">-</span><span class="lit">04</span><span class="pun">-</span><span class="lit">26</span><span class="pun">-</span><span class="pln">barcamp</span><span class="pun">-</span><span class="pln">boston</span><span class="pun">-</span><span class="lit">4</span><span class="pun">-</span><span class="pln">roundup</span><span class="pun">.</span><span class="pln">textile</span></code></li><li class="L8"><code><span class="pun">|--</span><span class="pln"> _site</span></code></li><li class="L9"><code><span class="str">`-- index.html</span></code></li></ol></pre>

<p>简单介绍一下他们的作用：</p>

<h4>_config.yml</h4>

<p>配置文件，用来定义你想要的效果，设置之后就不用关心了。</p>

<h4>_includes</h4>

<p>可以用来存放一些小的可复用的模块，方便通过<code>{ % include file.ext %}</code>（去掉前两个{中或者{与%中的空格，下同）灵活的调用。这条命令会调用_includes/file.ext文件。</p>

<h4>_layouts</h4>

<p>这是模板文件存放的位置。模板需要通过<a href="https://github.com/mojombo/jekyll/wiki/YAML-Front-Matter" target="_blank" class="external">YAML front matter</a>来定义，后面会讲到，<code>{ { content }}</code>标记用来将数据插入到这些模板中来。</p>

<h4>_posts</h4>

<p>你的动态内容，一般来说就是你的博客正文存放的文件夹。他的命名有严格的规定，必须是<code>2012-02-22-artical-title.MARKUP</code>这样的形式，MARKUP是你所使用标记语言的文件后缀名，根据_config.yml中设定的链接规则，可以根据你的文件名灵活调整，文章的日期和标记语言后缀与文章的标题的独立的。</p>

<h4>_site</h4>

<p>这个是Jekyll生成的最终的文档，不用去关心。最好把他放在你的<code>.gitignore</code>文件中忽略它。</p>

<h4>其他文件夹</h4>

<p>你可以创建任何的文件夹，在根目录下面也可以创建任何文件，假设你创建了<code>project</code>文件夹，下面有一个<code>github-pages.md</code>的文件，那么你就可以通过<code>yoursite.com/project/github-pages</code>访问的到，如果你是使用一级域名的话。文件后缀可以是<code>.html</code>或者<code>markdown</code>或者<code>textile</code>。这里还有很多的例子：<a href="https://github.com/mojombo/jekyll/wiki/Sites" target="_blank" class="external">https://github.com/mojombo/jekyll/wiki/Sites</a></p>

<h3 id="menuIndex14">Jekyll的配置</h3>

<p>Jekyll的配置写在_config.yml文件中，可配置项有很多，我们不去一一追究了，很多配置虽有用但是一般不需要去关心，<a href="https://github.com/mojombo/jekyll/wiki/configuration" target="_blank" class="external">官方配置文档</a>有很详细的说明，确实需要了可以去这里查，我们主要说两个比较重要的东西，一个是<code>Permalink</code>，还有就是自定义项。</p>

<p><code>Permalink</code>项用来定义你最终的文章链接是什么形式，他有下面几个变量：</p>

<ul>
<li><code>year</code> 文件名中的年份</li>
<li><code>month</code> 文件名中的月份</li>
<li><code>day</code> 文件名中的日期</li>
<li><code>title</code> 文件名中的文章标题</li>
<li><code>categories</code> 文章的分类，如果文章没有分类，会忽略</li>
<li><code>i-month</code> 文件名中的除去前缀0的月份</li>
<li><code>i-day</code> 文件名中的除去前缀0的日期</li>
</ul>


<p>看看最终的配置效果：</p>

<ul>
<li><code>permalink: pretty</code> /2009/04/29/slap-chop/index.html</li>
<li><code>permalink: /:month-:day-:year/:title.html</code> /04-29-2009/slap-chop.html</li>
<li><code>permalink: /blog/:year/:month/:day/:title</code> /blog/2009/04/29/slap-chop/index.html</li>
</ul>


<p>我使用的是：</p>

<ul>
<li><code>permalink: /:title</code> /github-pages</li>
</ul>


<p>自定义项的内容，例如我们定义了<code>title:BeiYuu的博客</code>这样一项，那么你就可以在文章中使用<code>{ { site.title }}</code>来引用这个变量了，非常方便定义些全局变量。</p>

<h3 id="menuIndex15">YAML Front Matter和模板变量</h3>

<p>对于使用YAML定义格式的文章，Jekyll会特别对待，他的格式要求比较严格，必须是这样的形式：</p>

<pre class="prettyprint linenums"><ol class="linenums"><li class="L0"><code><span class="pun">---</span></code></li><li class="L1"><code><span class="pln">layout</span><span class="pun">:</span><span class="pln"> post</span></code></li><li class="L2"><code><span class="pln">title</span><span class="pun">:</span><span class="pln"> </span><span class="typ">Blogging</span><span class="pln"> </span><span class="typ">Like</span><span class="pln"> a </span><span class="typ">Hacker</span></code></li><li class="L3"><code><span class="pun">---</span></code></li></ol></pre>

<p>前后的<code>---</code>不能省略，在这之间，你可以定一些你需要的变量，layout就是调用<code>_layouts</code>下面的某一个模板，他还有一些其他的变量可以使用：</p>

<ul>
<li><code>permalink</code> 你可以对某一篇文章使用通用设置之外的永久链接</li>
<li><code>published</code> 可以单独设置某一篇文章是否需要发布</li>
<li><code>category</code> 设置文章的分类</li>
<li><code>tags</code> 设置文章的tag</li>
</ul>


<p>上面的<code>title</code>就是自定义的内容，你也可以设置其他的内容，在文章中可以通过<code>{ { page.title }}</code>这样的形式调用。</p>

<p>模板变量，我们之前也涉及了不少了，还有其他需要的变量，可以参考官方的文档：<a href="https://github.com/mojombo/jekyll/wiki/template-data" title="Jekyll Template Data" target="_blank" class="external">https://github.com/mojombo/jekyll/wiki/template-data</a></p>

<h2 id="menuIndex18">搭建本地jekyll环境</h2>

<p>这里主要介绍一下在Mac OS X下面的安装过程，其他操作系统可以参考官方的<a href="https://github.com/mojombo/jekyll/wiki/Install" target="_blank" class="external">jekyll安装</a>。</p>

<p>作为生活在水深火热的墙内人民，有必要进行下面一步修改gem的源，方便我们更快的下载所需组建：</p>

<pre class="prettyprint linenums"><ol class="linenums"><li class="L0"><code><span class="pln">sudo gem sources </span><span class="pun">--</span><span class="pln">remove http</span><span class="pun">:</span><span class="com">//rubygems.org/ </span></code></li><li class="L1"><code><span class="pln">sudo gem sources </span><span class="pun">-</span><span class="pln">a http</span><span class="pun">:</span><span class="com">//ruby.taobao.org/ </span></code></li></ol></pre>

<p>然后用Gem安装jekyll</p>

<pre class="prettyprint linenums"><ol class="linenums"><li class="L0"><code><span class="pln">$ gem install jekyll</span></code></li></ol></pre>

<p>不过一般如果有出错提示，你可能需要这样安装：</p>

<pre class="prettyprint linenums"><ol class="linenums"><li class="L0"><code><span class="pln">$ sudo gem install jekyll</span></code></li></ol></pre>

<p>我到了这一步的时候总是提示错误<code>Failed to build gem native extension</code>，很可能的一个原因是没有安装rvm，<a href="https://rvm.io/rvm/install/" target="_blank" class="external">rvm的安装</a>可以参考这里，或者敲入下面的命令：</p>

<pre class="prettyprint linenums"><ol class="linenums"><li class="L0"><code><span class="pln">$ curl </span><span class="pun">-</span><span class="pln">L https</span><span class="pun">:</span><span class="com">//get.rvm.io | bash -s stable --ruby</span></code></li></ol></pre>

<p>然后还需要安装Markdown的解释器，这个需要在你的_config.yml里面设置<code>markdown:rdiscount</code>：</p>

<pre class="prettyprint linenums"><ol class="linenums"><li class="L0"><code><span class="pln">$ gem install jekyll rdiscount</span></code></li></ol></pre>

<p>好了，如果一切顺利的话，本地环境就基本搭建完成了，进入之前我们建立的博客目录，运行下面的命令：</p>

<pre class="prettyprint linenums"><ol class="linenums"><li class="L0"><code><span class="pln">$ jekyll </span><span class="pun">--</span><span class="pln">server</span></code></li></ol></pre>

<p>这个时候，你就可以通过<code>localhost:4000</code>来访问了。还有关于<a href="http://jekyllbootstrap.com/" target="_blank" class="external">jekyll bootstrap</a>的资料，需要自己修改调试的，可以研究一下。</p>

<p>我在这个过程中还遇到两个诡异的没有解决的问题，一个是我放在根目录下面的blog.md等文件，在GitHub的pages服务上一切正常，可以通过<code>beiyuu.com/blog</code>访问的到，但是在本地环境下，总是<code>not found</code>，很是让人郁闷，看生成的<code>_site</code>目录下面的文件，也是正常的<code>blog.html</code>，但就是找不到，只有当我把URL改为<code>localhost:4000/blog.html</code>的时候，才能访问的到，环境不同真糟糕。</p>

<p>还有一个是关于<code>category</code>的问题，根据<code>YAML</code>的语法，我们在文章头部可以定义文章所属的类别，也可以定义为<code>category:[blog,rss]</code>这样子的多类别，我在本地试一切正常，但是push到GitHub之后，就无法读取了，真让人着急，没有办法，只能采用别的办法满足我的需求了。这里还有一篇<a href="http://chxt6896.github.com/blog/2012/02/13/blog-jekyll-native.html" target="_blank" class="external">Jekyll 本地调试之若干问题</a>，安装中如果有其他问题，也可以对照参考一下。</p>

<h2 id="menuIndex19">结语</h2>

<p>如果你跟着这篇不那么详尽的教程，成功搭建了自己的博客，恭喜你！剩下的就是保持热情的去写自己的文章吧。</p>
