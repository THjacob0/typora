# OpenGL学习笔记

## 一：第一次学习

GLFW：为Opengl开发库全称为OpenGL Framework主要用于创建窗口、上下文和处理用户输入（如键盘和鼠标事件）。它提供了一个简单、统一的API来处理窗口系统和输入设备，使得开发者可以更容易地编写跨平台的OpenGL应用程序

GLEW：为OPENGL扩展加载库全称为OpenGL Extension Wrangler LibraryGLEW 用于管理OpenGL的扩展。OpenGL是一个不断发展的API，经常有新的功能被添加进来，这些新功能被称为"扩展"。GLEW 允许开发者查询和使用这些扩展，而不需要关心它们是如何在不同平台上实现的。

### 操作项目为：Opengl_triangle

1.下载对应的学习库,主要是库的链接，注意使用相对路径

$(SolutionDir)

2.创建三角形

`glBegin(GL_TRIANGLES);`

`//中间放置代码`

`创建三个顶点(这种方式是直接写，不需要缓冲区之类的东西)：`

`glVertex2f(-0.5f,-0.5f);//两个位置分别为x，y`

`glVertex2f(0.0f,0.5f);`

`glVertex2f(0.5f,-0.5f);`

`glEnd();`

## 二：第二次学习

操作项目为：Opengl_triangle

1.使用GLEW，在包含glew.h头文件时需要将它防止在头文件最前面

2.无法链接(link),需要在属性中的C/C++中定义中的预处理器添加glew static

3.在调用glewInit时发生了报错，不能调用的原因是需要创建一个有效的opengl渲染上下文创建位置为glfwMakeContextCurrent()之后

    //在这里创建glew的渲染上下文
    /*************************************/
    glfwMakeContextCurrent(window);
    if (glewInit() != GLEW_OK)
        std::cout << "Error!" << std::endl;
    unsigned int a;
    glGenBuffers(1, &a);
    /*************************************/

**4.使用现代的opengl来进行操作**（包括缓冲区，着色器等等）

为顶点缓冲区内容：

    //定义缓冲区
       float positions[6] = {
           -0.5f, -0.5f,
            0.0f,  0.5f,
            0.5f, -0.5f
       };
       unsigned int buffer;//用于存储缓冲区id
       glGenBuffers(1,&buffer);//生成缓冲区，第一个参数为生成数量，第二个为指向存储生成的缓冲区的id的数组
       glBindBuffer(GL_ARRAY_BUFFER,buffer);
       glBufferData(GL_ARRAY_BUFFER, 6 * sizeof(float),positions,GL_STATIC_DRAW);
       //使用 glBufferData 函数为 GL_ARRAY_BUFFER 目标的当前绑定缓冲区分配存储空间
       //参数分别为：指定缓冲区目标，大小，上传的数据，数据的使用方式
       /* Loop until the user closes the window */
       while (!glfwWindowShouldClose(window))
       {
           /* Render here 在这里渲染*/
           glClear(GL_COLOR_BUFFER_BIT);
           glDrawArrays(GL_TRIANGLES, 0, 3);
           //三个参数分别为mode：指定绘制模式，GL_TRIANGLES 表示每个三个顶点组成一个三角形
           //first：指定从顶点数组中开始读取顶点数据的起始索引
           //count：指定要绘制的顶点数量。设置为 3，意味着绘制三个顶点，形成一个三角形
       /* Swap front and back buffers */
       glfwSwapBuffers(window);
    
       /* Poll for and process events */
       glfwPollEvents();
       }
5.顶点属性，布局定义

上文中的是顶点的坐标，而坐标不是顶点，坐标是顶点的一部分顶点还包括颜色，纹理，法线，颜色，按法线切线等等。 glVertexAttribPointer(index,size,type,normalized,stride,pointer)中所包含的属性

index,表示当前属性是顶点中的第几个属性，假设顶点有位置纹理法线等属性，将位置放在index0，纹理放在index1，法线放在index2

size，表示占用几个数字，比如二维坐标就用2个

type，数据类型

normalized，是否正则化，颜色区间从0-255转为0-1

stride，每个顶点之间的字节偏移量，例子中，有三个顶点，每个顶点有位置，纹理坐标，法线三个顶点属性。位置为三维，纹理为二维，法线为三维。3+2+3=8一个浮点数为4字节，8*4=32，所以stride=32

pointer，表示当前顶点到当前属性的距离，顶点有位置，纹理坐标，法线三个顶点属性，位置为index0，纹理坐标index1，法线index2，那么位置属性与当前顶点的偏移量为0，若为三维坐标纹理为3*4=12，则法线为12+4*2=20

glVertexAttribPointer(0,2,GL_FLOAT,GL_FALSE,2*sizeof(float),0);

6.着色器（一个运行在gpu上的程序）分为顶点着色器和片段着色器

顶点着色器：提供给那些顶点位置，能够给他们提供转换，这样opengl可以将这些数字转换成在屏幕上的坐标，所以我们在我们的窗口的正确位置正确地方上可以看到我们的图形，每个顶点运行一次

片段着色器：会为每个像素运行一次光栅化，窗口实际上由像素组成，像是一个像素数组，而片段着色器的基本目的是决定像素的颜色应该是什么。

