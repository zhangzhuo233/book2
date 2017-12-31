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

# 4.pm2.5模块设计

4.1添加pm2.5模块头文件

```cpp
/*form_pm25.h*/
/*
*********************************************************************************************************
*
*    Ä£¿éÃû³Æ : PM2.5½çÃæ³ÌÐò
*    ÎÄ¼þÃû³Æ : form_pm25.h
*    °æ    ±¾ : V1.0
*
*    Copyright (C), 2017/12/31, zhangzhuo, zhangzhuo2018@yeah.net
*
*********************************************************************************************************
*/

#ifndef _FORM_PM25_H_
#define _FORM_PM25_H_

extern void TestPM25(void);

#endif

/***************************** °²¸»À³µç×Ó www.armfly.com (END OF FILE) *********************************/
```

4.2添加pm2.5模块主程序界面

