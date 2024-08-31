马里奥像素时钟

一.硬件
ESP32
老五家的128*96的HUB75E接口像素屏（需要拆焊原有芯片）

二.拆焊屏幕原有芯片
固定用的UV胶可以轻松用一字螺丝刀拆下。

原有芯片采用合宙air720sd的4G芯片+GPS模块。在开发难度高，所以替换成为esp32。

模块拆卸方法：
使用粗铜线+大量锡 焊接到原有芯片的邮票孔上，为了导热
因为原有芯片下面有个片铁氟龙贴，有间距。
用一把螺丝刀，塞在间距中。
拆焊之前，在屏幕下面放一个导热硅脂，或导热铜片，防止led灯损坏。
拆焊时，两头分别用电烙铁加热到焊锡熔化。

三.连接ESP32开发板引脚到屏幕
屏幕使用hub75e 32扫的64*96的2块屏。
以为使用32扫 所以E_PIN需要引出
#define R1_PIN 25
#define G1_PIN 26
#define B1_PIN 27
#define R2_PIN 14
#define G2_PIN 12
#define B2_PIN 13
#define A_PIN 23
#define B_PIN 19
#define C_PIN 5
#define D_PIN 17
#define E_PIN 32 
#define LAT_PIN 4
#define OE_PIN 15
#define CLK_PIN 16

四.程序部分
// 项目地址：https://github.com/jnthas/clockwise
git clone项目
使用platformio烧录调试

五.修改HUB75E驱动库
// 接口库地址：https://github.com/mrfaptastic/ESP32-HUB75-MatrixPanel-I2S-DMA#2-wiring-esp32-with-the-led-matrix-panel
因为屏幕硬件比较奇怪，原有驱动库需要修改一下代码

修改这个文件ESP32-VirtualMatrixPanel-I2S-DMA.h
找到return coords;这行代码
在它上面加入下面代码
	if(coords.x>=0 and coords.x<96 and coords.y>=0 and coords.y<32){
		coords.x = coords.x + 96;
	}else if(coords.x>=0 and coords.x<96 and coords.y>=32 and coords.y<64){
		coords.y = coords.y - 32;
	}else if(coords.x>=96 and coords.x<192 and coords.y>=0 and coords.y<32){
		coords.y = coords.y + 32;
	}else if(coords.x>=96 and coords.x<192 and coords.y>=32 and coords.y<64){
		coords.x = coords.x - 96;
	}

六.修改时钟主题适配屏幕分辨率
1.cw-gfx-engine/Game.h
const int DISPLAY_WIDTH = 96;
const int DISPLAY_HEIGHT = 128;

2.cw-commons/StatusController.h
	void clockwiseLogo()
	{
		// Locator::getDisplay()->drawRGBBitmap(1, 1, epd_bitmap_clockwise64, 63, 21);
		Locator::getDisplay()->drawRGBBitmap(16, 1, epd_bitmap_clockwise64, 63, 21);
	}

	void wifiConnecting()
	{
		// Locator::getDisplay()->fillRect(0, 24, 64, 52, 0);
		// Locator::getDisplay()->drawBitmap(16, 24, CW_STATUS_WIFI, 32, 32, 0x2459);
		// printCenter("Connecting WiFi", 61);
		Locator::getDisplay()->fillRect(0, 32, 96, 52, 0);
		Locator::getDisplay()->drawBitmap(32, 32, CW_STATUS_WIFI, 32, 32, 0x2459);
		printCenter("Connecting WiFi", 88);
	}

	void wifiConnectionFailed(const char *msg)
	{
		// Locator::getDisplay()->fillRect(0, 24, 64, 52, 0);
		// Locator::getDisplay()->drawBitmap(16, 24, CW_STATUS_WIFI, 32, 32, 0xFA28);
		// printCenter(msg, 61);
		Locator::getDisplay()->fillRect(0, 32, 96, 52, 0);
		Locator::getDisplay()->drawBitmap(32, 32, CW_STATUS_WIFI, 32, 32, 0xFA28);
		printCenter(msg, 88);
	}

	void ntpConnecting()
	{
		// Locator::getDisplay()->fillRect(0, 24, 64, 52, 0);
		// Locator::getDisplay()->drawBitmap(16, 24, CW_STATUS_NTP, 32, 32, 0xBCBF);
		// printCenter("NTP Server", 61);
		Locator::getDisplay()->fillRect(0, 32, 96, 52, 0);
		Locator::getDisplay()->drawBitmap(32, 32, CW_STATUS_NTP, 32, 32, 0xBCBF);
		printCenter("NTP Server", 88);
	}

	void printCenter(const char *buf, int y)
	{
		int16_t x1, y1;
		uint16_t w, h;
		Locator::getDisplay()->setFont(&Picopixel);
		Locator::getDisplay()->getTextBounds(buf, 0, y, &x1, &y1, &w, &h);
		// Locator::getDisplay()->setCursor(32 - (w / 2), y);
		Locator::getDisplay()->setCursor(48 - (w / 2), y);
		Locator::getDisplay()->setTextColor(0xffff);
		Locator::getDisplay()->print(buf);
	}







// 画图板：https://editor.clockwise.page/
// 在线图片转位图：https://javl.github.io/image2cpp/