## 三：第三次学习

1.编译着色器与创建着色器

`//编译着色器`
`static signed int CompileShader(unsigned int type,`
                                `const std::string& source)`
`{`
    `unsigned int id= glCreateShader(type);`
    `//指针指向地址`
    `const char* src = source.c_str();`
    `//调用glShaderSource指定着色器:id,count,源的内存地址，变量长度`
    `glShaderSource(id, 1, &src, nullptr);`
    `//开始编译着色器`
    `glCompileShader(id);`
    `//TODO:可能会存在语法错误等等问题，用一行源代码去表示出来`
    `//TODO:Error handling`
    `//检查出错`
    `/************************************************/`
    `int result;`
    `glGetShaderiv(id, GL_COMPILE_STATUS, &result);`
    `if (result == GL_FALSE)`
    `{`
        `int length;`
        `glGetShaderiv(id, GL_INFO_LOG_LENGTH,&length);`
        `char* message=(char*)alloca(length*sizeof(char));`
        `glGetShaderInfoLog(id, length, &length,message);`
        `std::cout << "编译失败"<<`
                   `(type==GL_VERTEX_SHADER ? "vertex":"fragment")`
                  `<< "着色器！" << std::endl;`
        `std::cout << message << std::endl;`
        `//着色器编译失败了删除它`
        `glDeleteShader(id);`
        `return 0;`
    `}`
    `/************************************************/`
    `return id;`
`}`
`//创建着色器`
`/************************************************/`
`static unsigned int CreateShader(const std::string& vertexShader,`
                        					`const std::string& fragmentShader)`
`{`
    `unsigned int program = glCreateProgram();`
    `unsigned int vs = CompileShader(GL_VERTEX_SHADER,vertexShader);`
    `unsigned int fs = CompileShader(GL_FRAGMENT_SHADER,fragmentShader);`
    `//将两个着色器链接在一起`
    `glAttachShader(program, vs);`
    `glAttachShader(program, fs);`
    `glLinkProgram(program);`
    `glValidateProgram(program);`
    `//链接后可以删除着色器中间内容`
    `glDeleteShader(vs);`
    `glDeleteShader(fs);`
    `return program;`
`}`
`/************************************************/`

2.**在主函数中编写着色器代码**

`//写着色器的代码部分`
`/*****************顶点着色器********************/
std::string vertexShader =
    //表示使用gl着色语言
    "#version 330 core\n"
    "\n"
    "layout(location = 0) in vec4 position;\n"
    "\n"
    "void main()\n"
    "{\n"
    //由于gl_Positon只能接受vec4，因此需要将其转化为vec4形式
    "   gl_Position = position;\n"
    "}\n";
/**********************************************/`

    /*****************片段着色器********************/
`std::string fragmentShader =`
    `//表示使用gl着色语言`
    `"#version 330 core\n"`
    `"\n"`
    `"layout(location = 0) out vec4 color;\n"`
    `"\n"`
    `"void main()\n"`
    `"{\n"`
    `//由于gl_Positon只能接受vec4，因此需要将其转化为vec4形式`
    `//这里颜色表示为R,G,B,A`
    `"   color = vec4(1.0,0.0,0.0,1.0);\n"`
    `"}\n";`
`/**********************************************/`
`unsigned int shader = CreateShader(vertexShader,fragmentShader);`
`glUseProgram(shader);`

3.如何重写着色器

项目结构：

![image-20240605170712733](C:\Users\jacob\AppData\Roaming\Typora\typora-user-images\image-20240605170712733.png)

将上文中的shader放入该文件中删除""等等，可以采用ctl+H替换

在Application文件中需要将下文加入，这是将着色器代码进行读取

`//使用文件流`
`#include<fstream>`
`//使用字符串流`
`#include<sstream>`

`struct ShaderProgramSource`
`{`
    `std::string VertexSource;`
    `std::string FragmentSource;`

`};`


`//这段代码将着色器文件中的内容放入`
`static ShaderProgramSource ParseShader(const std::string& filepath)`
`{` 
    `std::ifstream stream(filepath);`
    `if (!stream.is_open())`
    `{`
        `std::cerr << "Failed to open file: " << filepath << std::endl;`
        `return { "", "" };`
    `}`
    `else`
    `{`
        `std::cout << "Success to open file: " << filepath << std::endl;`

    }
    
    //判断类型
    enum class ShaderType
    {
        NONE = -1, VERTEX = 0,FRAGMENT=1
    };
    std::string line;
    std::stringstream ss[2];//一个为顶点，一个为片段
    ShaderType type = ShaderType::NONE;
    while (getline(stream,line))
    {
        if (line.find("#shader") != std::string::npos)
        {
            if (line.find("vertex") != std::string::npos)
            {
                //设置模式为vertex
                type = ShaderType::VERTEX;
            }
            else if (line.find("fragment") != std::string::npos)
            {
                //设置模式为fragment
                type = ShaderType::FRAGMENT;
            }
    
        }
        else
        {   
            
                ss[(int)type] << line << "\n";


``            
        }
    }
    return { ss[0].str(),ss[1].str() };
