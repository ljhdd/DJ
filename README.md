# DJ
* This project is developed using Tuya SDK, which enables you to quickly develop branded apps connecting and controlling smart scenarios of many devices.For more information, please check Tuya Developer Website.
# 一、方案名称
基于stm32单片机的宠物自动喂食器
# 二、方案市场分析与前景
* 由于疫情的突发,许多人突发性的被隔离或者是由于其他原因回不了家,家里空荡荡的,回不了家对于家里面有爱宠的"同志们"来说可是一种痛苦,只能活生生的看着爱宠们靠吃猫砂甚至塑料袋来度过,有的还甚至被饿死。而且当今社会中，人们的工作与学习是十分的繁忙，当人们外出时间比较长时对宠物的食物和水的供给就出出现了比较大的问题。特别是当人们出差在外或者是，旅游度假时，经常不能及时的对家中的宠物给予很好地照顾，宠物的饮食常常成为困扰人们主要问题。本装置就是针对此问题而设计的一种装置。他对家居智能化起到了积极地作用，在提高人们生活效率与乐趣的同时，也对人们的出行减少了后顾之忧。
  毫无疑问，智能宠物喂食器不仅仅是一款智能设备，它更是连接主人与爱宠的纽带，而功能强大的APP将万千家庭集结到一起，相比单纯的产品，这种基于兴趣、圈子的应用与服务更能打动我们，未来智能宠物喂食器的APP还有很大优化的空间，而且还会基于游戏娱乐的基础之上进一步开拓空间。
 # 三、方案使用场所及使用人群
 * 使用场所一般是人们所居住的室内，以及一些宠物饲养基地，面向的人群有饲养宠物者其中家中有宠物由于工作原因经常性出差人群是最主要的，还有就是一些大型宠物饲养基地。
 # 四、项目具体方案
 * 从物联网技术在宠物喂养上的应用出发，研究了一种基于物联网技术的智能宠物投喂器，此投喂器的功能包括对宠物实时或者定时喂食、利用录音和语音提醒宠物进食、残料自动清理等，这些功能利用stm32单片机对系统中的各个特殊功能进行控制，如通过H桥直流电机驱动功能板，直流供电电源板对步进电机下料控制、语言操控模块控制、电动推杆行程控制等，利用MATLAB技术实现了机器学习与对系统稳定性的测试。通过以上功能的结合便能较好的完成对宠物进行远程喂食的功能，实现对宠物的智能投喂，以及通过光电传感器（光电传感器由敏感元件、转换元件、基本电路三部分组成，其主要原理是光电效应。）来监测宠物的实时状况，现在食物平台放置少许食物，当宠物在盛放食物的停留2分钟以上即代表宠物正在进食，该投喂器将继续投喂适量食物，当宠物在平台上待了一段时间后检测器监测到宠物离开以后，投喂器自动将平台清空，并重新放置少许食物，其中传感器采集到的数据通过wifi上传到涂鸦iot平台并在涂鸦app进行显示,主人可在涂鸦app上实现远程投喂,监测喂食器的各种状态。其中投喂器的储存仓里具有保持干燥的装置。
 # 五、方案实施流程
 ## 准备所需材料
 * 语音涂鸦三明治开发套件
 * stm32F103最小版
 * 光电传感器模块
 * 涂鸦iot平台
 * 涂鸦app
 * 步进电机
 * 铝型材及符合木板胶枪胶棒各种元器件
 ## 实施方案
 * 光电传感器模块通过串口与stm32单片机连接，在通过单片机将采集到的的各种数据通过WiFi模块进行上报。
 *   串口程序
  ```c
  uint8_t reData[9],reDndex=0,reflag=0;

void USART1_IRQHandler(void)
{
	if(USART_GetITStatus(USART1, USART_IT_RXNE) != RESET)  //接收中断
	{
		USART_ClearITPendingBit(USART1,USART_IT_RXNE);         //清除接收标志
		uint8_t res=USART_ReceiveData(USART1);//(USART1->DR);	//读取接收到的数据
		reData[reDndex++] = res;
		if(reDndex==9) 
		{
			 reDndex=0;
			 reflag = 1;
		}
	}
}
```
*  WiFi模块程序
```c
/*
name:创建的wifi名字
key:创建wifi的密码
wifi默认ip为192.168.1.1
*/
void ESP8266_TCP_AP(char *name,char *key)
{
	ESP8266_Init();
	int8_t sta = -1;
	char *p = NULL;
	p=(char *)malloc(255);	
	ESP8266_QuitTrans();
	sta = ESP8266_SendCmd("AT+RESTORE\r\n", "ready\r\n",4000);
	sta = ESP8266_SendCmd("AT+CWMODE=2\r\n", "OK\r\n",1000);
	sta = ESP8266_SendCmd("AT+RST\r\n", "ready\r\n",5000);
	sprintf((char *)p,(char *)("AT+CWSAP=\"%s\",\"%s\",1,3\r\n"),name,key);
	sta = ESP8266_SendCmd(p, "OK\r\n",1000);
	sprintf((char *)p,(char *)("AT+CIPAP=\"%s\"\r\n"),AP_IP);
	sta = ESP8266_SendCmd(p, "OK",5000);			// 模式IP
	sta = ESP8266_SendCmd("AT+CIPMUX=1\r\n", "OK\r\n",1000);
	sprintf((char *)p,(char *)("AT+CIPSERVER=1,%s\r\n"),AP_PORT);
	sta = ESP8266_SendCmd(p, "OK\r\n",5000);					// AP模式的端口
	if(sta==1)		
	{
		wifi_struct.index = 0;
		memset(wifi_struct.buff, 0, WIFI_DATA_LEN);    	//清空接收区
		wifi_struct.mode = TCP_AP;
		wifi_struct.con_num = 0;
	}		
	free(p);
	p=NULL;
}

// 连接指定IP和密码，创建服务器，端口在头文件
char* ESP8266_TCP_STA(char *name,char *key)
{
	ESP8266_Init();
	int8_t sta=-1,index=0;
	char *p = NULL;
	p=(char *)malloc(255);	
	char *ip = NULL;
	ip=(char *)malloc(15);
	
	ESP8266_QuitTrans();
	sta = ESP8266_SendCmd("AT+RESTORE\r\n", "ready\r\n",4000);
	sta = ESP8266_SendCmd("AT+CWMODE=1\r\n", "OK\r\n",1000);
	sta = ESP8266_SendCmd("AT+RST\r\n", "ready\r\n",5000);
	sprintf((char *)p,(char *)("AT+CWJAP=\"%s\",\"%s\"\r\n"),name,key);
	sta = ESP8266_SendCmd(p, "OK\r\n",10000);		
	sta = ESP8266_SendCmd("AT+CIPMUX=1\r\n", "OK\r\n",1000);// 模式IP
	sprintf((char *)p,(char *)("AT+CIPSERVER=1,%s\r\n"),AP_PORT);
	sta = ESP8266_SendCmd(p, "OK\r\n",5000);					// AP模式的端口
	sta = ESP8266_SendCmd("AT+CIFSR\r\n", "OK\r\n",1000);// 模式IP
	if(sta==1)
	{
		int i = WIFIStrstr((char *)wifi_struct.buff, "STAIP");
		while(1)
		{
			if(wifi_struct.buff[i+6]!='\r')
			{
				ip[index] = wifi_struct.buff[i+6];
				index++;
				i++;
			}
			else break;
		}
	}
	if(sta==1)		
	{
		wifi_struct.index = 0;
		memset(wifi_struct.buff, 0, WIFI_DATA_LEN);    	//清空接收区
		wifi_struct.mode = TCP_STA;
		wifi_struct.con_num = 0;
	}		
	free(p);
	p=NULL;
	return ip;
}


// 连接指定IP和密码，创建服务器，端口在头文件
char* ESP8266_TCP_CLIENT(char *name,char *key,char *ip,uint32_t port)
{
	ESP8266_Init();
	int8_t sta=-1,index=0;
	char *p = NULL;
	p=(char *)malloc(255);	
	char *rip = NULL;
	rip=(char *)malloc(15);
	
	ESP8266_QuitTrans();
	sta = ESP8266_SendCmd("AT+RESTORE\r\n", "ready\r\n",4000);
	sta = ESP8266_SendCmd("AT+CWMODE=1\r\n", "OK\r\n",1000);
	sta = ESP8266_SendCmd("AT+RST\r\n", "ready\r\n",5000);
	sprintf((char *)p,(char *)("AT+CWJAP=\"%s\",\"%s\"\r\n"),name,key);
	sta = ESP8266_SendCmd(p, "OK\r\n",10000);		
	sta = ESP8266_SendCmd("AT+CIFSR\r\n", "OK\r\n",1000);// 模式IP
	if(sta==1)
	{
		int i = WIFIStrstr((char *)wifi_struct.buff, "STAIP");
		while(1)
		{
			if(wifi_struct.buff[i+6]!='\r')
			{
				rip[index] = wifi_struct.buff[i+6];
				index++;
				i++;
			}
			else break;
		}
	}
	
	sprintf((char *)p,(char *)("AT+CIPSTART=\"TCP\",\"%s\",%d\r\n"),ip, port);    // 断电不重新连接
	// sprintf((char *)p,(char *)("AT+SAVETRANSLINK=1,\"%s\",%s\r\n,\"TCP\""),ip, port);   // 断电重新连接			AT+SAVETRANSLINK=0    关闭上电重联
	sta = ESP8266_SendCmd(p, "OK\r\n",5000);					// AP模式的端口
	
	sta = ESP8266_SendCmd("AT+CIPMODE=1\r\n", "OK\r\n",1000);					// 开启透传模式
	sta = ESP8266_SendCmd("AT+CIPSEND\r\n", ">",1000);					// 开启透传模式
	
	free(p);
	p=NULL;
	
	if(sta==1)		
	{
		wifi_struct.index = 0;
		memset(wifi_struct.buff, 0, WIFI_DATA_LEN);    	//清空接收区
		wifi_struct.mode = TCP_CLIENT;
		wifi_struct.con_num = 0;
	}		
	else{
		rip[0] = 0;
		return rip;
	}

	return rip;
}
```
* 在涂鸦iot平台上创建产品并配置设备面板
* 下载涂鸦app,使用涂鸦串口调试助手调试模组并进行模组的配网,检查是否在app上进行相应的操作
* 开发板调试完成后开始进行产品的模拟组装
* 用木板根据设计好的图纸切割和步进电机及电动推杆将喂食器组装好
* 进行产品演示
# 六、开发计划
* 3.5-3.9 准备材料,购买相应的开发板。
* 3.9-3.11 研究语音模块。
* 3.11-3.15 学习涂鸦iot模块,移植mcu到stm32单片机上。
* 3.15-3.18 结合所有的模块，使其系统稳定。
* 3.18-3.22 开始动手实践,对喂食器进行设计及组装,并进行调试。
* 3.23-3.24 录制视频,准备ppt进行讲解。
# 七预计效果
* ![百度](https://image.baidu.com/search/detail?ct=503316480&z=0&ipn=d&word=%E5%AE%A0%E7%89%A9%E8%87%AA%E5%8A%A8%E5%96%82%E9%A3%9F%E5%99%A8%E5%9B%BE%E7%89%87&hs=2&pn=3&spn=0&di=60940&pi=0&rn=1&tn=baiduimagedetail&is=0%2C0&ie=utf-8&oe=)
