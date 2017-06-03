MATLAB Coder可以从MATLAB代码生成独立的、可读性强、可移植的C/C++代码。

使用MATLAB Coder产生代码的3个步骤：
1. 准备用于产生代码的MATLAB算法；
2. 检查MATLAB代码的兼容性（有些matlab代码语句并不能生成c/c++代码）；
3. 产生最终使用的源代码或MEX。

利用MATLAB Coder生成c++代码，并在vs2010中验证：

**第1步：** 打开Matlab2013a，新建interweava.m文件与deinterweaving.m文件
```matlab
function [interweava_out,interweava_zeros] = interweava(interweava_in,mode) %#codegen
interweava_zeros = 0;
if strcmp(mode,'无')
    interweava_out = interweava_in;
elseif strcmp(mode, '块交织')
    interweava_zeros = mod(length(interweava_in),100);
    if interweava_zeros
        interweava_zeros = 100 - interweava_zeros;
        interweava_in = [interweava_in,zeros(1,interweava_zeros)];
    end

    interweava_temp = reshape(interweava_in,100,[]);
    interweava_out = reshape(interweava_temp',1,[]);
else
    interweava_out = 1;
end
end
```
其中，%#codegen可以防止出现警告错误
```matlab
function [deinterweava_out] = deinterweaving(deinterweava_in,mode,interweava_zeros)
if strcmp(mode,'无')
    deinterweava_out = deinterweava_in;
elseif strcmp(mode, '块交织')
    deinterweava_temp = reshape(deinterweava_in,[],100);
    deinterweava_out = reshape(deinterweava_temp',1,[]);
    deinterweava_out = deinterweava_out(1:end-interweava_zeros);
else
    deinterweava_out = 1;
end
end
```

**第2步：** 在命令窗口，输入mex -setup,选中一个存在的编译器；

**第3步：** 在命令窗口输入coder（图形界面），回车，弹出MATLAB Coder Project对话框；

**第4步：** 在New选项卡Name中输入一个工程名interweava.prj；点击Ok，弹出MATLAB Coder MEX Function对话框；

**第5步：** 在Overview选项卡中，点击Add files，弹出对话框，选中interweava.m打开，并选择输入变量的数据类型；

**第6步：** 选中Build选项卡，Output type中选择c/c++ Static Library；选中Generate code only；

**第7步：** 点击Build，进行编译；点击View report，弹出Code Generation Report对话框，此时，变量interweava_in、mode、interweava_out、interweava_zeros会显示相应的变量信息；

![](VC调用Matlab生成的c\1.png) ![](VC调用Matlab生成的c\2.png)

**第8步：** 打开VS2010，新建Win32 Console Application工程，并选择Empty project；

![](VC调用Matlab生成的c\3.png) ![](VC调用Matlab生成的c\4.png)

**第9步：** 将Matlab生成的codegen\\lib\\deinterweaving\\下所有.c和.h文件复制到新建工程目录，并添加到工程；

![](VC调用Matlab生成的c\5.png)

**第10步：** 新建一个cpp文件，代码为
```cpp
#include <iostream>
extern"C"
{
#include "deinterweaving.h"
#include "interweava.h"
#include "deinterweaving_types.h"
#include "deinterweaving_emxAPI.h"
}
using namespace std;
void main()
{
	double a[10] = {1,0,0,1,1,1,0,0,0,0};
	emxArray_real_T * data_in = emxCreate_real_T(1,1001);
	emxArray_char_T * mode_in = emxCreateWrapper_char_T("块交织",1,6);
	emxArray_real_T * data_out = emxCreate_real_T(1,1100);
	emxArray_real_T * data_in1 = emxCreate_real_T(1,1001);
	real_T zero_num = 9;

	for (int ii=0;ii<100;ii++)
	{
		for (int jj=0;jj<10;jj++)
		{
			data_in->data[ii*10+jj] = a[jj];
		}
	}
	data_in->data[1000] = 1;

	interweava(data_in,mode_in,data_out,&zero_num);
	deinterweaving(data_out,mode_in,zero_num,data_in1);

	for (int ii=0;ii<1100;ii++)//data_out->allocatedSize
	{
		cout<<data_in->data[ii]<<" "<<data_in1->data[ii]<<" "<<data_out->data[ii]<<" "<<ii<<endl;
	}
	cout<<zero_num<<endl;
	emxDestroyArray_char_T(mode_in);
	emxDestroyArray_real_T(data_in);
	emxDestroyArray_real_T(data_out);
	emxDestroyArray_real_T(data_in1);
	system("pause");
}
```
**注意：** Matlab中的矩阵转化为c中的结构体或数组时，数据是按列转化的