`}`

这段是使用代码：

`ShaderProgramSource source = ParseShader("./res/shaders/Basic.shader");`

`std::cout << "VERTEX" << std::endl;`
`std::cout << source.VertexSource << std::endl;`
`std::cout << "FRAGMENT" << std::endl;`
`std::cout << source.FragmentSource << std::endl;`

`if (source.VertexSource.empty() || source.FragmentSource.empty())`
`{`
    `std::cerr << "Shader source code is empty!" << std::endl;`
    `glfwTerminate();`
    `return -1;`
`}`

`unsigned int shader = CreateShader(source.VertexSource, source.FragmentSource);`
`glUseProgram(shader);`

4.索引缓冲区（GPU上面运行的都是三角形）允许重复使用顶点

以正方形为例修改position代码：

    float positions[] = {
        -0.5f, -0.5f,//0
         0.5f, -0.5f,//1
         0.5f,  0.5f,//2
        -0.5f,  0.5f,//3
    
    };
    //表示组成三角形的顶点
    unsigned int indices[] = {
        0,1,2,
        2,3,0
    };

`unsigned int ibo;//用于存储缓冲区id`
`glGenBuffers(1, &ibo);//生成缓冲区，第一个参数为生成数量，第二个为指向存储生成的缓冲区的id的数组`
`glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, ibo);`
`glBufferData(GL_ELEMENT_ARRAY_BUFFER, 6 * sizeof(unsigned int), indices, GL_STATIC_DRAW);`

        glDrawElements(GL_TRIANGLES, 6 ,GL_UNSIGNED_INT,nullptr);
        //绘制方式允许你通过索引来重复使用顶点
        // mode：指定要渲染的几何图形的模式，例如 GL_TRIANGLES。
        //count：指定要渲染的元素数量。
        //type：指定元素数组中索引的数据类型，例如 GL_UNSIGNED_INT 或 GL_UNSIGNED_BYTE。
        //indices：指向包含索引的数组的指针。

## 四：第四次学习

1.错误（主要两种方法，一种为gl get error为一个可以调用的函数

第二种为opengl添加了一个功能，gl debug message callback）

**gl get error部分**

首先，在我们给opengl的每个函数调用之前，我们清除所有错误，我们在while循环中调用gl get error直到它等于没有错误意味着我们从opengl中检索了所有可能的错误。然后调用gldrawelements然后再次调用gl geterror

`//glgeterror部分,清除错误`
`static void GLClearError()`
`{`
    `while (glGetError() != GL_NO_ERROR);`
`}`
`//检测错误`
`static void GLCheckError()`
`{`
    `while (GLenum error = glGetError())`
    `{`
        `std::cout << "[OpenGL Error](" << error << ")" << std::endl;`
    `}`
`}`

调用上面两个函数，先清除上文中的错误，最后在检查，就可以保证检查出来的所有错误都来自于中间。

    GLClearError();
    glDrawElements(GL_TRIANGLES, 6,GL_UNSIGNED_INT,nullptr);
    GLCheckError();

上文的方式只能显示出一个错误代码，下文进行了修改

首先修改检查代码：

`static void GLClearError()`
`{`
    `while (glGetError() != GL_NO_ERROR);`
`}`
`//检测错误`
`static bool GLLogCall(const char* function,const char* file,int line)`
`{`
    `while (GLenum error = glGetError())`
    `{`
        `std::cout << "[OpenGL Error](" << error << ")" << function<<`
            `" " << file << ":" << line << std::endl;`
        `return false;`
    `}`
    `return true;`
`}`

然后使用宏的方式，便不需要再主函数中定义，可以直接检查：

`//使用断言，如果为假，则会调用一个函数会在代码中插入断点并打断调试器`
`#define ASSERT(x) if (!(x)) __debugbreak();
//定义glcearerror来确保我们清除了所有东西
#define GLCall(x) GLClearError();\
    x;\
    ASSERT(GLLogCall(#x,__FILE__,__LINE__))`

使用GLCall去包含每一个OPENGL函数即可对每一个opengl函数进行检查。例如：

`GLCall(glDrawElements(GL_TRIANGLES,6,GL_UNSIGNED_INT,nullptr));`

2.uniforms(基本上只是从cpu端获取数据的一种方式，在这里从c++到我们的着色器，这样就可以将他当作变量来用)

首先修改着色器：

`#shader vertex`
`#version 330 core`
`layout(location = 0) in vec4 position;`
`void main()`
`{`
    `gl_Position = position;`
`};`
`#shader fragment`
`#version 330 core`
`layout(location = 0) out vec4 color;`

`//修改部分`

`uniform vec4 u_Color;`

`void main()`
`{`
    `color = u_Color;`
`};`

主函数中在 GLCall(glUseProgram(shader));后面加入：

    GLCall(int location = glGetUniformLocation(shader,"u_Color"));
    ASSERT(location!=-1);
    GLCall(glUniform4f(location,0.2f,0.3f,0.8f,1.0f));
    float r = 0.0f;
    float increment = 0.05f;


