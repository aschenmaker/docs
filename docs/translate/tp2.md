# 【翻译计划4】How does Shazam work? Music Recognition Algorithms, Fingerprinting, and Processing

Created: Mar 15, 2021 2:22 PM
link: https://www.toptal.com/algorithms/shazam-it-music-processing-fingerprinting-and-recognition
writer: JOVAN JOVANOVIC
标签: 技术, 音频处理

Jovan is an entrepreneur and engineer with a strong mathematical background and an extensive skillset for problem-solving.

【About Project】上一次说到翻译计划，似乎还是16年在点石团队的时候。“Open Eyes Project”，那时候缺乏了推动力。这次重新开始，翻译一些技术文章或者，感兴趣的内容。暂时不给他一个名字吧。`新翻译计划` 它来了。这是本系列的第二篇翻译文章。

上周，拿到实习公司的一个需求，对音频进行模式识别，大致能够区分这段音频的内容。这是一个有趣的问题，在寻求解决方案的时候，发现了这样一篇文章。讲述了Shazam（听歌识曲软件）它是怎么做到的。

以下是本文的正文内容：

# Shazam是如何工作的？音乐识别算法，声纹以及处理过程

## Introduction

你在俱乐部或者餐馆里听到了一首熟悉的歌。这首歌真诚的打动了你，尽管你已经听过上千遍，你想明天再次去听这首歌，但是你回忆不出来它的名字了。幸运的是，在这个神奇的世界里，你有一部安装了听歌识曲的手机，你得救了。你可以放轻松了，因为软件会告诉你这首歌的名字，你知道你又能听它了，只到它成为你的一部分……或者你厌倦了它。

 移动技术，和音频信号处理技术的巨大发展，赋予了我们的算法开发人员去创造音乐识别软件的能力。其中最有名的音乐识别软件之一，Shazam。如果你捕捉到一首歌的二十秒，无论是前奏(intro)，主歌(verse)还是副歌(chorus)，它都会为录音样本创造一个声纹（Voice Fingerprint，voiceprint），查询数据库，并使用音乐识别算法告诉你正在听哪一首歌。

