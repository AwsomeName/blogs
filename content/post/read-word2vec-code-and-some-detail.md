---
title: "Read Word2vec Code and Some Detail"
date: 2019-06-09T18:34:33+08:00
lastmod: 2019-06-09T18:34:33+08:00
draft: false
keywords: []
description: "朋友发了一个有趣的[链接](https://mp.weixin.qq.com/s?__biz=MzU3NjE4NjQ4MA==&mid=2247485614&idx=1&sn=6ff3dc634cc8a1f354695fd4e139b0b3&chksm=fd16f9b1ca6170a78e721df8d34fce70303aa67ee0d2bfb498b4d0015a2dffaa939bbe077ff1&mpshare=1&scene=24&srcid=0607FNjJEeaXpNxLNtpWugS2&pass_ticket=V5RKkOY%2BgoyS%2Bmyz2ZPNVWpSgMlbqy89xa6maVii56%2BWngb%2FbfH6eFjtcPUbLAyB#rd)过来，关于word2vec的，然后就有了这篇文章"
tags: ["word2vec","NLP"]
categories: ["NLP"]
author: "lc"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: false
autoCollapseToc: true
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

---

朋友(Q God)发了一个有趣的[链接](https://mp.weixin.qq.com/s?__biz=MzU3NjE4NjQ4MA==&mid=2247485614&idx=1&sn=6ff3dc634cc8a1f354695fd4e139b0b3&chksm=fd16f9b1ca6170a78e721df8d34fce70303aa67ee0d2bfb498b4d0015a2dffaa939bbe077ff1&mpshare=1&scene=24&srcid=0607FNjJEeaXpNxLNtpWugS2&pass_ticket=V5RKkOY%2BgoyS%2Bmyz2ZPNVWpSgMlbqy89xa6maVii56%2BWngb%2FbfH6eFjtcPUbLAyB#rd)过来，关于word2vec的，然后就有了这篇文章
<!--more-->

本文讨论了关于word2vec的经典解释：Skip-Gram模型，负采样训练。

本文中，上文主要指朋友发的[链接](https://mp.weixin.qq.com/s?__biz=MzU3NjE4NjQ4MA==&mid=2247485614&idx=1&sn=6ff3dc634cc8a1f354695fd4e139b0b3&chksm=fd16f9b1ca6170a78e721df8d34fce70303aa67ee0d2bfb498b4d0015a2dffaa939bbe077ff1&mpshare=1&scene=24&srcid=0607FNjJEeaXpNxLNtpWugS2&pass_ticket=V5RKkOY%2BgoyS%2Bmyz2ZPNVWpSgMlbqy89xa6maVii56%2BWngb%2FbfH6eFjtcPUbLAyB#rd)

本文中，w2v就是指SkipGram负采样的训练方式。

本文的[代码][w2c_github]地址
那么就开始吧！！

(这也是本人MarkDown写的第一篇技术博客，如有问题请多指教)


# 上文中的观点和问题
上文指出，大部分的博客关于w2c的解释如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/YicUhk5aAGtBKLuofqf0zXxr8VV8yRMF5IJs92SekN5KTfYqxy7gkZzMnJa8ZbpmFibRUaF1tCeibPHPh4uYB8c5A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
但是上文中说，上图只能看出有两个向量，而w2c中的实现和这个完全不同

实际的实现中，每个单词有两个向量，一个是作为中心词的向量（也是最终作为词向量的向量），一个是作为上下文时的向量。

分别保存在两个矩阵（也就是数组中）。

而且这里使用了特殊的初始化技巧，对于中心词向量，是随机初始化的。对于上下文词向量，是使用零初始化的。

貌似在word2vec的论文中，并没有提到这一点。所以上文作者提出了他的质疑。

上文的编辑贴出作者的观点：
>   因为负样本 (Negative Sample) 来自全文上下，并没太根据词频来定权重，这样选哪个单词都可以，通常这个词的向量还没经过多少训练。
>   而如果这个向量已经有了一个值，那么它就可以随意移动 (Move Randomly) 中心词了。
>   解决方法是，把所有负样本设为零，这样依赖只有那些比较高频出现的向量，才会影响到另外一个向量的表征。

原作者的观点[地址](https://github.com/bollu/bollu.github.io)
***
# 实地考察的w2c的代码实现

通过上文，和实际考察，可以确定：
>   syn0数组，存储每个词作为中心词的向量，也是最终的词向量。是随机初始化的
```
https://github.com/tmikolov/word2vec/blob/20c129af10659f7c50e86e3be406df663beff438/word2vec.c#L369
    for (a = 0; a < vocab_size; a++) for (b = 0; b < layer1_size; b++) {
        next_random = next_random * (unsigned long long)25214903917 + 11;
        syn0[a * layer1_size + b] = 
        (((next_random & 0xFFFF) / (real)65536) - 0.5) / layer1_size;
    }
```
>   syn1neg数组，存储每个词作为上下文词的向量，是零初始化的。
```
https://github.com/tmikolov/word2vec/blob/20c129af10659f7c50e86e3be406df663beff438/word2vec.c#L365
    for (a = 0; a < vocab_size; a++) for (b = 0; b < layer1_size; b++)
        syn1neg[a * layer1_size + b] = 0;
```

核心的训练函数是：
`TrainModelThread(void *id)`

其中的核心代码是：
```
if (negative > 0) for (d = 0; d < negative + 1; d++) {
    if (d == 0) {
        target = word;
        label = 1;
    } else {
        next_random = next_random * (unsigned long long)25214903917 + 11;
        target = table[(next_random >> 16) % table_size];
        if (target == 0) target = next_random % (vocab_size - 1) + 1;
        if (target == word) continue;
        label = 0;
    }
    l2 = target * layer1_size;
    f = 0;
    for (c = 0; c < layer1_size; c++) f += syn0[c + l1] * syn1neg[c + l2];
    if (f > MAX_EXP) g = (label - 1) * alpha;
    else if (f < -MAX_EXP) g = (label - 0) * alpha;
    else g = (label - expTable[(int)((f + MAX_EXP) * (EXP_TABLE_SIZE / MAX_EXP / 2))]) * alpha;
    for (c = 0; c < layer1_size; c++) neu1e[c] += g * syn1neg[c + l2];
    for (c = 0; c < layer1_size; c++) syn1neg[c + l2] += g * syn0[c + l1];
}
// Learn weights input -> hidden
for (c = 0; c < layer1_size; c++) syn0[c + l1] += neu1e[c];
```
这段代码核心功能是，针对某个中心词，做基于负采样的计算，累积的梯度会更新到syn0，也就是作为中心词的数组。

可以看出，确实是用了两个数组，而且syn1neg的更新并非是从syn0中直接抽取，是自己逐步更新的。
（在看代码之前，我猜测，上下文的数组只是大部分为零，从中心词数组中抽取高频词的向量更新自己，这里看来我猜测的不对，打脸了）
这里基本验证了(虽然上文标题起的些许夸张)，原作者的观点是对的。有些细节的问题，后面讨论。


***
# 本文观点和私货
这确实是一个问题。这可能算不上是学术造假，但是这个问题确实是会给尝试复现的人造成困扰，困惑。
原作者的分析也是很到位。但是有一点，就是**“没太根据词频来定位权重”**，这一点我有些疑虑。
w2c中是根据`table`来抽样的，这个数据结构的生成可以看到是根据词频生成的，代码如下。
```
void InitUnigramTable() {
  int a, i;
  double train_words_pow = 0;
  double d1, power = 0.75;
  table = (int *)malloc(table_size * sizeof(int));
  for (a = 0; a < vocab_size; a++) train_words_pow += pow(vocab[a].cn, power);
  i = 0;
  d1 = pow(vocab[i].cn, power) / train_words_pow;
  for (a = 0; a < table_size; a++) {
    table[a] = i;
    if (a / (double)table_size > d1) {
      i++;
      d1 += pow(vocab[i].cn, power) / train_words_pow;
    }
    if (i >= vocab_size) i = vocab_size - 1;
  }
}
```
可以看出，词频越高的词，在table中占的坑位越多，抽中的概率就越大。

##  其他的发现，关于skipgram模式的理解

最早我对skipgram的理解，是中心词f(focus)，对应着上下文的词c(content)，f对每个c是正例，f对其他词是负例。

而在这个w2v中，实现是这样的，对中心词f，当前上下文中的c，c对f是正例，c对其他是负例。代码如下：
```
void *TrainModelThread(void *id) {
    ...
    while (1) {
        ...
        if (cbow) {  //train the cbow architecture
            ...
        } else {  //train skip-gram
            for (a = b; a < window * 2 + 1 - b; a++) if (a != window) {
                c = sentence_position - window + a;
                if (c < 0) continue;
                if (c >= sentence_length) continue;
                last_word = sen[c];
                if (last_word == -1) continue;
                l1 = last_word * layer1_size;
                for (c = 0; c < layer1_size; c++) neu1e[c] = 0;
                // HIERARCHICAL SOFTMAX
                if (hs) for (d = 0; d < vocab[word].codelen; d++) {
                    ...
                }
                // NEGATIVE SAMPLING
                if (negative > 0) for (d = 0; d < negative + 1; d++) {
                  if (d == 0) {
                    target = word;
                    label = 1;
                  } else {
                    next_random = next_random * (unsigned long long)25214903917 + 11;
                    target = table[(next_random >> 16) % table_size];
                    if (target == 0) target = next_random % (vocab_size - 1) + 1;
                    if (target == word) continue;
                    label = 0;
                  }
                  l2 = target * layer1_size;
                  f = 0;
                  for (c = 0; c < layer1_size; c++) f += syn0[c + l1] * syn1neg[c + l2];
                  if (f > MAX_EXP) g = (label - 1) * alpha;
                  else if (f < -MAX_EXP) g = (label - 0) * alpha;
                  else g = (label - expTable[(int)((f + MAX_EXP) * (EXP_TABLE_SIZE / MAX_EXP / 2))]) * alpha;
                  for (c = 0; c < layer1_size; c++) neu1e[c] += g * syn1neg[c + l2];
                  for (c = 0; c < layer1_size; c++) syn1neg[c + l2] += g * syn0[c + l1];
                }
                // Learn weights input -> hidden
                for (c = 0; c < layer1_size; c++) syn0[c + l1] += neu1e[c];
            }
        }
        ...
    }
  ...
}
```

以上是关键代码，简单的讲，word是当前句子中的中心词（词f），last_word是当前的上下文词（词c），负采样采到的词是target。
l1是last_word的词向量坐标，l2是target负采样的词的向量坐标，训练中，是基于这两个坐标指示的向量，计算梯度和更新上下文词向量（syn1neg数组)
所以这里，正样例是c到f，负样例是c到其他。和我理解的是相反的。

当然这从原理上讲，貌似无伤大雅，因为我理解的word2vec的假设是，如果两个词很像，它会有相似的上下文。上下文的词和中心词的对应关系本身也没有方向性。不过还是一个很有意思的问题。


所以下一步我会看一下tensorflow的nec_loss是怎样实现的，对比一下。
近期我也会看一下w2c的原论文，证实以上种种疑惑，再来补充。

如果有任何问题请和我指出，感谢您的指证。
可以在我的[wordpress](http://blog.tureornot.com/2019/06/09/关于word2wec的一点讨论)留言。




[w2c_github]:(https://github.com/tmikolov/word2vec/blob/20c129af10659f7c50e86e3be406df663beff438/word2vec.c)