在 glClear(GL_COLOR_BUFFER_BIT);后面加入：

        GLCall(glUniform4f(location, r, 0.3f, 0.8f, 1.0f));
        GLCall(glDrawElements(GL_TRIANGLES, 6 ,GL_UNSIGNED_INT,nullptr));
        if (r > 1.0f)
        {
            increment = -0.05f;
        }
        else if(r < 0.0f)
            increment = 0.05f;
        r += increment;

可以实现动画过渡

3.顶点数组（基本上是绑定缓冲，顶点缓冲的一种方式对实际的顶点缓冲的布局有一定规范）顶点数组对象让我们做的是绑定顶点规范，我们通过使用顶点来指定一个指向实际缓冲区的行程指针，指向实际顶点缓冲，或一系列顶点缓冲，这取决于我们如何组织它。

绘制东西的方式：绑定着色器，绑定顶点缓冲区，设置顶点布局，绑定索引缓冲区，发出绘制调用，它从那样变成绑定我们的着色器。

在绑定缓冲区之前创建顶点数组对象并绑定：

    //创建顶点数组对象
    unsigned int vao;
    GLCall(glGenVertexArrays(1,&vao));
    GLCall(glBindVertexArray(vao));

你要么为整个程序创建一个全局vao，然后每次绑定不同的缓冲区和不同的顶点规则，要么为每一块几何图形都创建一个单独的vao，或者每一块几何图形都是独一无二的。

## 四：第四次学习

1.在过去写的代码中抽象成类，可以更容易的使用

 关注于顶点缓冲区和索引缓冲区与顶点数组

创建renderer.h与renderer.cpp 

删除主函数中的断言部分将其放入到renderer.h中

```
#include<GL/glew.h>
//使用断言，如果为假，则会调用一个函数会在代码中插入断点并打断调试器
#define ASSERT(x) if (!(x)) __debugbreak();
//定义glcearerror来确保我们清除了所有东西
#define GLCall(x) GLClearError();\
    x;\
    ASSERT(GLLogCall(#x,__FILE__,__LINE__))
//glgeterror部分,清除错误
void GLClearError();
//检测错误
bool GLLogCall(const char* function, const char* file, int line);
```



在主函数中添加#include "Renderer.h"

对于顶点缓冲区与索引缓冲区创建俩个文件为IndexBuffer.cpp，IndexBuffer.h与VertexBuffer.cpp与VertexBuffer.h

IndexBuffer.cpp代码为：

```
#include "IndexBuffer.h"
#include "Renderer.h"
IndexBuffer::IndexBuffer(const unsigned int* data, unsigned int count)
    //使用表达式将count设置为count
    : m_Count(count)
{
    

    ASSERT(sizeof(unsigned int) == sizeof(GLuint));
    //这里的第二个参数为rendere id   
    glGenBuffers(1, &m_RenderedID);//生成缓冲区，第一个参数为生成数量，第二个为指向存储生成的缓冲区的id的数组
    glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, m_RenderedID);
    glBufferData(GL_ELEMENT_ARRAY_BUFFER,count*sizeof(GLuint), data, GL_STATIC_DRAW);

}

IndexBuffer::~IndexBuffer()
{
    GLCall(1,&m_RenderedID);
}

void IndexBuffer::Bind()const
{
    glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, m_RenderedID);
}

void IndexBuffer::Unbind()const
{
    glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, 0);

}
```

IndexBuffer.h代码为：

```
#pragma once
class IndexBuffer
{
private:
	unsigned int m_RenderedID;
	unsigned int m_Count;
public:
	//创建一个公共构造函数
	IndexBuffer(const unsigned int* data, unsigned int count);
	~IndexBuffer();
	void Bind() const;
	void Unbind() const;
	inline unsigned int GetCount() const { return m_Count; }
};
```

VertexBuffer.cpp代码为：

```
#include "VertexBuffer.h"
#include "Renderer.h"
VertexBuffer::VertexBuffer(const void* data, unsigned int size)
{
    //这里的第二个参数为rendere id   
    glGenBuffers(1, &m_RenderedID);//生成缓冲区，第一个参数为生成数量，第二个为指向存储生成的缓冲区的id的数组
    glBindBuffer(GL_ARRAY_BUFFER, m_RenderedID);
    glBufferData(GL_ARRAY_BUFFER, size, data, GL_STATIC_DRAW);
}

VertexBuffer::~VertexBuffer()
{
    GLCall(1,&m_RenderedID);
}

void VertexBuffer::Bind()const
{
    glBindBuffer(GL_ARRAY_BUFFER, m_RenderedID);
}

void VertexBuffer::Unbind()const
{
    glBindBuffer(GL_ARRAY_BUFFER, 0);
}
```

VertexBuffer.h代码为：

```
#pragma once
class VertexBuffer
{
private:
	unsigned int m_RenderedID;
public:
	//创建一个公共构造函数
	VertexBuffer(const void* data, unsigned int size);
	~VertexBuffer();
	void Bind()const;
	void Unbind()const;
	
};
```

删除主函数中代码为：

```
unsigned int buffer;//用于存储缓冲区id
glGenBuffers(1,&buffer);//生成缓冲区，第一个参数为生成数量，第二个为指向存储生成的缓冲区的id的数组
glBindBuffer(GL_ARRAY_BUFFER,buffer);
glBufferData(GL_ARRAY_BUFFER, 4 * 2 * sizeof(float),positions,GL_STATIC_DRAW);
```

