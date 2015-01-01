---
layout: post
title: The MIFARE Hack ———— 安卓版
description: 本文将分为理论和实践两个部分来描述一个小白破解热水卡的蛋疼之旅。
category: blog
---

> 本文仅用于技术交流，请勿闷声作大死 0x0

## 1、热水卡的前世今生

首先我们要知道一种东西叫无线射频识别，即RFID。RFID card当然就是使用无线射频的卡（废话），一般说来有两种RFID card，主动式的和被动式的。主动式的有自己的内置电源，被动式的则要依靠读卡器提供的电量进行驱动。

最普遍使用的一种RFID card中是MIFARE Classic chip，现在广泛使用的交通卡、热水卡、饭卡等一大坨就是典型的MIFARE Classic，据说占了75%的市场份额，还有一大堆乱七八糟的协议，不过幸好这不是我们的重点。 （现在我们只需要关心手上的热水卡。

热水卡是一种被动式的RFID card，结构相当简单粗暴，掰弯（欸）之后会看到一个大的线圈以及一个小芯片。线圈负责为芯片供能，芯片负责存储以及预算，至于电是怎么来的，可以去问高中物理老师 0w0

![MIFARE Classic](http://img.tradekey.com/p-2558210-20090703015402/mifare-classic-card.gif)
---

## 2、MIFARE Classic的人体结构 （大雾

为了黑出姿势黑出水平，当然有必要来看下人体结构 （正脸

现在比较常见的MIFARE classic有三种容量：1k、2k、4k。接下来我们将认识到一个真理，“大，即是正义”。

拿1k的卡来说，一般这种卡直接被称为M1卡。M1卡中有16个分区（sector），每个分区有4个块（block），每个块有16个byte的数据。

其中，编号为0的分区的第0块中的数据一般会被固化，存有卡片的id以及厂家信息。每个分区的第3块为trailer，存有俩分别为6byte的key（KEY A & KEY B）和一个4byte的读写控制模块。KEY A一般是不可读的，但是KEY B可以设置为可读，用来存贮数据。

因为网上关于M1卡的技术资料比较少，表示实在是不知道KEY A & KEY B是怎么分工的，但是据说可以设置一个控制本扇区的读权限，一个控制本扇区的写权限。（当然没有KEY B的时候就KEY A既负责读也负责写。

M1的权限验证过程说起来还是很简单的~，首先卡会发给读卡器一个随机数，读卡器利用自身的加密算法加密这个随机数，然后返回给卡。卡用自身带的密钥进行同样的运算，得出结果与读卡器的返回值比较，来判断读卡器权限。

![Miface](../images/hot_water_card/Mifare.jpg)
---

## 3、破解方法

现在我们的任务只是破解掉第3块的密钥就可以读写某个分区。

通过以上的说明可以发现，卡本身是没有记录验证次数的。

所以破解的方法大致有以下几种：

### 1、暴力破解

暴力破解是破解工作永远的话题，只要你拥有庞大的计算资源，管你什么密码都能破解。而且，在CRYPTO1算法的细节没有被泄露之前，最有效的方法就是暴破了。还有一个很重要的原因就是，M1卡是被动卡，需要读卡器为它提供能量，一旦读卡器切断了电源，卡中的临时数据就会丢失，这样就没有办法记录下攻击者究竟输错了多少次密码，卡永远不会因为密码输入错误太多而被锁定，只要攻击者有时间慢慢跟它耗，密码肯定会出来的。

比较常见的密钥有：

FFFFFFFFFFFF

A0A1A2A3A4A5

D3F7D3F7D3F7

000000000000

A0B0C0D0E0F0

A1B1C1D1E1F1

B0B1B2B3B4B5

4D3A99C351DD

1A982C7E459A

AABBCCDDEEFF

B5FF67CBA951

714C5C886E97

587EE5F9350F

A0478CC39091

533CB6C723F6

24020000DBFD

000012ED12ED

8FD0A4F256E9

EE9BD361B01B

### 2、重放攻击

重放攻击是基于M1卡的PRNG算法漏洞实现的，当卡接近读卡器获得能量的时候，就会开始生成随机数序列，但这有一个问题，因为卡是被动式卡，本身自己不带电源，所以断电后数据没办法保存，这时基于LSRF的PRNG算法缺陷就出来了，每次断电后再重新接入电，卡就会生成一摸一样的随机数序列，所以我们就有可能把这个序列计算出来，所以只有我们控制好时间，就能够知道在获得能量后的某一刻时间的随机数是多少，然后进行重放攻击，就有可能篡改正常的数据。如果卡的所有权在我们手上的时候，我们甚至不需要浪费太多的时间就可以实现。

### 3、密钥流窃听

利用神器proxmark 3可以嗅探到全部扇区都加密的M1卡，在卡和已经授权的读卡器交换数据的时候进行窃听，就能把tag数据读取出来，利用XOR算key工具就可以把扇区的密钥计算出来，这也是PRNG算法的漏洞所导致的。

### 4、验证漏洞

验证漏洞是目前使用最多的M1破解手段，在读卡器尝试去读取一个扇区时，卡会首先发一个随机数给读卡器，读卡器接到随机数之后利用自身的算法加密这个随机数再反馈回给卡，卡再用自己的算法计算一次，发现结果一致的话就认为读卡器是授权了的，然后就用开始自己的算法加密会话并跟读卡器进行传送数据。这时候问题就来了，当我们再次尝试去访问另一个扇区，卡片又会重复刚才那几个步骤，但此时卡跟读卡器之间的数据交换已经是被算法加密了的，而这个算法又是由扇区的密钥决定的，所以密钥就被泄露出来了。因此验证漏洞要求我们至少知道一个扇区的密钥，但目前大部分的扇区都没有全部加密，所以很容易就会被破解。

### 以及其他

---

## 4、安卓表示有话说 =w=

现在大部分的安卓手机上都有NFC功能，NFC是由RFID演变来的，所以向下兼容了RFID。

所以**安卓手机是可以作为读卡器用的**

在android SDK中，已经提供了nfc相应的库类，我们只需要用就行啦 0w0

### 1、权限声明

首先要在AndroidManifest.xml文件中声明权限

    <uses-permission android:name="android.permission.NFC" />

并且设置支持的最小安卓版本

    <uses-sdk android:minSdkVersion="14" android:targetSdkVersion="14" />

### 2、NFC TAG发布

因为NFC感应是被动的，就是说并不需要用户控制连接操作（像蓝牙就得用户操作连接），所以我们需要配置TAG发布系统来获取系统中处理此TAG信号的Activity。昂，Tag 代表一个被动式Tag对象，可以代表一个标签，卡片等。当Android设备检测到一个Tag时，会创建一个Tag对象，将其放在Intent对象，然后发送到相应的Activity。

像这种事情当然需要Intent filter，TAG分发系统中定义了三种intent。按优先级从高到低分别为：

> NDEF_DISCOVERED, TECH_DISCOVERED, TAG_DISCOVERED

我们使用的intent-filter的Action类型为TECH_DISCOVERED从而可以处理所有类型为ACTION_TECH_DISCOVERED并且使用的技术为nfc_tech_filter.xml文件中定义的类型的TAG。

    AndroidManifest.xml:
    <intent-filter>   
    	<action android:name="android.nfc.action.TECH_DISCOVERED" />   
    </intent-filter>   
    <meta-data   
    	android:name="android.nfc.action.TECH_DISCOVERED"   
    	android:resource="@xml/nfc_tech_filter" />

    nfc_tech_filter.xml:
    <resources xmlns:xliff="urn:oasis:names:tc:xliff:document:1.2"> 
    <tech-list> 
       <tech>android.nfc.tech.MifareClassic</tech> 
    </tech-list> 
    </resources>

![tag](../images/hot_water_card/tag.jpg)

### 3、在Activity中处理TAG

首先需要在onResume()中处理所收到的TAG，这里直接分发给readFromTag进行处理。

    public void onResume()
    {
    	super.onResume();
    	if (NfcAdapter.ACTION_TAG_DISCOVERED.equals(getIntent().getAction()))
    	{
    		readFromTag(getIntent());
    	}
    }

    private boolean readFromTag(Intent intent)
    {
    	mfc = MifareClassic.get(tagFromIntent);
    	boolean auth = false;
    	try
    	{
    		String metaInfo = "";
    		MifareClassic mfc.connect();
    		int type = mfc.getType();
        	int sectorCount = mfc.getSectorCount();		//获取分区数量
        	String typeS = "";
        	switch (type)
        	{
        	case MifareClassic.TYPE_CLASSIC:
        		typeS = "TYPE_CLASSIC";
        		break;
        	case MifareClassic.TYPE_PLUS:
        		typeS = "TYPE_PLUS";
        		break;
        	case MifareClassic.TYPE_PRO:
        		typeS = "TYPE_PRO";
        		break;
        	case MifareClassic.TYPE_UNKNOWN:
        		typeS = "TYPE_UNKNOWN";
        		break;
            }
        	metaInfo += "卡片类型：" + typeS + "\n共" + sectorCount + "个扇区\n共"
					+ mfc.getBlockCount() + "个块\n存储空间: " + mfc.getSize() + "B\n";
        	System.out.println("metaInfo:" + metaInfo);

            for (int i = 0; i < sectorCount; i++)
            {
            	auth = mfc.authenticateSectorWithKeyA(i, MifareClassic.KEY_DEFAULT);	//通过key A获取权限
            	int bCount;
            	int bIndex;
            	if (auth)
            	{
            		metaInfo += "Sector" + i + "验证成功\n";
            		bCount = mfc.getBlockCountInSector(i);      //获取块数
            		bIndex = mfc.sectorToBlock(i);              //获取i块扇区第一块
            		for (int j = 0; j < bCount; j++)
            		{
            			byte[] data = mfc.readBlock(bIndex);
            			metaInfo += "Block " + bIndex + " : "
            				+ bytesToHexString(data) + "\n";	//转换为十六进制
            			bIndex++;
            		}
            	}
            	else
            	{
            		metaInfo += "Sector" + i + "验证失败";
            	}
    		}
            System.out.println("metaInfo:" + metaInfo);
        }
        catch (IOException e)
        {
            e.printStackTrace();
        }
        finally
        {
            try
            {
                mfc.close();
            }
            catch (IOException e)
            {
                e.printStackTrace();
            }
        }
        return false;
    }

同理，写入的时候只需要调用mfc.writeBlock(1, bytes[]);

缺省的KEY值一般是全FF或者0.

执行效果~

![run](../images/hot_water_card/run.png)
---

## 5、最后

嘛，有同学要说了。。。博主最后也没有给出破解这不是坑人嘛！

博主正脸表示说，我是用热水卡来存密码的，或者把热水卡贴在寝室门上大家扫一下就知道在下在不在
寝室。（…………）

写到这么多已经不想写了。。。

不过还是强烈建议大家，不！要！破！解！

因为饭卡公交卡什么的都是联网的，就算水卡也是有保险措施的，一来不道德，二来不安全。只有在下这种闲的蛋疼的人才会去折腾 Orz
