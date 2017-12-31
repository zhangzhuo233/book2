# 1.app icon text

ico名称改成“pm2.5\_dp”

```c
//main_menu.c
66    {ID_ICON, 0, 0, ICON_HEIGHT, ICON_WIDTH, (uint16_t *)acChiLun, "pm2.5_dp"},
```

# 2.app icon

开发中...

# 3.app print

3.1主程序采用 状态机实现程序功能切换+pm2.5模块

```cpp
/*main.c*/
+#include "form_pm25.h"
+case MS_PM25:
TestPM25();						/*pm2.5*/
ucStatus = MS_MAIN_MENU;
break;
```