添加：VertexBuffer vb(positions, 4*2*sizeof(float));

删除主函数中代码为：

```
unsigned int ibo;//用于存储缓冲区id
glGenBuffers(1, &ibo);//生成缓冲区，第一个参数为生成数量，第二个为指向存储生成的缓冲区的id的数组
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, ibo);
glBufferData(GL_ELEMENT_ARRAY_BUFFER, 6 * sizeof(unsigned int), indices, GL_STATIC_DRAW);
```

添加：IndexBuffer ib(indices, 6);

删除主函数中代码为：

GLCall(glBindBuffer(GL_ELEMENT_ARRAY_BUFFER,ibo));

添加：ib.bind();

2.顶点数组并进行抽象（顶点数组需要做的事是将一个顶点缓冲区与某种布局绑定在一起）

在application程序中添加了

        VertexArray va;
        //创建顶点缓冲区对象VBO
        VertexBuffer vb(positions, 4 * 2 * sizeof(float));
        //定义缓冲区布局
        VertexBufferLayout layout;
        layout.Push<float>(2);
        va.AddBuffer(vb,layout);

创建VertexBufferLayout.h文件，VertexArray.h，VertexArray.cpp文件

3.将编写的opengl着色器代码抽象出来，放在一个易于使用和理解的api后面。

创建Shader.h与Shader.cpp文件，将应用程序中的代码放入到这两个文件中进行改造，并且将应用程序中的一些解绑与绑定更换为unbind与bind方法。

使用了缓存机制来查找位置

## 五：第五次学习

1.创建渲染器类（渲染器（Renderer）的作用是将场景数据转换为屏幕上的图像。渲染器是图形渲染管道中的核心组件，它接收来自应用程序的数据（如顶点、纹理、光照信息等），并通过一系列计算将这些数据转换为最终的图像。这些计算通常包括几何处理、光照计算、着色处理等。）

在renderer.h与renderer.cpp进行修改，将应用程序中的gldraw与glclear函数写入renderer中然后在主函数中进行调用

**2.纹理**

示例（将电脑上的png文件转换，在着色器中进行采样，将他绘制在屏幕上。第一步以某种方式将图像加载到cpu中，在opengl中创建纹理，在渲染的时候我们绑定纹理，修改着色器来绑定纹理插槽，在着色器中对纹理采样）

加载图像到cpu内存中：

使用github中的代码

创建texture.h与texture.cpp文件并在其中完成操作

程序代码中放入

        Texture texture("res/textures/ChernoLogo.png");
        texture.Bind();
        shader.SetUniform1i("u_Texture",0);

3.进行混合(要注意图片得是带透明度的原始png，不能是转换的)

    GLCall(glEnable(GL_BLEND));
        GLCall(glBlendFunc(GL_SRC_ALPHA,GL_ONE_MINUS_SRC_ALPHA));

# 六：第六次学习

1.glm数学库

利用glm数学库可以使得将纹理映射协调，使用投影矩阵

在application文件中加入：

```
#include "glm/glm.hpp"
#include "glm/gtc/matrix_transform.hpp"
```

创建投影矩阵

```
glm::mat4 proj = glm::ortho(-0.2f, 2.0f, -1.5f, 1.5f,-1.0f,1.0f);
//投影空间的左边界是 - 0.2f。
//右边界是 2.0f。
//底部边界是 - 1.5f。
//顶部边界是 1.5f。
//近裁剪平面的Z坐标是 - 1.0f。
//远裁剪平面的Z坐标是 1.0f。
```

要做的事情是将顶点位置与投影矩阵相乘（着色器中完成）

在着色器basic代码中添加统一变量

`uniform mat4 u_MVP;`

并与position相乘

在Shader.h中添加setmat4函数

在shader.cpp中添加相应的setmat4函数

2.投影矩阵（将空间中所有3D点转换为2D窗口中的东西）

将所有位置转换为标准设备坐标（即标准化空间，xyz轴在-1到1的那种）

3.模型视图投影（mvp）

一个相机——视图矩阵是相机的位置（旋转，缩放等等）

模型矩阵是位置（变换）

