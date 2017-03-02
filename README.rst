********************
OTPL 模板编程语言规范
********************

**OpenTPL** (or OTPL) 是开放模板编程语言(Open Template Programming Language)的简称。
她由 ``OTPL`` 模板语言与 ``OTIL`` 模板中间语言两个部分组成，她简单易于使用，抱着开放的心态促进模板化编程的统一。
她努力使她能在当今任何语言、任何平台中使用而不用另外多做工作。


layout.html::

    <html>
        <header>
            {{@header}}
        </header>
        <body>
            {{@body}}
        </body>
    </html>


otpl.html::

    {{layout "layout" }}

    {{block header}}
    <title>{{ title }}</title>
    {{/block}}

    <ul>
        {{for name, item : items}}
            <li>{{ name }}: {{ item }}</li>
        {{/for}}
    </ul>


导航
====

- OTPL `中文语法规范`_
- OTIL `中文IL规范`_


实现引擎
=======
- Nodejs https://github.com/opentpl/otpl-node

- Golang

- Java

- PHP

- C#/.Net 



许可协议
=======
为了维护OTPL的统一性，OTPL使用对商业友好但不允许产生另一个“OTPL”的 `知识共享署名-禁止演绎 4.0`_ 国际许可协议进行许可。

.. image:: https://i.creativecommons.org/l/by-nd/4.0/88x31.png

.. _知识共享署名-禁止演绎 4.0: http://creativecommons.org/licenses/by-nd/4.0/
.. _中文语法规范: https://github.com/opentpl/otpl-spec/blob/master/syntax-cn.rst
.. _中文IL规范: https://github.com/opentpl/otpl-spec/blob/master/otil-cn.rst
