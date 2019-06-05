>在计网课设中需要通过raw socket的方式实现一个ping程序，而raw socket是一种原始套接字可以接收本机网卡上的数据帧或者数据包，但是java仅仅支持到应用层，是无法直接操作底层的，如果想访问ICMP可以下载jpcap这个类库

## 一，JPCAP

>Jpcap是一个可以监控当前网络情况的中间件，弥补了java对网络层以下的控制

### jpcap下载与安装

**1.下载**

地址：[网盘地址](https://pan.baidu.com/s/1yaOV_joHBG1akJckSNd-vA)   
提取码：0ks5

**安装**

Jpcap运行需要依赖winCap和Jpcap的dll动态库和Jpcap.jar包

1.双击WinPacap_4_1_2.exe安装

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190605172051322.png)

2.Jpcap.dll复制到java jdk的安装目录下的bin目录里

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190605172236219.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

3.eclipse导入jpcap.jar包,导入成功后显示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190605172325508.png)

## 二，java通过JPCAP实现简易ping程序

```java
import java.net.InetAddress;
import java.util.ArrayList;
import java.util.GregorianCalendar;
import java.util.List;
import java.util.Scanner;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

import jpcap.JpcapCaptor;
import jpcap.JpcapSender;
import jpcap.NetworkInterface;
import jpcap.packet.EthernetPacket;
import jpcap.packet.ICMPPacket;
import jpcap.packet.IPPacket;

public class JPing {

	private NetworkInterface[] devices = JpcapCaptor.getDeviceList();
	private JpcapSender sender;
	private JpcapCaptor jpcap;
	private ICMPPacket icmpPacket;
	private List<String> listResult = new ArrayList<String>();

	/**
	 * 组织ICMP报文发送，并开启线程接收报文
	 * 
	 * @param ip
	 */
	public void ping(String ip) {
		try {
			jpcap = JpcapCaptor.openDevice(devices[0], 200, false, 20);
			sender = jpcap.getJpcapSenderInstance();
			// 拦截器，只接受ICMP报文
			jpcap.setFilter("icmp", true);
			icmpPacket = new ICMPPacket();
			// 发送回显请求报文
			icmpPacket.type = ICMPPacket.ICMP_ECHO;

			icmpPacket.setIPv4Parameter(0, false, false, false, 0, false, false, false, 0, 1010101, 100,
					IPPacket.IPPROTO_ICMP, devices[0].addresses[1].address, InetAddress.getByName(ip));

			// icmp数据报（发送包）数据（任意）
			icmpPacket.data = "abcdefghijklmnopqrstuvwxyzabcdef".getBytes();
			EthernetPacket ethernetPacket = new EthernetPacket();
			ethernetPacket.frametype = EthernetPacket.ETHERTYPE_IP;
			ethernetPacket.src_mac = devices[0].mac_address;

			// 广播地址
			ethernetPacket.dst_mac = new byte[] { (byte) 0xff, (byte) 0xff, (byte) 0xff, (byte) 0xff, (byte) 0xff,
					(byte) 0xff };
			icmpPacket.datalink = ethernetPacket;
			listResult.add("Pinging " + icmpPacket.dst_ip + " with " + icmpPacket.data.length + " bytes of data: ");

			// 接收数据报
			startCapThread(jpcap);

			// 发送数据报
			for (int i = 0; i < 4; i++) {
				icmpPacket.sec = 0;
				// icmpPacket.usec = System.currentTimeMillis();
				icmpPacket.usec = new GregorianCalendar().getTimeInMillis();// 记录发送时间
				icmpPacket.seq = (short) (1000 + i);
				icmpPacket.id = (short) (999 + i);
				sender.sendPacket(icmpPacket);
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

	/**
	 * 接收ICMP报文
	 * 
	 * @param jpcap
	 */
	public void getIcmpPacket(JpcapCaptor jpcapCaptor) {
		try {
			long tmp = 0;
			String tmpStr = null;
			ICMPPacket rp;
			while ((rp = (ICMPPacket) jpcapCaptor.getPacket()) != null) {
				System.out.println("接收到的icmp数据报------》" + rp);
				// 正确接收到ICMP数据包
				if (rp != null) {
					// 计算发送与接收的时间差
					tmp = (rp.sec * 1000 + rp.usec / 1000 - icmpPacket.sec * 1000 - icmpPacket.usec);
					if (tmp <= 0) {
						tmpStr = " < 1 ms ";
					} else {
						tmpStr = "= " + tmp + " ms ";
					}
					// ICMP数据报信息
					System.out.println("Reply from " + rp.src_ip.getHostAddress() + ": bytes = " + rp.data.length
							+ " time " + tmpStr + "TTL = " + rp.hop_limit);
					// ICMP数据报接收信息结果集
					listResult.add("Reply from " + rp.src_ip.getHostAddress() + ": bytes = " + rp.data.length + " time "
							+ tmpStr + "TTL = " + rp.hop_limit);

				} else {
					System.out.println("request timeout");
				}
			}
		} catch (Exception e) {
			e.printStackTrace();
		}

	}

	/**
	 * 接收ICMP报文
	 * 
	 * @param jpcap
	 */
	public void startCapThread(final JpcapCaptor jpcap) {

		Runnable runner = new Runnable() {
			public void run() {
				getIcmpPacket(jpcap);
			}
		};
		new Thread(runner).start();
	}

	/**
	 * 判断是否是正确的IP地址
	 */
	public static boolean isboolIp(String ipAddress) {
		String ip = "([1-9]|[1-9]\\d|1\\d{2}|2[0-4]\\d|25[0-5])(\\.(\\d|[1-9]\\d|1\\d{2}|2[0-4]\\d|25[0-5])){3}";
		Pattern pattern = Pattern.compile(ip);
		Matcher matcher = pattern.matcher(ipAddress);
		return matcher.matches();
	}

	/**
	 * 命令：
	 * 
	 * ping 一次：ping xxxx
	 * 
	 * ping 无限次：ping -t xxxx
	 */
	public static void main(String[] args) {

		JPing jPing = new JPing();
		Scanner scanner = new Scanner(System.in);
		System.out.println("请输入指令：");
		String command = scanner.nextLine();
		String[] str = command.split(" ");
		if ("-t".equals(str[1])) {
			if (isboolIp(str[2])) {
				while (true) {
					jPing.ping(str[2]);
				}
			} else {
				System.out.println("ip输入有误");
			}
		} else if (isboolIp(str[1])) {
			jPing.ping(str[1]);
		} else {
			System.out.println("输入有误");
		}
	}
}

```

结果图：

执行 ping 127.0.0.1

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190605173411405.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

执行 ping -t 12.0.0.1 

![在这里插入图片描述](https://img-blog.csdnimg.cn/201906051736042.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)