**要做的事情就是相乘（投影矩阵乘上相机矩阵乘上模型矩阵乘上顶点位置）**

        //创建投影矩阵,下面为4×3的纵横比
        glm::mat4 proj = glm::ortho(0.0f, 960.0f, 0.0f, 540.0f,-1.0f,1.0f);
        //投影空间的左边界是 - 0.2f。
        //右边界是 2.0f。
        //底部边界是 - 1.5f。  
        //顶部边界是 1.5f。
        //近裁剪平面的Z坐标是 - 1.0f。
        //远裁剪平面的Z坐标是 1.0f。
        //创建顶点位置,视口位于屏幕的(100, 100)位置，宽度和高度都是0，深度是1，
        //glm::vec4 vp(100.0f, 100.0f, 0.0f, 1.0f);
        //进行矩阵的加法，模拟相机，逆向运算，相机向左，则物品往右
        //创建了一个glm::mat4类型的矩阵view，这个矩阵用于表示相机（或观察者）的视图变换。glm::translate函数用于创建一个平移矩阵，它将一个对象沿着给定的向量移动。
        //glm::mat4(1.0f)创建了一个单位矩阵，它是一个4x4的矩阵，对角线上的元素都是1，其余元素都是0。这个单位矩阵表示没有变换发生。
        //glm::translate函数的第一个参数是单位矩阵，第二个参数是平移向量。glm::translate函数将单位矩阵与平移向量结合，生成一个新的变换矩阵，这个矩阵将对象沿着x轴负方向移动100个单位。
        glm::mat4 view= glm::translate(glm::mat4(1.0f), glm::vec3(-100, 0, 0));;
        //创建模型矩阵，模型上移
        glm::mat4 model = glm::translate(glm::mat4(1.0f), glm::vec3(200, 200, 0));;
        //下面即为模型视图投影
        glm::mat4 mvp = proj * view * model;

# 七：第七次学习

1.ImGui

将文件包含进vendor

将例子中的代码进行粘贴

创建imgui上下文：

```
ImGui::CreateContext();
ImGui_ImplGlfwGL3_Init(window, true);
ImGui::StyleColorsDark();
```

创建glfw帧：放在renderer.Clear();之后

```
ImGui_ImplGlfwGL3_NewFrame();
```

最后进行渲染

    ImGui::Render();      ImGui_ImplGlfwGL3_RenderDrawData(ImGui::GetDrawData());

最后关闭渲染窗口：

```
ImGui_ImplGlfwGL3_Shutdown();
ImGui::DestroyContext();
```

渲染窗口设计：

    bool show_demo_window = true;
    bool show_another_window = false;
    ImVec4 clear_color = ImVec4(0.45f, 0.55f, 0.60f, 1.00f);
                {
                    static float f = 0.0f;
                    static int counter = 0;
                    ImGui::Text("Hello, world!");                           // Display some text (you can use a format string too)
                    ImGui::SliderFloat("float", &f, 0.0f, 1.0f);            // Edit 1 float using a slider from 0.0f to 1.0f    
                    ImGui::ColorEdit3("clear color", (float*)&clear_color); // Edit 3 floats representing a color
    
                    ImGui::Checkbox("Demo Window", &show_demo_window);      // Edit bools storing our windows open/close state
                    ImGui::Checkbox("Another Window", &show_another_window);
    
                    if (ImGui::Button("Button"))                            // Buttons return true when clicked (NB: most widgets return true when edited/activated)
                        counter++;
                    ImGui::SameLine();
                    ImGui::Text("counter = %d", counter);
    
                    ImGui::Text("Application average %.3f ms/frame (%.1f FPS)", 1000.0f / ImGui::GetIO().Framerate, ImGui::GetIO().Framerate);
                }

2.批量渲染对象

3.设置测试框架

