# 1.app icon text

ico名称改成“pm2.5\_dp”

```c
//main_menu.c
66    {ID_ICON, 0, 0, ICON_HEIGHT, ICON_WIDTH, (uint16_t *)acChiLun, "pm2.5_dp"},
```

# 2.app icon

开发中...

# 3.main主程序

3.1主程序采用 状态机实现程序功能切换+pm2.5模块

```cpp
/*main.c*/
+#include "form_pm25.h"
+case MS_PM25:
+    TestPM25();                        /*pm2.5模块*/
+    ucStatus = MS_MAIN_MENU;
+break;
```

# 4.pm2.5模块程序设计

![](/assets/3.PNG)

## 1.打印pm2.5示数到串口，屏幕显示

添加pm2.5模块头文件

```cpp
/*
*********************************************************************************************************
*
*    模块名称 : pm2.5模块主库文件
*    文件名称 : form_pm25.h
*    版    本 : V1.0
*
*    Copyright (C), 2017/12/31, zhangzhuo2018@yeah.net
*
*********************************************************************************************************
*/
#ifndef _FORM_PM25_H_
#define _FORM_PM25_H_

extern void TestPM25(void);

#endif

/***************************** (END OF FILE) *********************************/
```

添加pm2.5模块主程序

```c
/*
*pm2.5模块主要部件
*/
uint16_t Handshake[]= {0x1B,0xAA,0xFF,0xFF,0xEF,0xEF};//液晶握手指令

uint16_t ADC_ConvertedValue, num;

#define SAMP_COUNT        20                /* 样本个数，表示200ms内的采样数据求平均值 */
// 局部变量，用于保存转换计算后的电压值        

float ADC_Vol;
void     GP2Y_GPIOInit(void);
uint32_t  GP2Y_GetADCValue(void);
extern uint8_t Answer;
void ReadData(void);

void GP2Y_GPIOInit(void)
{
    GPIO_InitTypeDef GPIO_InitStructure;
        /* PA5 output */
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_5;
    GPIO_InitStructure.GPIO_Mode  = GPIO_Mode_OUT;                                 //设置为输出模式
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;                              //设置端口输出速度为50MHZ
    GPIO_InitStructure.GPIO_OType = GPIO_OType_PP;                                 //设置为推挽输出
    GPIO_InitStructure.GPIO_PuPd  = GPIO_PuPd_NOPULL;                              //无上拉下拉
    GPIO_Init(GPIOA, &GPIO_InitStructure);        
    GPIO_ResetBits(GPIOA, GPIO_Pin_5);                                         

}

uint32_t GP2Y_GetADCValue(void)
{

    static uint16_t buf[SAMP_COUNT];
    static uint8_t write;
    uint32_t sum;
    uint8_t i;


    GPIO_ResetBits(GPIOA, GPIO_Pin_5);
    //Delay_ms(28);

    buf[write] =  ADC_GetConversionValue(ADC1);

    if (++write >= SAMP_COUNT)
    {
            write = 0;
    }
        sum = 0;
    for (i = 0; i < SAMP_COUNT; i++)
    {
                sum += buf[i];
    }
    ADC_ConvertedValue = sum / SAMP_COUNT;                                            /* ADC采样值由若干次采样值平均 */

    ADC_SoftwareStartConv(ADC1);                                                //使能指定的ADC1的软件转换启动功能
    //Delay_ms(4);
    GPIO_SetBits(GPIOA, GPIO_Pin_5);                                             //default low, ILED closed

    return ADC_ConvertedValue;
}





/*
*********************************************************************************************************
*    函 数 名: TestPM25
*    功能说明: 展示当前pm2.5示数&建议
*    形    参: 无
*    返 回 值: 无
*********************************************************************************************************
*/
void TestPM25(void)
{
        /***********************************************************************************
        *输出pm2.5示数
        *实验效果：
        *在LCD屏幕上打印PM2.5的数值
        *打开串口可以看到PM2.5的值
        ************************************************************************************/
        uint32_t pm;

        uint8_t ucKeyCode;        /* 按键代码 */
        uint8_t ucTouch;        /* 触摸事件 */
        uint8_t fRefresh;        /* 刷屏请求标志,1表示需要刷新 */
        FONT_T tFont, tFontBtn;    /* 定义一个字体结构体变量，用于设置字体参数 */
        char buf[128];
        uint16_t x, y;
        uint16_t usLineCap = 18;

        int16_t tpX, tpY;
        BUTTON_T tBtn;

        LCD_ClrScr(CL_BLUE);      /* 清屏，背景蓝色 */

    /* 设置字体参数 */
    {
        tFont.FontCode = FC_ST_16;    /* 字体代码 16点阵 */
        tFont.FrontColor = CL_WHITE;    /* 字体颜色 */
        tFont.BackColor = CL_BLUE;    /* 文字背景颜色 */
        tFont.Space = 0;                /* 文字间距，单位 = 像素 */

        /* 按钮字体 */
        tFontBtn.FontCode = FC_ST_16;
        tFontBtn.BackColor = CL_MASK;    /* 透明色 */
        tFontBtn.FrontColor = CL_BLACK;
        tFontBtn.Space = 0;
    }

    x = 5;
    y = 3;
    LCD_DispStr(x, y, "/*** PM2.5界面  made by zhangzhuo ***/", &tFont);            /* 在(8,3)坐标处显示一串汉字 */
    y += usLineCap;


    fRefresh = 1;    /* 1表示需要刷新LCD */
    while (1)
    {
        bsp_Idle();

        if (fRefresh)
        {
            fRefresh = 0;

            /* 显示按钮 */
            {
                tBtn.Font = &tFontBtn;

                tBtn.Left = BUTTON_RET_X;
                tBtn.Top = BUTTON_RET_Y;
                tBtn.Height = BUTTON_RET_H;
                tBtn.Width = BUTTON_RET_W;
                tBtn.Focus = 0;    /* 失去焦点 */
                tBtn.pCaption = "返回";
                LCD_DrawButton(&tBtn);
            }
        }

        ucTouch = TOUCH_GetKey(&tpX, &tpY);    /* 读取触摸事件 */
        if (ucTouch != TOUCH_NONE)
        {
            switch (ucTouch)
            {
                case TOUCH_DOWN:        /* 触笔按下事件 */
                    if (TOUCH_InRect(tpX, tpY, BUTTON_RET_X, BUTTON_RET_Y, BUTTON_RET_H, BUTTON_RET_W))
                    {
                        tBtn.Font = &tFontBtn;

                        tBtn.Left = BUTTON_RET_X;
                        tBtn.Top = BUTTON_RET_Y;
                        tBtn.Height = BUTTON_RET_H;
                        tBtn.Width = BUTTON_RET_W;
                        tBtn.Focus = 1;    /* 焦点 */
                        tBtn.pCaption = "返回";
                        LCD_DrawButton(&tBtn);
                    }
                    break;

                case TOUCH_RELEASE:        /* 触笔释放事件 */
                    if (TOUCH_InRect(tpX, tpY, BUTTON_RET_X, BUTTON_RET_Y, BUTTON_RET_H, BUTTON_RET_W))
                    {
                        tBtn.Font = &tFontBtn;

                        tBtn.Left = BUTTON_RET_X;
                        tBtn.Top = BUTTON_RET_Y;
                        tBtn.Height = BUTTON_RET_H;
                        tBtn.Width = BUTTON_RET_W;
                        tBtn.Focus = 1;    /* 焦点 */
                        tBtn.pCaption = "返回";
                        LCD_DrawButton(&tBtn);

                        return;        /* 返回 */
                    }
                    else    /* 按钮失去焦点 */
                    {
                        tBtn.Font = &tFontBtn;

                        tBtn.Left = BUTTON_RET_X;
                        tBtn.Top = BUTTON_RET_Y;
                        tBtn.Height = BUTTON_RET_H;
                        tBtn.Width = BUTTON_RET_W;
                        tBtn.Focus = 0;    /* 焦点 */
                        tBtn.pCaption = "返回";
                        LCD_DrawButton(&tBtn);

                    }
            }
        }

        ucKeyCode = bsp_GetKey();    /* 读取键值, 无键按下时返回 KEY_NONE = 0 */
        if (ucKeyCode != KEY_NONE)
        {
            /* 有键按下 */
            switch (ucKeyCode)
            {
                case JOY_DOWN_OK:        /* 摇杆OK键 */
                    break;

                default:
                    break;
            }
        }
        /*打印到串口，LCD显示*/
        pm = GP2Y_GetADCValue();
        sprintf(buf, " pm2.5 = %08X", pm);
        printf("%s\r\n", buf);
        LCD_DispStr(x, y, buf, &tFont);
    }
}
```

## 2.根据示数给出建议

```c
if(pm <= 75)
		{
			sprintf(buf, " 空气very good");
			printf("%s\r\n", buf);
		}
		else if(pm>75 && pm<=150)
		{
			sprintf(buf, " 空气look good");
			printf("%s\r\n", buf);
		}
		else if(pm>150 && pm<=300)
		{
			sprintf(buf, " 空气good");
			printf("%s\r\n", buf);
		}
		else if(pm>300 && pm<=1050)
		{
			sprintf(buf, " 空气just so so");
			printf("%s\r\n", buf);
		}
		else if(pm>1050 && pm <=3000)
		{
			sprintf(buf, " 空气bad");
			printf("%s\r\n", buf);
		}
		else
		{
			sprintf(buf, " 空气very bad");
			printf("%s\r\n", buf);
		}
		y += usLineCap;
		LCD_DispStr(x, y, buf, &tFont);
		y -= usLineCap;
	}
```