![https://dist.lyneee.com/blog/2021-03-17-1-shazam.png](https://dist.lyneee.com/blog/2021-03-17-1-shazam.png)

Music recognition software:' Shazam'

但是Shazam是怎样工作的呢？2003年，Avery Li-Chung Wang向世界展示了Shazam的算法（作为论文被发表）。在这篇文章中，我们将介绍Shazam的音乐识别算法的基本原理。

## Analog to Digital - Sampling a Signal

模拟信号到数字信号 - 信号采样

声音到底是什么？它是某种神秘的物质，我们不能触摸，但是可以飞到我们的耳朵里，让我们听到一些东西？

当然事实并非如此。我们都知道，在现实中，声音是一种震动，它以压力与位移的机械波的形式，通过空气或者水这样的介质传播。当这种震动传到我们的耳朵，特别是鼓膜的时候，耳骨将震动进一步的传到我们的内耳深处的毛细胞（Hair Cell）。最后这些毛细胞产生电脉冲，通过听觉神经传入我们的大脑。

录音设备相当接近的还原了这个过程，利用声波的压力产生的震动转换成电信号。空气中的实际声波是一种连续的压力信号。麦克风是这种压力信号（声波）遇到的第一个电子元件，将压力信号转换成了模拟电压信号——同样也是连续（continuous）的。这种连续的信号在数组世界里没有多大的用处，所以在处理它之前，必须将它转换成一个离散（discrete）的电信号。这是通过捕获信号振幅的数字值来实现的。

这种转换涉及到了输入的量化，必然会引起少量的误差。因此，不同于一次性的转换，模拟数字转换器需要对很小的片段执行多次转换，这个过程称为采样。

![https://dist.lyneee.com/blog/2021-03-17-2-sampling.png](https://dist.lyneee.com/blog/2021-03-17-2-sampling.png)

Analog to digital signal

奈奎斯特-香农定理$^1$告诉我们，要捕获连续信号中的某个频率，必须使用特定的采样率。特别是为了捕获所有人类能够在音频信号中听到的所有频率，我们必须在人类听觉范围内的两倍频率对信号进行采样。人类的耳朵能够听到大约从20Hz到20000Hz之间的频率。因此，音频通常以44,100Hz的采样率进行记录。这是光盘的采样率，也是MPEG-1音频（VCD、MP3）最常用的采样率。Sony公司最早采用了这个频率，因为它可以以每秒25帧（PAL）或每秒30帧的速度（使用NTSC单色录像机）录制在经过修改的视频设备中，覆盖20,000Hz的带宽，在当时被认为是可以与专业的录音设备相媲美。所以当你选择采样率的时候，你可能会趋向于选择44,100Hz。

## Recording - Capturing the Sound

录音-捕获声音

记录已经被采样的音频信号很容易。因为现代声卡已经有了模数转换器，只需要一种编程语言，找到一个合适库，设置采样率、通道数量（通常是单声道或立体声），采样大小（比如16位采样）。然后像打开任何输入流一样从声卡读入行写入到字节数组中。下面有一段Java的示例代码：

```java
private AudioFormat getFormat() {
		float sampleRate = 44100;
		int sampleSizeInBits = 16;
		int channels = 1;
		boolean signed = true;     //表示数据是否是签名的
		boolean bigEndian = true;  //表示数据是否是按照大字节序或者小字节序存储的
		return new AudioFormat(sampleRate, sampleSizeIntBits, channels, signed, bigEndian);
}

final AudioFormat format = getFormat(); //按照设定填充AudioFormat
DataLine.Info info = new DataLine.Info(TargetDataLine.class, format);
final TargetDataLine line = (TargetDataLine) AudioSystem.getLine(info);
line.open(format);
line.start;
```

只需要从TargetDateLined中读取数据即可。（在这个例子中，running的标志是一个全局变量，它被另外一个线程停止-例如，我们有带有GUI的STOP按钮。）

```java
OutputStream out = new ByteArrayOutputStream();
running = true;

try {
    while (running) {
        int count = line.read(buffer, 0, buffer.length);
        if (count > 0) {
            out.write(buffer, 0, count);
        }
    }
    out.close();
} catch (IOException e) {
    System.err.println("I/O problems: " + e);
    System.exit(-1);
}
```

*一般情况下可以根据Read的字节长度来判断是否写入到buffer中。

## Time-Domain and Frequency-Domain

时域和频域

这个字节数组中有记录在时域中的信号。时域（Time-Domain）信号表示的是信号随时间变化的幅度。

在十九世纪早期，Jean-Baptiste Joseph Fourier发现，任何时域中的信号都等价于一些简单的正弦信号（可能是无限个）的和，给定每个分量的正弦波都有特定的频率、振幅和相位。合在一起形成的原始时域信号的正弦波序列被称为了傅里叶级数。

换句话说，只要给出组成信号的正弦波所对应的频率、相位和振幅，就可以表示任何时域的信号。这种信号的表示方法被称为了频域（Frequency-Domain）。在某些方面，频域作为时域信号的指纹或者签名的一种类型，提供了动态信号的静态表示。

![https://dist.lyneee.com/blog/2021-03-18-3-time-domian.png](https://dist.lyneee.com/blog/2021-03-18-3-time-domian.png)

Time Domain & Frequency Domain

下面的GIF动画演示了1Hz方波的傅里叶级数，以及如何从正弦分量中产生（近似）方波。信号显示在上面的时域中，频域显示在下面的图像中。

![https://dist.lyneee.com/blog/2021-03-18-4-square-wave.gif](https://dist.lyneee.com/blog/2021-03-18-4-square-wave.gif)

1Hz Square Wave

在频域中分析信号可以极大的简化许多事情。在数字信号处理领域中，这种方法更加的简便，因为工程师们可以见就频谱，并确定哪些频率是存在的，哪些频率是缺失的。之后，你可以做过滤，增加或者减少一些频率，或者从给定的频率中识别确切的音调。

## The Discrete Fourier Transform

离散傅里叶变化

因此，我们需要找到一种方法，将我们的信号从时域转到频域中来。这里我们使用离散傅立叶变换$^2$（DFT，Discrete Fourier Transform）。 DFT是一种用于对离散（采样）信号执行傅立叶分析的数学方法。通过考虑这些正弦波是否具有相同的采样率，他将由一个函数的等距采样列表，转换成了一个按照频率排列的有限组合的正弦曲线的系数列表。

计算DFT的最流行的数值算法之一，就是快速傅立叶变换$^3$（FFT，Fast Fourier Transform）。到目前为止，最常用的FFT算法是Cooley-Tukey算法。这是一种分治算法，递归的将DFT分解成许多小的DFT进行计算。直接计算DFT会的时间复杂度是$O(n^2)$ ，使用分治算法将时间复杂度降低到了$O(n log n)$。

找到一个合适FFT库并不难，一下列举了几个：

- C - FFTW
- C++ - Eigen FFT
- Java - JTransform
- Python - NumPy
- Ruby - Ruby-FFTW3(Interface to FFTW)

下面是一个用Java编写的FFT函数的例子。（FFT采用复数作为输入。要理解复数和三角函数之间的关系，建议阅读欧拉公式$^4$）

```java
public static Complex[] fft(Complex[] x) {
    int N = x.length;
    
    // fft 偶数项
    Complex[] even = new Complex[N / 2];
    for (int k = 0; k < N / 2; k++) {
        even[k] = x[2 * k];
    }
    Complex[] q = fft(even);

    // fft 奇数项
    Complex[] odd = even; // reuse the array
    for (int k = 0; k < N / 2; k++) {
        odd[k] = x[2 * k + 1];
    }
    Complex[] r = fft(odd);

    // 合并
    Complex[] y = new Complex[N];
    for (int k = 0; k < N / 2; k++) {
        double kth = -2 * k * Math.PI / N;
        Complex wk = new Complex(Math.cos(kth), Math.sin(kth));
        y[k] = q[k].plus(wk.times(r[k]));
        y[k + N / 2] = q[k].minus(wk.times(r[k]));
    }
    return y;
}
```

下面是一个信号在FFT分析前后的样子：

![https://dist.lyneee.com/blog/2021-03-18-5-after-fft.png](https://dist.lyneee.com/blog/2021-03-18-5-after-fft.png)

before & after of FFT

## Music Recognition: Fingerprinting a Song

音乐识别：通过声纹识别一首歌

一个使用FFT的副作用是我们丢失了大量关于时间维度的信息。（尽管理论上是可以避免的，但是性能开销是巨大的。）对于一首三分钟的歌曲，我们看见了他们所有的频率，和它们的大小，但是我们不知道它们是什么时候出现在歌曲中的。但是这个关键信息（出现频率的时间点）才使得这首歌成为了这首歌。无论如何，我们都需要每个频率出现的时间点。

这就是为什么我需要引入滑动窗口（sliding window），或者说数据块（chunk of data），只转换一部分信息。每个快的大小可以用几种不同的方法来确定。例如，我们用立体声，16-bit采样在44.1kHz录制声音，这种声音的一秒钟是$44,100 samples*2 bytes * 2channels \approx176kB$。如果我们选择4kB作为块的大小，那么我们每秒将有44个数据块进行分析。这个粒度足够用来进行音频识别所需的详细分析。

现在回到编程，还是Java：

```java
byte audio [] = out.toByteArray()
int totalSize = audio.length
int sampledChunkSize = totalSize/chunkSize;
Complex[][] result = ComplexMatrix[sampledChunkSize][];

for(int j = 0;i < sampledChunkSize; j++) {
  Complex[chunkSize] complexArray;

  for(int i = 0; i < chunkSize; i++) {
		// 这里将样本块放入到虚部为0的复数中
    complexArray[i] = Complex(audio[(j*chunkSize)+i], 0);
  }

  result[j] = FFT.fft(complexArray);
}
```

在for循环内部，我们将时域数据（the samples，采样）放入到一个虚部为0的复数中。在外部循环中，我们迭代便利所有的块对每一个进行FFT分析。

一旦我们掌握了信号频率的组成信息，我们就可以开始形成歌曲的数字指纹。这是整个Shazam音频识别过程中最重要的那部分。这里的主要挑战是在已经捕获的频率海洋里去区分，哪些频率是最重要的。直观地，我们搜索具有最高幅度（峰值）的频率。

然而在一首歌中，最强的频率范围可能在低音$C-C_1$ （32.70Hz）和高音$C-C_8$（4186.01Hz）之间变化。这覆盖了一个巨大的范围。因此，预期一次性分析完整个频率的范围，我们可以选择几个较小的范围，这些范围是根据重要音乐成分的共同频选择的，并分别进行分析。例如，我们可以选择this guy$^5$为实现Shazam的算法所选择的时间间隔。这些间隔是30Hz-40Hz，40Hz-80Hz和80Hz-120Hz为低音(包括低音吉他) ，和120Hz-180Hz和180Hz-300Hz为中等和高音(包括人声和大多数其他乐器)。

现在在每个时间间隔内，我们可以简单地识别出峰值的频率。这些信息形成了歌曲这一数据块的签名，而这些签名成为了整首歌曲声纹的一部分。

```java
public final int[] RANGE = new int[] { 40, 80, 120, 180, 300 };

// 获取频率所在范围
public int getIndex(int freq) {
    int i = 0;
    while (RANGE[i] < freq)
        i++;
    return i;
}
    
// result is complex matrix obtained in previous step
// 结果是上一步所获取的复数矩阵（上一个代码块⬆️）
for (int t = 0; t < result.length; t++) {
    for (int freq = 40; freq < 300 ; freq++) {
        // 获取幅度
        double mag = Math.log(results[t][freq].abs() + 1);

        // 获取频率所在范围
        int index = getIndex(freq);

        // 保存最高的幅度和对应的频率:
        if (mag > highscores[t][index]) {
            points[t][index] = freq;
        }
    }
    
    // 形成hash tag
    long h = hash(points[t][0], points[t][1], points[t][2], points[t][3]);
}

private static final int FUZ_FACTOR = 2;

private long hash(long p1, long p2, long p3, long p4) {
    return (p4 - (p4 % FUZ_FACTOR)) * 100000000 + (p3 - (p3 % FUZ_FACTOR))
            * 100000 + (p2 - (p2 % FUZ_FACTOR)) * 100
            + (p1 - (p1 % FUZ_FACTOR));
}
```

需要注意的是，我们必须假定录音不是在完美的条件下完成的，因此我们必须包含模糊因子（fuzz factor）。模糊因子的分析在一个真实的系统中必须被重视到，程序应该有一个根据记录条件设置这个参数的选项。

为了方便搜索，这个签名成为了hash表中的key。对应的value则是出现这组频率在歌曲中的时间，以及歌曲的ID和歌手。下面是这些歌曲如何记录出现在数据库中的示例。

$$\begin{array}{|c|c|c|}\hline Hash\ Tag&Time\ in\ Seconds&Song\\\hline  30\  51\  99\  121\ 195&53.52&Song\ A\ by\ artist\ A\\\hline 33\  56\  92\  151\ 185&12.32&Song\ B\ by\ artist\ B\\\hline 39\  26\  89\  141\ 251&15.34&Song\ C\ by\ artist\ C\\\hline 32\  67\  100\  128\ 270&78.43&Song\ D\ by\ artist\ D\\\hline 30\  51\  99\  121\ 195&10.89&Song\ E\ by\ artist\ E\\\hline 34\  57\  95\  111\ 200&54.52&Song\ A\ by\ artist\ A\\\hline 34\  41\  93\  161\ 202&11.89&Song\ E\ by\ artist\ E\\\hline \end{array}$$

如果我们通过这个音乐识别过程运行一个完整的歌曲库，我们可以建立一个数据库，其中包括了， 每首歌曲的完整声纹。

## The Music Algorithm: Song Identification

音乐算法：歌曲识别

为了识别当前在俱乐部播放的歌，我们用手机录制了这首歌，然后像上面一样通过相同的声纹识别程序进行处理。接着，我们在数据库中搜索匹配的hash标签。

碰巧，许多hash标签将对应着多首歌的音乐标志符。例如，某一首歌的a的某一段听起来就像是歌曲e的某一段。当然，这并不奇怪，因为音乐家们总是会彼此“借鉴“一些licks和riffs，而且现在有的制作人会从别的歌曲中采样。每次我们匹配一个hash 标签的时候，可能匹配的数量就会变得更少，但是这个信息本省可能不会将匹配范围缩小到一首歌中。所以还有一件事情需要去检查我们的识别算法，就是时间。

我们在俱乐部中记录的歌曲样本可能来自于音乐的任何一部分，因此我们不能简单将匹配的Hash的时间戳和样本的时间戳进行配对。然而，使用多个hash进行匹配，我们可以分析出相对的时间，从而增加确定性。

例如你看上面的表格，你会发现hash标签为30 51 99 121 195分别对应的是Song A和Song E。如果一秒后，我们匹配到的hash是34 57 95 111 200，那更加的匹配Song A，在这种情况下，我们知道hash和时间差都匹配。

```java
// Class that represents specific moment in a song
// 代表了歌曲特定时刻的类
private class DataPoint {

    private int time;
    private int songId;

    public DataPoint(int songId, int time) {
        this.songId = songId;
        this.time = time;
    }
    
    public int getTime() {
        return time;
    }
    public int getSongId() {
        return songId;
    }
}
```

我们将$i_1$和$i_2$作为录制歌曲中的时刻，$j_1$和$j_2$作为数据库中的时刻。我们可说我们有两个时间差的匹配：

$$RecordedHash(i_1) = SongInDataBaseHash(j_1)  \\ \&\& \\ RecordedHash(i_2) = SongInDataBaseHash(j_2)$$

$$\\ abs(i_1-i_2)=abs(j_1-j_2)$$

这使得我们能够灵活的从头，中间，或者结尾录制歌曲。

最后，我们在俱乐部录制的歌曲在任何一个时刻都不可能跟我们在图书馆录制的一首歌的每一个对应时刻相匹配。录音会包含很多噪音，这些噪音会在匹配中引入一些错误。所以，我们没有试图去从匹配列表中移除，除了正确歌曲以外的所有歌曲，而是在最后，我们将所有匹配的歌曲按照可能性进行排序，而需要返回的则是排名列表中的第一个歌曲。

## From Top to Bottom

自顶向下

要回答这个问题，“How does Shazam work?”下面则是整个音乐识别和匹配过程的概述，自顶向下：

![https://dist.lyneee.com/blog/2021-03-23-6-top2bottom.png](https://dist.lyneee.com/blog/2021-03-23-6-top2bottom.png)

Processing Pipeline of Shazam

对于这种系统来说，数据库可能会变得非常庞大，因此使用一种可伸缩的数据库是非常重要的。它不需要关系，而且数据模型也很简单，所以使用一种No SQL数据库是一个很好的选择。

## How Does Shazam Work? Now You Know

现在你知道了

这种歌曲识别软件可以用来寻找歌曲之间的相似性。现在你已经了解了 Shazam 的工作原理，您可以看到除了播放出租车收音机播放的怀旧歌曲 Shazaming 之外，它还具有其他应用。例如，它可以帮助找到音乐中的抄袭，或者找出谁是布鲁斯、爵士、摇滚、流行音乐或任何其他流派先驱的最初灵感来源。

也许一个好的实验是用巴赫、贝多芬、维瓦尔第、瓦格纳、肖邦和莫扎特的古典音乐填充歌曲样本数据库，试着找出歌曲之间的相似之处。你可能会认为连鲍勃 · 迪伦、猫王和罗伯特 · 约翰逊都是剽窃者！

但是我们仍然不能给他们定罪。

因为音乐只是我们在头脑中听到、记忆和不断重复的一个波，它在脑海中进化和演变，直到我们录制了它，然后传递给下一个伟大的音乐天才。

译完这篇，其实已经能够解决我在工作场景中遇到的问题，对几种录音进行，捕获，转换到频域，计算声纹，然后匹配两个时间差。我非常认同最后一段话，这段本不应该出现在技术文章中的话。美好的事情应该被传递下一个人脑海，给普通人带来快乐，给天才们创造和发挥的灵感。技术也是，就像大家所信奉的开源精神一样，带给这个世界以改变。

作为收尾，我会去实现一个Go的Shazam算法，可能已经有了。至此，翻译计划的第三篇已经完成，计划也迎来了一位新的小伙伴，下一篇文章将是由他进行选题，然后共同翻译。

## 参考

[1]奈奎斯特-香农定理： [https://en.wikipedia.org/wiki/Nyquist–Shannon_sampling_theorem](https://en.wikipedia.org/wiki/Nyquist%E2%80%93Shannon_sampling_theorem)

[2]离散傅立叶变换： [https://en.wikipedia.org/wiki/Discrete_Fourier_transform](https://en.wikipedia.org/wiki/Discrete_Fourier_transform)

[3]Cooley-Tukey算法:[https://en.wikipedia.org/wiki/Cooley–Tukey_FFT_algorithm](https://en.wikipedia.org/wiki/Cooley%E2%80%93Tukey_FFT_algorithm)

[4]欧拉公式定义复数下三角函数

$\sin z = \frac{e^{iz}-e^{-iz}}{2i}$

$\cos z = \frac{e^{iz}+e^{-iz}}{2}$

$e^{ix}=\cos x +i \sin x$  

$e^{-ix}=\cos x -i \sin x$

[5]用Java实现Shazam 算法： [https://www.royvanrijn.com/blog/2010/06/creating-shazam-in-java/](https://www.royvanrijn.com/blog/2010/06/creating-shazam-in-java/)