```
Application.cpp代码存放

#include<GL/glew.h>
#include <GLFW/glfw3.h>
#include<iostream>
#include<string>
//使用文件流
#include<fstream>
//使用字符串流
#include<sstream>
#include "Renderer.h"
#include "IndexBuffer.h"
#include "VertexBuffer.h"
#include "VertexArray.h"
#include "Shader.h"
#include "VertexBufferLayout.h"
#include "Texture.h"

#include "glm/glm.hpp"
#include "glm/gtc/matrix_transform.hpp"
#include "imgui/imgui.h"
#include "imgui/imgui_impl_glfw_gl3.h"
int main(void)
{
    GLFWwindow* window;

    /* Initialize the library */
    if (!glfwInit())
        return -1;
    
    //创建一个glfwwindow上下文
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);




    /* Create a windowed mode window and its OpenGL context */
    window = glfwCreateWindow(640, 480, "Hello World", NULL, NULL);
    if (!window)
    {
        glfwTerminate();
        return -1;
    }
    
    /* Make the window's context current */
    //在这里创建glew的渲染上下文
    /*************************************/
    glfwMakeContextCurrent(window);
    
    glfwSwapInterval(1); 
    
    if (glewInit() != GLEW_OK)
        std::cout << "Error!" << std::endl;
    std::cout << glGetString(GL_VERSION) << std::endl;
    {
        /*************************************/
        //定义缓冲区
        float positions[] = {
             -50.0f,  -50.0f,0.0f,0.0f,//0 后面两个为纹理坐标，纹理从左下角开始
              50.0f,  -50.0f,1.0f,0.0f,//1
              50.0f,   50.0f,1.0f,1.0f,//2
             -50.0f,   50.0f,0.0f,1.0f //3


        };
        unsigned int indices[] = {
            0,1,2,
            2,3,0
        };
    
        //进行混合
        GLCall(glEnable(GL_BLEND));
        GLCall(glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA));
        
        //创建顶点数组对象VAO
        //unsigned int vao;
        //GLCall(glGenVertexArrays(1, &vao));
        //GLCall(glBindVertexArray(vao));
        //创建顶点缓冲区对象VBO
    
        VertexArray va;
        //创建顶点缓冲区对象VBO
        VertexBuffer vb(positions, 4 * 4 * sizeof(float));
        //定义缓冲区布局
        VertexBufferLayout layout;
        layout.Push<float>(2);
        layout.Push<float>(2);
        va.AddBuffer(vb,layout);
    
        //启用顶点属性数组并设置顶点属性指针，这里将VAO和VBO两个连在了一起
        //GLCall(glEnableVertexAttribArray(0));
        //GLCall(glVertexAttribPointer(0, 2, GL_FLOAT, GL_FALSE, 2 * sizeof(float), 0));
        //创建索引缓冲区对象（IBO）并绑定索引数据
        IndexBuffer ib(indices, 6);
    
        //创建投影矩阵,下面为4×3的纵横比
        glm::mat4 proj = glm::ortho(0.0f, 960.0f, 0.0f, 540.0f,-1.0f,1.0f);
        //投影空间的左边界是 - 0.2f。
        //右边界是 2.0f。
        //底部边界是 - 1.5f。  
        //顶部边界是 1.5f。
        //近裁剪平面的Z坐标是 - 1.0f。
        //远裁剪平面的Z坐标是 1.0f。
        //创建顶点位置,视口位于屏幕的(100, 100)位置，宽度和高度都是0，深度是1，
        //glm::vec4 vp(100.0f, 100.0f, 0.0f, 1.0f);
        //进行矩阵的加法，模拟相机，逆向运算，相机向左，则物品往右
        //创建了一个glm::mat4类型的矩阵view，这个矩阵用于表示相机（或观察者）的视图变换。glm::translate函数用于创建一个平移矩阵，它将一个对象沿着给定的向量移动。
        //glm::mat4(1.0f)创建了一个单位矩阵，它是一个4x4的矩阵，对角线上的元素都是1，其余元素都是0。这个单位矩阵表示没有变换发生。
        //glm::translate函数的第一个参数是单位矩阵，第二个参数是平移向量。glm::translate函数将单位矩阵与平移向量结合，生成一个新的变换矩阵，这个矩阵将对象沿着x轴负方向移动100个单位。
        glm::mat4 view= glm::translate(glm::mat4(1.0f), glm::vec3(0, 0, 0));;



        // 创建着色器程序
        Shader shader("./res/shaders/Basic.shader");
        shader.Bind();
        // 获取并设置统一变量 u_Color 的位置和初始值 
        shader.SetUniform4f("u_Color", 0.8f, 0.3f, 0.8f, 1.0f);
        
        Texture texture("./res/textures/ChernoLogo.png");
        texture.Bind();
        shader.SetUniform1i("u_Texture",0);
        // 解除绑定 VAO、着色器程序、顶点缓冲区和索引缓冲区
        va.Unbind();
        shader.Unbind();
        vb.Unbind();
        ib.Unbind();        
        /***********************动画过渡+uniform形式*************************/
        //创建渲染器
        Renderer renderer;
        //创建imgui上下文
        ImGui::CreateContext();
        ImGui_ImplGlfwGL3_Init(window, true);
        ImGui::StyleColorsDark();
        glm::vec3 translationA(200, 200, 0);
        glm::vec3 translationB(400, 200, 0);
        // 定义颜色变化的初始值和增量
        float r = 0.0f;
        float increment = 0.05f;
        /************************************************/
        //imgui窗口



        //使用 glBufferData 函数为 GL_ARRAY_BUFFER 目标的当前绑定缓冲区分配存储空间
        //参数分别为：指定缓冲区目标，大小，上传的数据，数据的使用方式
        /* Loop until the user closes the window */
        //主渲染循环，直到窗口关闭
        while (!glfwWindowShouldClose(window))
        {
            /* Render here 在这里渲染*/
            // 清除颜色缓冲区
            renderer.Clear();
            //创建帧
            ImGui_ImplGlfwGL3_NewFrame();
            // 使用着色器程序，并设置统一变量 u_Color 的值
            
            {
                //创建模型矩阵，模型上移
                glm::mat4 model = glm::translate(glm::mat4(1.0f), translationA);
                //下面即为模型视图投影
                glm::mat4 mvp = proj * view * model;
                /***********************动画过渡***************************/
                shader.Bind();
                shader.SetUniformMat4f("u_MVP", mvp);
                renderer.Draw(va, ib, shader);
            }
            {
                //创建模型矩阵，模型上移
                glm::mat4 model = glm::translate(glm::mat4(1.0f), translationB);
                //下面即为模型视图投影
                glm::mat4 mvp = proj * view * model;
                /***********************动画过渡***************************/
                shader.Bind();
                shader.SetUniformMat4f("u_MVP", mvp);
                renderer.Draw(va, ib, shader);
            }
            //GLCall(glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, nullptr));
            // 更新颜色变化的值
            if (r > 1.0f)
            {
                increment = -0.05f;
            }
            else if (r < 0.0f)
                increment = 0.05f;
            r += increment;
            //imgui窗口******************************************************************************/
            {
                ImGui::SliderFloat3("Translation A", &translationA.x, 0.0f, 960.0f);            // Edit 1 float using a slider from 0.0f to 1.0f    
                ImGui::SliderFloat3("Translation B", &translationB.x, 0.0f, 960.0f);            // Edit 1 float using a slider from 0.0f to 1.0f    
                ImGui::Text("Application average %.3f ms/frame (%.1f FPS)", 1000.0f / ImGui::GetIO().Framerate, ImGui::GetIO().Framerate);
            }
            /**************************************************************************/
            ImGui::Render();
            ImGui_ImplGlfwGL3_RenderDrawData(ImGui::GetDrawData());
            /********************************************************/
    
            //绘制方式允许你通过索引来重复使用顶点
            // mode：指定要渲染的几何图形的模式，例如 GL_TRIANGLES。
            //count：指定要渲染的元素数量。
            //type：指定元素数组中索引的数据类型，例如 GL_UNSIGNED_INT 或 GL_UNSIGNED_BYTE。
            //indices：指向包含索引的数组的指针。
    
            //glDrawArrays(GL_TRIANGLES, 0, 6);
            //三个参数分别为mode：指定绘制模式，GL_TRIANGLES 表示每个三个顶点组成一个三角形
            //first：指定从顶点数组中开始读取顶点数据的起始索引
            //count：指定要绘制的顶点数量。设置为 3，意味着绘制三个顶点，形成一个三角形


            /* Swap front and back buffers */
            glfwSwapBuffers(window);
    
            /* Poll for and process events */
            glfwPollEvents();
        }
        
    }
    ImGui_ImplGlfwGL3_Shutdown();
    ImGui::DestroyContext();
    glfwTerminate();
    return 0;

}
```

