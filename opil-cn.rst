****
OTIL 规范
****




数据定义
=======

定义::

    /**
    * 数据类型
    */
    enum DataType {
        /**
        * 空
        */
        NULL    = 0x00
        /**
        * 字符串
        */
        STR     = 0x01
        /**
        * 整数
        */
        INT     = 0x02
        /**
        * 长整数
        */
        LONG    = 0x03
        /**
        * 浮点数
        */
        FLOAT   = 0x04
        /**
        * 真
        */
        TRUE    = 0x05
        /**
        * 假
        */
        FALSE   = 0x06
    }

    /**
    * 运算符
    */
    enum Operator {
        /**
        * 相加
        */
        Add     = 0x01
        /**
        * 相减
        */
        Sub     = 0x02
        /**
        * 相乘
        */
        Mul     = 0x03
        /**
        * 相除
        */
        Div     = 0x04
        /**
        * 取模
        */
        Mod     = 0x05
        /**
        * 取负
        */
        Neg     = 0x06
        /**
        * 取正
        */
        Pos     = 0x07
        /**
        * 逻辑等于
        */
        Eq      = 0x08
        /**
        * 逻辑不等于
        */
        Ne      = 0x09
        /**
        * 逻辑大于
        */
        Gt      = 0x0A
        /**
        * 逻辑大于等于
        */
        Ge      = 0x0B
        /**
        * 逻辑小于
        */
        Lt      = 0x0C
        /**
        * 逻辑小于等于
        */
        Le      = 0x0D
        //Leg   = 0x014
        /**
        * 逻辑与
        */
        And     = 0x0E
        /**
        * 逻辑或
        */
        Or      = 0x0F

    }

    /**
    * 字符编码
    */
    enum Encodings{
        UTF8    = 0x00
        ASCII   = 0x01
    }


基本数据类型
===========

操作码头 CodeHead ::

    Ptr     : 2字节(uint16,0~65535) 地址
    CodeType: 1字节(byte，0~255)    编码(值由具体操作码定义为常值)
    LineNo  : 2字节(uint16,0~65535) 行号
    Flag    : 1字节(byte，0~255)    保留或控制标志(由具体操作码定义为常值)


字符串 String::

    字符串长度: int32
    字符串   :  []byte




Doc(文件头)
===========

结构::

    名称：4字节(固定字符OTIL)
    版本：2字节(uint16)
    编码：1字节(固定0为UTF8编码)
    保留：3字节
    头结束地址：2字节
    源文件修改时间: 8字节(int64,UNIX时间戳毫秒)
    源文件相对路径：字符串
    
    


Nop
===

表示一个占位，不执行具体操作，结构::

    CodeHead{
        CodeType: 0x02
    }


Jump
=====

表示一个跳转或中断，结构::

    CodeHead{
        CodeType: 0x03
        Flag    : 类型：
                        0x01: EXIT 退出当前流
                        0x02: NEVER 始终跳转到指定地址
                        0x03: TRUE 从栈顶弹出一个元素，值为真则跳转到指定地址
                        0x04: FALSE 从栈顶弹出一个元素，值为假则跳转到指定地址
    }
    目标地址：Ptr，如中断类型为0x00则没有该地址


LoadConst
=========

表示载入一个常量到栈顶，结构::

    CodeHead{
        CodeType: 0x04
        Flag    : DataType (值类型)
    }
    值：根据类型决定


LoadVariable
============

表示载入一个变量到栈顶，结构::

    CodeHead{
        CodeType: 0x05
    }
    变量名: String



SetVariable
===========

从栈顶获取一个值并设置到指定变量，结构::

    CodeHead{
        CodeType: 0x06
    }
    变量名: String

Call
====

调用一个函数
从栈顶获取一个元素字符串（函数名）或方法(如js的闭名，java的method)
如果是方法则从栈顶获取调用方法的实例对象
根据参数个数获取参数，执行并将结果存入栈顶
结构::

    CodeHead{
        CodeType: 0x07
        Flag    : 参数个数
    }


Print
=====

从栈顶弹出一个元素并打印到输出流，结构::

    CodeHead{
        CodeType: 0x08
        Flag    : bool (是否编码)
    }


Operation
=========

根据运算类型从栈顶弹出相应元素并将结果存入栈顶，结构::

    CodeHead{
        CodeType: 0x09
        Flag    : Operator (运算符)
    }


LoadMember
==========

表示载入一个对象成员到栈顶
从栈顶弹出对象实例对象
从栈顶弹出参数组
结构::

    CodeHead{
        CodeType: 0x0A
        Flag    : 参数个数
    }

Scope
=====

表示开启或关闭一个变量范围
结构::

    CodeHead{
        CodeType: 0x0B
        Flag    : bool(true表示启用新的作用域，false 表示关闭当前作用域)
    }

Block
=====

声明一个块。
块结束时有一个break.EXIT,解释到这里时退出块的执行
结构::

    CodeHead{
        CodeType: 0x0C
    }
    块名:String


BlockCall
=========

调用一个块。
在调用块的时候将会使用一个新的scope
结构::

    CodeHead{
        CodeType: 0x0D
        Flag    : 参数个数
    }
    块名:String


Reference
=========

引用另一个资源。
结构::

    CodeHead{
        CodeType: 0x0E
        Flag    : 类型：
                        0x01: INCLUDE 包含，如果另一个存在则调用
                        0x02: REQUIRE 资源必须存在，并且只调用其头定义，不执行主体
                        0x03: LAYOUT 做为一个布局文件存在
    }
    资源地址：String


CastToIterator
==============

从栈顶弹出一个元素转换成迭代后再存入栈顶。
结构::

    CodeHead{
        CodeType: 0x0F
    }

迭代器 Iterator::

    interface Iterator{
        // 检查是否还可以继续迭代
        bool hasNext()                                       

        // 迭代并返回值
        object next()                                          

        // 设置上下文件的变量
        // keyName 键的变量名
        // valueName 值的变量名
        void setVariable(keyName string,valueName string)    
    }