> [!NOTE]
>
> 这次更新了主函数的代码，上面为更新前的批量渲染下面为更新后的测试

```
#include<GL/glew.h>
#include <GLFW/glfw3.h>
#include<iostream>
#include<string>
//使用文件流
#include<fstream>
//使用字符串流
#include<sstream>
#include "Renderer.h"
#include "IndexBuffer.h"
#include "VertexBuffer.h"
#include "VertexArray.h"
#include "Shader.h"
#include "VertexBufferLayout.h"
#include "Texture.h"

#include "glm/glm.hpp"
#include "glm/gtc/matrix_transform.hpp"
#include "imgui/imgui.h"
#include "imgui/imgui_impl_glfw_gl3.h"
#include "tests/TestClearColor.h"

int main(void)
{
    GLFWwindow* window;

    /* Initialize the library */
    if (!glfwInit())
        return -1;
    
    //创建一个glfwwindow上下文
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);




    /* Create a windowed mode window and its OpenGL context */
    window = glfwCreateWindow(640, 480, "Hello World", NULL, NULL);
    if (!window)
    {
        glfwTerminate();
        return -1;
    }
    
    /* Make the window's context current */
    //在这里创建glew的渲染上下文
    /*************************************/
    glfwMakeContextCurrent(window);
    
    glfwSwapInterval(1); 
    
    if (glewInit() != GLEW_OK)
        std::cout << "Error!" << std::endl;
    std::cout << glGetString(GL_VERSION) << std::endl;
    {


        //进行混合
        GLCall(glEnable(GL_BLEND));
        GLCall(glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA));


​        

        Renderer renderer;
        //创建imgui上下文
        ImGui::CreateContext();
        ImGui_ImplGlfwGL3_Init(window, true);
        ImGui::StyleColorsDark();


        //创建test
       test::TestClearColor test;


        //使用 glBufferData 函数为 GL_ARRAY_BUFFER 目标的当前绑定缓冲区分配存储空间
        //参数分别为：指定缓冲区目标，大小，上传的数据，数据的使用方式
        /* Loop until the user closes the window */
        //主渲染循环，直到窗口关闭
        while (!glfwWindowShouldClose(window))
        {
            /* Render here 在这里渲染*/
            // 清除颜色缓冲区
            renderer.Clear();
            test.onUpdate(0.0f);
            test.onRender();
            //创建帧
            ImGui_ImplGlfwGL3_NewFrame();
            test.onImGuiRender();
            ImGui::Render();
            ImGui_ImplGlfwGL3_RenderDrawData(ImGui::GetDrawData());
            /********************************************************/
    
            //绘制方式允许你通过索引来重复使用顶点
            // mode：指定要渲染的几何图形的模式，例如 GL_TRIANGLES。
            //count：指定要渲染的元素数量。
            //type：指定元素数组中索引的数据类型，例如 GL_UNSIGNED_INT 或 GL_UNSIGNED_BYTE。
            //indices：指向包含索引的数组的指针。
    
            //glDrawArrays(GL_TRIANGLES, 0, 6);
            //三个参数分别为mode：指定绘制模式，GL_TRIANGLES 表示每个三个顶点组成一个三角形
            //first：指定从顶点数组中开始读取顶点数据的起始索引
            //count：指定要绘制的顶点数量。设置为 3，意味着绘制三个顶点，形成一个三角形


            /* Swap front and back buffers */
            glfwSwapBuffers(window);
    
            /* Poll for and process events */
            glfwPollEvents();
        }
        
    }
    ImGui_ImplGlfwGL3_Shutdown();
    ImGui::DestroyContext();
    glfwTerminate();
    return 0;

}
```

